---
title: "构建Singularity镜像"
teaching: 30
exercises: 30
questions:
- "如何创建自己的 Singularity 镜像？"
objectives:
- "了解不同的 Singularity 容器文件格式。"
- "了解如何构建和共享您自己的 Singularity 容器。"
keypoints:
- "Singularity定义文件用于定义映像的构建过程和配置。"
- "Singularity 的 Docker 容器提供了一种在未安装 Singularity 但可以使用 Docker 的平台上构建映像的方法。"
- "来自远程注册中心（如 Docker Hub 和 Singularity Hub）的现有镜像可用作创建新 Singularity 镜像的基础。"
---

## 构建Singularity镜像

### 介绍

Singularity作为在科研软件和HPC社区中广泛使用的平台，为重现性提供了强大的支持。如果您为某些科学软件构建Singularity镜像，您和/或其他人可能希望能够再次重现完全相同的环境。也许您想验证代码的结果，或者提供一种其他人可以用来验证结果的方法来支持论文或报告。也许您正在向其他人提供一个工具，并希望确保他们拥有完全正确的代码版本/配置。

与Docker和许多其他现代软件工具类似，Singularity遵循“配置即代码”方法，容器配置可以存储在一个文件中，然后可以与其他代码一起提交到您的版本控制系统。假设它被适当配置，那么您或其他人（或通过自动构建工具）可以使用此文件在将来的某个时候重现具有相同配置的容器。

### 构建镜像的不同方法

构建奇点镜像有多种方法。 我们在这里强调两种不同的方法，并重点关注其中一种：

- _在沙盒中构建：_ 您可以在沙盒环境中交互地构建容器。这意味着您在容器环境中获得一个shell，并在退出沙箱并将其转换为容器映像之前根据需要安装和配置包和代码。
- _从 [Singularity 定义文件] 构建（https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories）_：这相当于从 Singularity 构建 Docker 容器的一个`Dockerfile`，我们将在本节讨论这种方法。

您可以查看 Singularity 的“[构建容器](https://sylabs.io/guides/3.5/user-guide/build_a_container.html)”文档，了解有关构建容器的不同方法的更多详细信息。

> ## 为什么要查看Singularity定义文件？
> 为什么您认为我们可能会在这里查看_定义文件方法_而不是_沙盒方法_？
>
> > ## 讨论
> >
> > 沙盒方法非常适合制作原型和测试镜像配置，但它不能为我们的_reproducibility_ 最终目标提供最佳支持。如果您花时间坐在终端前的shell 前键入不同的命令来添加配置，那么您可能意识到自己犯了一个错误，因此您撤消了一项配置并进行了更改。这一直持续到您完成了工作配置，但没有明确记录您为创建该配置所做的确切操作。
> >
> > 假设您的容器映像文件被意外删除，或者其他人想要创建一个等效的映像来测试某些东西。他们将如何做到这一点并确定他们具有与您相同的配置？
> > 使用定义文件，配置步骤被明确定义并且可以轻松存储（和重新运行）。
> >
> > 定义文件是小型文本文件，而容器文件可能是非常大的数GB文件，移动起来既困难又费时。这使得定义文件及其修订版非常适合存储在版本控制系统中。
> {: .solution}
{: .challenge}

### 创建Singularity定义文件

Singularity定义文件是一个文本文件，其中包含一系列用于创建容器映像的语句。根据上面提到的 _configuration as code_ 方法，定义文件可以与您的应用程序代码一起存储在您的代码存储库中，并用于创建可重现的镜像。 这意味着对于存储库中的给定提交，该提交中存在的定义文件的版本可用于重现具有已知状态的容器。

我们现在来看一个非常简单的定义文件示例：

~~~
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'
~~~
{: .language-bash}

定义文件有许多可选部分，使用`%`前缀指定，用于在映像构建过程的不同阶段定义或进行不同的配置。您可以在 Singularity 的 [定义文件文档](https://sylabs.io/guides/3.5/user-guide/definition_files.html) 中找到完整的详细信息。 在我们非常简单的示例中，我们只使用了 `%post` 和 `%runscript` 部分。

让我们逐步浏览这个定义文件并更详细地查看这些行：

~~~
Bootstrap: docker
From: ubuntu:20.04
~~~
{: .language-bash}

前两行定义了从哪里 _bootstrap_ 我们的镜像。为什么我们不能将一些应用程序二进制文件放入空白镜像中？我们想要运行的任何应用程序或工具都需要与标准系统库以及潜在的各种其他库和工具进行交互。这些需要在映像中可用，因此我们需要某种操作系统作为映像的基础。实现此目的最直接的方法是从包含操作系统的现有基础映像开始。在本例中，我们将从最小的 Ubuntu 20.04 Linux Docker 映像开始。请注意，我们使用 Docker 映像作为创建 Singularity 映像的基础。这展示了在创建新的 Singularity 镜像时能够从不同类型的镜像开始的灵活性。

`Bootstrap: docker` 行类似于在使用 `singularity pull` 命令时为镜像路径添加前缀 `docker://`。支持一系列 [不同的引导选项](https://sylabs.io/guides/3.5/user-guide/definition_files.html#preferred-bootstrap-agents)。 `From: ubuntu:20.04` 表示我们想使用来自 Docker Hub 的带有标签 `20.04` 的 `ubuntu` 镜像。

接下来我们有定义文件的 `%post` 部分：

~~~
%post
    apt-get -y update && apt-get install -y python3
~~~
{: .language-bash}

在文件的这一部分中，我们可以执行诸如包安装、从远程位置提取数据文件以及在映像中进行本地配置等任务。 本节中出现的命令是标准的 shell 命令，它们在我们的新容器映像的上下文中运行。 因此，在本示例中，这些命令是在一个最小的 Ubuntu 20.04 映像的上下文中运行的，该映像最初只安装了非常小的一组核心软件包。

在这里，我们使用 Ubuntu 的包管理器来更新我们的包索引，然后安装 `python3` 包以及任何所需的依赖项。 默认情况下，“-y”开关用于接受可能出现的交互式提示，要求您确认包更新或安装。 这是必需的，因为我们的定义文件应该能够在无人值守的非交互环境中运行。

最后我们有 `%runscript` 部分：

~~~
%runscript
    python3 -c 'print("Hello World! Hello from our custom Singularity image!")'
~~~
{: .language-bash}

本节用于定义一个脚本，该脚本应在容器基于此镜像使用 `singularity run` 命令启动时运行。 在这个简单的示例中，我们使用 `python3` 将一些文本打印到控制台。

我们现在可以将上面显示的简单定义文件的内容保存到一个文件中，并基于它构建一个镜像。 在本例中，定义文件已命名为“my_test_image.def”。 （请注意，此处的说明假设您已将创建的镜像输出目录绑定到 Docker Singularity 容器中的 `/home/singularity` 目录，如“[_Docker Singularity 镜像入门_]（#getting- 开始于-docker-singularity-image）”部分。）：

~~~
$ singularity build /home/singularity/my_test_image.sif /home/singularity/my_test_image.def
~~~
{: .language-bash}

回想一下本节开头的详细信息，如果您从主机系统命令行运行命令，并为每次运行该命令运行一个 Docker 容器实例，您的命令将如下所示：

~~~
$ docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim build /home/singularity/my_test_image.sif /home/singularity/my_test_image.def
~~~
{: .language-bash}

上述命令请求基于 `my_test_image.def` 文件构建镜像，并将生成的镜像保存到 `my_test_image.sif` 文件中。 请注意，如果您正在运行本地安装的 Singularity 版本而不是通过 Docker 运行，则需要在命令前加上 `sudo`，因为必须具有管理权限才能构建映像。 您应该会看到类似于以下内容的输出：

~~~
INFO:    Starting build...
Getting image source signatures
Copying blob d51af753c3d3 skipped: already exists
Copying blob fc878cd0a91c skipped: already exists
Copying blob 6154df8ff988 skipped: already exists
Copying blob fee5db0ff82f skipped: already exists
Copying config 95c3f3755f done
Writing manifest to image destination
Storing signatures
2020/04/29 13:36:35  info unpack layer: sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e
2020/04/29 13:36:36  info unpack layer: sha256:fc878cd0a91c7bece56f668b2c79a19d94dd5471dae41fe5a7e14b4ae65251f6
2020/04/29 13:36:36  info unpack layer: sha256:6154df8ff9882934dc5bf265b8b85a3aeadba06387447ffa440f7af7f32b0e1d
2020/04/29 13:36:36  info unpack layer: sha256:fee5db0ff82f7aa5ace63497df4802bbadf8f2779ed3e1858605b791dc449425
INFO:    Running post scriptlet
+ apt-get -y update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
...
  [Package update output truncated]
...
Fetched 13.4 MB in 2s (5575 kB/s)                            
Reading package lists... Done
+ apt-get install -y python3
Reading package lists... Done
...
  [Package install output truncated]
...Processing triggers for libc-bin (2.31-0ubuntu9) ...
INFO:    Adding runscript
INFO:    Creating SIF file...
INFO:    Build complete: my_test_image.sif
$ 
~~~
{: .output}

您现在应该在当前目录中有一个 `my_test_image.sif` 文件。 请注意，在上面的输出中，“INFO: Starting build...”有一系列“skipped: already exists”消息用于“Copying blob”行。 这是因为 Ubuntu 20.04 映像的 Docker 映像切片之前已下载并缓存在运行此示例的系统上。 在您的系统上，如果镜像尚未缓存，当这些输出行出现时，您将看到从 Docker Hub 下载的切片。

> ## 创建的镜像文件的权限
>
> 您可能会发现在您的主机文件系统上创建的 Singularity 镜像文件归 `root` 用户所有，而不是您的用户。 在这种情况下，如果您没有 root 访问权限，您将无法直接更改文件的所有权/权限。
>
> 但是，您可以读取镜像文件，并且您应该能够以您将拥有的新名称获取该文件的副本。 然后，您将能够修改此映像副本的权限并删除原始 root 拥有的文件，因为默认权限应该允许这样做。
> 
{: .callout}

> ## 用于运行 Singularity 容器的集群平台配置
>
> 在此框中添加任何自定义配置的详细信息，这些配置需要在集群平台或您为学习本课程而提供访问权限的其他远程系统上完成。 如果 `singularity` 不需要用户在宿主平台上进行任何自定义配置，你可以去掉这个框。_
> 
{: .callout}

建议您将创建的 .sif 文件移动到安装了 Singularity 的平台，而不是尝试使用 Docker 容器运行映像。 但是，如果您确实希望尝试使用 Docker 容器，请参阅下面关于“_在 Docker 容器内运行Singularity_”的注释以获取更多信息。

如果您可以访问安装了 Singularity 的远程平台，您现在应该将创建的“.sif”镜像文件移动到该平台。 例如，您可以使用命令行安全复制命令 `scp` 执行此操作。

> ## 使用`scp`（安全复制）在系统之间复制文件
>
> `scp` 是一种广泛使用的工具，它使用 SSH 协议在系统之间安全地复制文件。 因此，语法类似于 SSH。
>
> 例如，如果您想将 `my_image.sif` 文件从本地系统的当前目录复制到远程系统上的主目录（例如 `/home/myuser/`) 在登录时需要 SSH 私钥的情况下，您将使用类似于以下的命令：
>
> ```
> scp -i /path/to/keyfile/id_mykey ./my_image.sif myuser@hpc.myinstitution.ac.uk:/home/myuser/
> ```
> 请注意，如果您离开 `/home/myuser` 并仅以 `:` 结束命令，默认情况下，该文件将被复制到您的主目录。
>
{: .callout}

我们现在可以尝试从我们构建的镜像运行容器：

~~~
$ singularity run my_test_image.sif
~~~
{: .language-bash}

如果一切顺利，您应该会看到 Python 打印的消息：

~~~
Hello World! Hello from our custom Singularity image!
~~~
{: .output}

> ## 在 Docker 容器中使用“singularity run”
>
> 强烈建议您不要使用 Docker 容器来运行 Singularity 映像，而仅用于创建它们，因为 Singularity 命令以 root 用户身份在容器中运行。
>
> 但是，对于这个简单示例的目的，以及潜在的测试/调试目的，了解如何在 Docker Singularity 容器中运行 Singularity 容器很有用。 您可能还记得在上一集中的 [从镜像运行容器](/06-singularity-images-prep/index.html#running-a-container-from-the-image) 部分中，我们使用了 `- -contain 使用 `singularity` 命令切换。如果您不使用此开关，您很可能会收到与 `/etc/localtime` 相关的错误，类似于以下内容：
>
> ~~~
> WARNING: skipping mount of /etc/localtime: no such file or directory
> FATAL:   container creation failed: mount /etc/localtime->/etc/localtime error: while mounting /etc/localtime: mount source /etc/localtime doesn't exist
> ~~~
> {: .output}
>  
> 发生这种情况是因为 Docker 容器中不存在提供时区配置的“/etc/localtime”文件。 如果你想使用 Docker 容器来测试你新创建的镜像是否运行，你可以使用 `--contain` 开关，或者你可以在 Docker 容器中打开一个 shell 并添加一个时区配置，如 [Alpine Linux 文档]（https://wiki.alpinelinux.org/wiki/Setting_the_timezone）：
>
> ~~~
> $ apk add tzdata
> $ cp /usr/share/zoneinfo/Europe/London /etc/localtime
> ~~~
> {: .language-bash}
>  
> `singularity run` 命令现在应该可以成功运行，而无需使用 `--contain`。 请记住，一旦您退出 Docker Singularity 容器外壳并关闭容器，此配置将不会持续存在。
{: .callout}


### 更高级的定义文件

在这里，我们查看了如何创建镜像的一个非常简单的示例。 在这个阶段，您可能想尝试为您自己的某些代码或您经常使用的应用程序创建自己的定义文件。 上面的示例中_not_使用了几个定义文件部分，它们是：

 - `%setup`
 - `%files`
 - `%environment`
 - `%startscript`
 - `%test`
 - `%labels`
 - `%help`

[`Sections` 定义文件文档的部分](https://sylabs.io/guides/3.5/user-guide/definition_files.html#sections) 详细介绍了所有部分，并提供了一个示例定义文件，该文件利用了所有 部分。

### 额外的Singularity功能

Singularity具有广泛的特征。您可以在 [Singularity 用户指南](https://sylabs.io/guides/3.5/user-guide/index.html) 中找到完整的详细信息，我们在此重点介绍一些可能有用/感兴趣的关键功能：

**Remote Builder 功能：** 如果您可以访问安装了 Singularity 的平台，但您没有创建容器的 root 访问权限，则可以使用 [Remote Builder](https://cloud.sylabs.io/builder) 功能，用于将构建映像的过程卸载到远程云资源。

**签署容器：** 如果您确实想直接与同事或合作者共享容器镜像 (`.sif`) 文件，您发送镜像的人如何确保他们已收到文件而不会被篡改还是在传输/存储过程中遭受腐败？你怎么能确定你从其他人那里收到的任何容器镜像文件都是一样的呢？ Singularity 支持签署容器。这允许将数字签名链接到镜像文件。此签名可用于验证镜像文件是否已由特定密钥的持有者签名，并且该文件自签名时起未更改。您可以在 [Signing and Verifying Containers] (https://sylabs.io/guides/3.0/user-guide/signNverify.html) 的 Singularity 文档中找到有关如何使用此功能的完整详细信息。


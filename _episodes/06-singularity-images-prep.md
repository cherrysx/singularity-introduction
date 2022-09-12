---
title: "为构建Singularity镜像做准备"
teaching: 15
exercises: 20
questions:
- "构建Singularity镜像需要什么环境以及如何设置？"
objectives:
- "了解Docker镜像如何为构建Singularity镜像提供环境。"
- "了解基于Docker的Singularity镜像运行容器的不同方法。"
keypoints:
- "基于Docker镜像运行Singularity - 这避免了在您的系统上安装本地Singularity的需要。"
- "Docker Singularity镜像可用于在 Linux、macOS 和 Windows 上构建容器。"
- "您还可以在Docker Singularity镜像中运行Singularity容器。"
---

# Singularity - 第二部分

## 简要回顾

在涵盖本Singularity材料第一部分中，我们已经了解了如何在您没有任何管理权限的计算平台上使用 Singularity。 该软件是预安装的，可以使用现有的镜像，例如已经存储在平台上的 Singularity 镜像文件或从远程镜像存储库（例如 Singularity Hub 或 Docker Hub）获得的镜像。

很明显，在 Singularity Hub 和 Docker Hub 之间，有大量可用的镜像，预先配置了广泛的软件应用程序、工具和服务。 但是，如果您想创建自己的镜像或自定义现有镜像怎么办？

接下来，我们将着眼于准备构建 Singularity 镜像。

## 准备使用 Singularity 构建镜像

到目前为止，您已经能够从您自己的用户帐户作为非特权用户使用Singularity。 Singularity材料的这一部分要求您在具有管理（根）访问权限的环境中使用 Singularity。虽然可以在没有root访问权限的情况下构建Singularity容器，但强烈建议您以_root_用户身份执行此操作，如[本节](https://sylabs.io/guides/3.5/user-guide/build_a_container.html#creating-writable-sandbox-directories) 的 Singularity文档。请记住，您用于构建容器的系统不一定是您打算运行容器的系统。例如，如果您打算构建一个随后可以在基于Linux的集群上运行的容器，您可以在您自己的基于Linux的台式机或笔记本电脑上构建该容器。然后，您可以将构建的镜像直接传输到目标平台或将其上传到镜像存储库并从该存储库将其拉到目标平台上。

有**三种**不同的选项可用于访问合适的环境来学习这部分：

 1. 在Docker容器中运行Singularity - 这将使您拥有构建镜像所需的权限
 1. 在您具有管理权限的系统上本地安装Singularity
 1. 在已预安装Singularity且您具有管理（根）访问权限的系统上使用Singularity

在这部分中，我们将重点关注第一个选项——_从 Docker 容器中运行Singularity_。如果您想直接在您的系统上安装Singularity，请参阅下面的框以获取更多指示。但是，请注意，安装过程是一项超出范围的高级任务，因此我们不会对此进行介绍。

> ## 在本地系统上安装Singularity（可选）\[高级任务\]
>
> 如果您正在运行Linux并希望在您的系统上本地安装Singularity，源代码由 [下一代高性能计算 (HPCng) 社区](https://github.com/hpcng) 提供 [Singularity存储库]（https://github.com/hpcng/singularity）。请参阅 [此处](https://github.com/hpcng/singularity/releases) 的版本。您需要在系统上安装各种依赖项，然后从源代码构建Singularity。
>
> _如果您不熟悉从源代码构建应用程序，强烈建议您使用Docker Singularity 镜像，如下面的“Docker Singularity 镜像入门”部分所述，而不是尝试自己构建和安装 Singularity。安装过程是一项高级任务，超出了本次会议的范围。_
> 
> 但是，如果您有Linux系统知识并想尝试在本地安装Singularity，您可以在[INSTALL.md](https://github.com/hpcng/singularity/blob/master/INSTALL.md) 中找到详细信息 ) 文件在 Singularity 存储库中解释如何安装先决条件以及构建和安装软件。Singularity是用 [Go](https://golang.org/) 编程语言编写的，Go是您需要在系统上安装的主要依赖项。 安装Go的过程和任何其他要求在 INSTALL.md 文件中有详细说明。
> 
{: .callout}

> ## 笔记
> 如果您无法访问安装了Docker的系统，或者您可以构建和安装Singularity但您在另一个系统上具有管理权限的Linux系统，您可以考虑安装虚拟化工具，例如[VirtualBox](https: //www.virtualbox.org/)，您可以在其上运行Linux虚拟机 (VM) 镜像。在Linux VM镜像中，您将能够安装 Singularity。
>
> 如果您无法在您拥有管理权限的系统上自己访问/运行Singularity，您仍然可以按照正在教授的方式阅读本材料（或者如果您没有参加课程的教学版本），因为这将有助于了解如何构建Singularity镜像。
>
> 您也可以尝试不使用root而是使用`singularity`命令的 [`--fakeroot`]（https://sylabs.io/guides/3.5/user-guide/fakeroot.html ） 选项。但是，在尝试构建镜像和运行容器时，您可能会遇到权限问题，这就是强烈建议以 root 身份运行命令的原因，这也是本课中描述的方法。
{: .callout}

## Docker Singularity镜像入门

[Singularity Docker镜像](https://quay.io/repository/singularity/singularity) 可从 [Quay.io](https://quay.io/) 获得。

> ## 熟悉 Docker Singularity 镜像
>  
> - 使用您之前获得的Docker知识，获取Singularity镜像，并确保您可以使用此镜像运行Docker容器。
>  
> - 在您的主机上创建一个目录（例如`$HOME/singularity_data`），您可以使用它来存储_定义文件_（我们将很快介绍这些）和生成的镜像文件。
>
> 每次运行时，该目录都应该绑定挂载到 Docker 容器的 `/home/singularity` 位置 - 这将为您提供一个存储构建镜像的位置，以便一旦容器可以在主机系统上使用它们 退出。（看看 `-v` 切换到 `docker run` 命令）
>
> _提示：为了能够使用 Docker Singularity容器构建镜像，您需要将 `--privileged` 开关添加到docker命令行。_
>
> _提示：如果你想在Docker Singularity容器中运行shell，你需要覆盖入口点来告诉容器运行`/bin/bash` - 看看Docker的 `--entrypoint`开关。_
> 
> 
> 问题/练习：
> 
> 1. 可以从 Docker Singularity镜像运行容器吗？ 运行容器时发生了什么？
> 1. 可以在 Docker Singularity容器中运行交互式的 `/bin/sh` shell 吗？
> 1. 你能在 Docker Singularity容器中的 Singularity 容器中运行交互式 Singularity shell 吗？！
> 
> > ## 从镜像运行容器
> > 答案：
> > 1. _你能从 Docker Singularity 镜像运行容器吗？ 运行容器时发生了什么？_
> >
> > 我们将使用的 Docker Singularity 镜像的名称/标签是：`quay.io/singularity/singularity:v3.5.3-slim`
> >
> > 在您正在运行的 Docker Singularity 容器中拥有一个可从主机系统访问的绑定目录将为您提供放置创建的 Singularity 映像的位置，以便在容器退出后可以在主机系统上访问它们。 首先切换到您在上面创建的用于存储定义文件和构建镜像的目录（例如`$HOME/singularity_data`）。
> >
> > 从镜像中运行一个 Docker 容器，并将当前目录绑定到容器内的 `/home/singularity` 可以实现如下：
> >   
> >     ```
> >     docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim
> >     ```
> >     请注意，镜像默认配置为运行 `singularity` 命令。 因此，当您在不带参数的情况下运行容器时，您会看到奇异性帮助输出，就像您在本地安装了 Singularity 并在命令行上键入了“singularity”一样。
> >     
> >     要直接从主机系统的终端在 docker 容器中运行 Singularity 命令，例如“singularity cache list”，您需要输入：
> >     
> >     ```
> >     docker run --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim cache list
> >     ```
> >     
> >     下图显示了如何使用 Docker Singularity 映像在主机系统上运行容器，以及如何依次在 Docker 容器中启动 Singularity 容器：
> >     
> >     ![](/fig/SingularityInDocker.png)
> >     
> > 1. _你能在 Docker Singularity容器中运行交互式shell吗？_
> >
> > 要在 Singularity Docker 容器中启动一个 shell，您可以在其中直接运行 `singularity` 命令：
> >    ```
> >    docker run -it --entrypoint=/bin/sh --privileged --rm -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim
> >    ```
> >    在这里，我们使用`--entrypoint`切换到 `docker`命令来覆盖启动容器时的默认行为，而不是直接运行`singularity` 命令，我们运行 'sh' shell。 我们还添加了 `-it` 开关以提供交互式终端连接。
> >     
> >         
> > 1. _你能在 Docker Singularity容器中的 Singularity 容器中运行交互式 Singularity shell 吗？！_
> >
> > 如上图所示，您可以这样做。 需要在 Docker Singularity 容器中运行`singularity shell <image file name>`。 您将使用类似于以下的命令（假设 `my_test_image.sif` 位于运行此命令的当前目录中）：
> >     
> >    ```
> >    docker run --rm -it --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim shell --contain /home/singularity/my_test_image.sif
> >    ```
> >    
> >    你可能会注意到有一个标志被传递给了奇异性shell（`--contain` - `-c` 是简写形式并且也有效）。 这是在做什么？ 在运行奇异容器时，您可能还记得我们强调过，主机系统中的一些关键文件/目录在启动时默认映射到容器中。 Docker Singularity 容器中的配置尝试将文件`/etc/localtime` 挂载到 Singularity 容器中，但 Docker Singularity 容器中不存在时区配置，并且该文件不存在导致错误。 `--contain` 防止将某些关键文件/目录默认挂载到容器中，并防止发生此错误。 在本材料的后面，有一个示例说明如何通过在 Docker Singularity 容器中创建时区配置来纠正问题，从而不再需要 --contain 开关。
> > 
> > _总结/评论_
> >
> > 您可以选择：
> > - 在Docker映像中打开一个shell，以便您可以在命令提示符下工作并直接运行`singularity`命令
> > - 每次要运行`singularity`命令时，使用`docker run`命令运行一个新的容器实例（Docker Singularity 映像配置了`singularity 命令作为其入口点）。
> >
> > 命令示例将直接使用 `singularity` 命令，例如`singularity cache list`。如果您在 Docker Singularity 容器中运行 shell，则可以输入出现的命令。如果您使用容器的默认运行行为并为每次运行命令运行容器实例，则需要将 `singularity` 替换为 `docker run --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim` 或类似的。
> >
> > 这可能有点麻烦。但是，如果您在主机系统上使用 Linux 或 macOS，您可以添加 _command alias_ 来为主机系统上的命令 `singularity` 设置别名，以运行 Docker Singularity 容器，例如（对于 bash shell - 其他 shell 的语法有所不同）：
> > 
> > ```
> > alias singularity='docker run --privileged -v ${PWD}:/home/singularity quay.io/singularity/singularity:v3.5.3-slim'
> > ```
> >  
> > 这意味着您只需在命令行中键入“singularity”，如本部分材料中的示例所示
> >
> {: .solution}
{: .challenge}

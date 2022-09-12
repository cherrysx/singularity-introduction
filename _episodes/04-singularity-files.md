---
title: "Singularity容器中的文件"
teaching: 10
exercises: 10
questions:
- "如何使数据在 Singularity 容器中可用？"
- "Singularity 容器中默认提供哪些数据？"
objectives:
- "了解主机系统中的哪些数据通常在容器中默认可用"
- "了解有关 Singularity 如何处理用户和绑定主机文件系统中的目录的更多信息。"
keypoints:
- "默认情况下，您的当前目录和主目录通常在容器中可用。"
- "您在容器中拥有与主机系统相同的用户名和权限。"
- "您可以指定容器中可用的其他主机系统目录。"
---

在 Singularity 容器中处理用户帐户和访问权限的方式与在 Docker 中（您实际上始终拥有超级用户/root 访问权限）中的方式非常不同。 运行 Singularity 容器时，您仅具有与在主机系统上运行的用户相同的访问文件的权限。

在这里，我们将研究在 Singularity 容器的上下文中处理文件，以及这如何与 Singularity 对容器内的用户和权限的方法联系起来。

## Singularity容器中的用户

首先要注意的是，当您在上一节结束时开始的容器shell中运行“whoami”时，您应该已经看到运行容器时在主机系统上登录的用户名。

例如，如果我的用户名是 `jc1000`，我希望看到以下内容：

~~~
$ singularity shell hello-world.sif
Singularity> whoami
jc1000
~~~
{: .language-bash}

我从Singularity Hub下载了标准的公开版`hello-world.sif`镜像。我没有以任何方式定制它。它是如何使用我自己的用户详细信息配置的？！

如果您熟悉Linux系统管理，您可能知道在Linux中，用户及其组分别在`/etc/passwd`和`/etc/group`文件中配置。为了让容器内的shell知道我的用户，相关的用户信息需要在容器内的这些文件中可用。

假设在您的系统上安装 Singularity 时启用了此功能，当容器启动时，Singularity 会将主机系统中的相关用户和组行附加到 `/etc/passwd` 和 `/etc/group` 文件中容器 [\[1\]]（https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf）。

这意味着主机系统可以有效地确保您无法访问/修改/删除您不应在主机系统上访问的任何数据，并且您无法运行您无权在主机系统上运行的任何内容，因为您受到限制容器内的用户权限与您在主机系统上的用户权限相同。

## Singularity容器中的文件和目录

Singularity 还会将运行“singularity”命令的主机系统中的一些_目录_绑定到您正在启动的容器中。请注意，此绑定过程不是将文件复制到正在运行的容器中，而是使主机系统上的现有目录在容器环境中可见和可访问。如果您在正在运行的容器中将文件写入此目录，则当容器关闭时，这些更改将保留在主机系统上的相关位置。

将哪些文件和目录绑定到容器中有一个默认配置，但最终控制如何在运行Singularity的系统上进行设置是由系统管理员决定的。因此，本节提供了一个概述，但您可能会发现在您运行的系统上有些不同。

在您启动的容器中可能可以访问的一个目录是您的_home目录_。您可能还会发现您发出`singularity`命令的目录（_当前工作目录_）也被映射了。

下面的示例说明了文件内容和目录从主机系统到 Singularity 容器的映射，显示了主机 Linux 系统和 Singularity 容器中的目录子集：

~~~
Host system:                                                      Singularity container:
-------------                                                     ----------------------
/                                                                 /
├── bin                                                           ├── bin
├── etc                                                           ├── etc
│   ├── ...                                                       │   ├── ...
│   ├── group  ─> user's group added to group file in container ─>│   ├── group
│   └── passwd ──> user info added to passwd file in container ──>│   └── passwd
├── home                                                          ├── usr
│   └── jc1000 ───> user home directory made available ──> ─┐     ├── sbin
├── usr                 in container via bind mount         │     ├── home
├── sbin                                                    └────────>└── jc1000
└── ...                                                           └── ...

~~~
{: .output}

> ## 问题和练习：Singularity容器中的文件
>
> **Q1:** 对于从hello-world映像开始的容器中文件的所有权，您有什么注意事项？ （例如，查看根目录（`/`）中文件的所有权）
>
> **练习 1：** 在这个容器中，尝试编辑（例如使用容器中应该可用的编辑器 `vi`）`/rawr.sh` 文件。 你注意到什么？
>
> _如果你不熟悉 `vi`，网上有很多快速参考页面显示了使用编辑器的主要命令，例如 [this one](http://web.mit.edu/merolish/Public/vi- 参考.pdf)._
>
> **练习 2：** 在容器shell的主目录中，尝试创建一个简单的文本文件。 是否有可能做到这一点？ 如果是这样，为什么？ 如果不是，为什么不呢？！ 如果你可以成功创建一个文件，当你退出shell并且容器关闭时它会发生什么？
>
> > ## 答案
> >
> > **A1:** 使用`ls -l`命令查看详细的文件列表，包括文件所有权和权限详细信息。您应该看到 `/` 目录中的大多数文件都归`root`所有，正如您在任何Linux系统上所期望的那样。如果您查看主目录中的文件，它们应该归您所有。
> >
> > **A Ex1:** 我们已经从上一个答案中看到`/` 中的文件归`root`所有，所以如果我们不是root，我们不希望能够编辑它们用户。但是，如果您尝试编辑`/rawr.sh`，您可能会看到该文件是只读的，如果您尝试删除该文件，您会看到类似于以下内容的错误：`cannot remove '/rawr .sh'：只读文件系统`。这告诉我们有关文件系统的其他信息。不仅仅是我们没有删除文件的权限，文件系统本身也是只读的，所以即使是“root”用户也无法编辑/删除这个文件。稍后我们将更详细地讨论这一点。
> >
> > **A Ex2:** 在您的主目录中，您_应该_能够成功创建文件。由于您在主机系统上看到已绑定到容器中的主目录，因此当您退出并且容器关闭时，当您在容器上查看您的主目录时，您在容器中创建的文件应该仍然存在主机系统。
> {: .solution}
{: .challenge}

## 将其他主机系统目录绑定到容器

您有时需要将其他主机系统目录绑定到您正在使用的容器中，而不是默认绑定的那些。 例如：

- 其它目录中可能有一个共享数据集，您需要在容器中访问该数据集
- 您可能需要容器中的可执行文件和软件库

`singularity` 命令的`-B`选项用于指定附加绑定。 例如，要将 `/work/z19/shared` 目录绑定到您可以使用的容器中（请注意，您正在使用的主机系统上不太可能存在此目录，因此您需要使用不同的目录进行测试）：

```
$ singularity shell -B /work/z19/shared hello-world.sif
Singularity> ls /work/z19/shared
```
{: .language-bash}
```
file1 file2 file3
file4 file5 file6
```
{: .output}

请注意，默认情况下，绑定安装在容器中与主机系统上相同的路径。 您还可以通过在选项中用冒号 (`:`) 分隔主机路径和容器路径来指定主机目录在容器中的挂载位置：

```
$ singularity shell -B /work/z19/shared:/shared-data hello-world.sif
Singularity> ls /shared-data
```
{: .language-bash}
```
file1 file2 file3
file4 file5 file6
```
{: .output}

您还可以通过逗号 (`,`) 分隔多个绑定到 `-B`。

如果镜像中需要一些静态数据，您还可以在构建时将数据复制到容器镜像中。 我们稍后会在构建 Singularity 容器的部分中介绍这一点。

## References

\[1\] Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Singularity P2. Intel HPC Developer Conference, 2017. Available at: https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf

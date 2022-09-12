---
title: "Singularity: 入门"
start: true
teaching: 30
exercises: 20
questions:
- "什么是Singularity，我为什么要使用它？"
objectives:
- "了解Singularity是什么以及何时可能需要使用它。"
- "首次运行一个简单的Singularity容器。"
keypoints:
- "Singularity是另一个容器平台，通常用于集群/HPC/研究环境。"
- "Singularity具有与其他容器平台不同的安全模型，这是它非常适合HPC和集群环境的关键原因之一。"
- "Singularity 有自己的容器映像格式 (SIF)。"
- "`singularity` 命令可用于从 Singularity Hub 拉取镜像并从镜像文件运行容器。"
---

本课的章节将向您介绍[Singularity](https://sylabs.io/singularity/)容器平台，并演示如何设置和使用Singularity。

该材料分为两部分：

*第一部分：基本用法，使用镜像*
 1. **Singularity：入门**：介绍
    
使用Singularity容器：
<ol start="2">
 <li><strong>Singularity缓存: </strong> Singularity 为什么、在哪里以及如何在本地缓存镜像？</li>
 <li><strong>在Singularity容器中运行命令：</strong> 如何在 Singularity 容器中运行命令。</li>
 <li><strong>使用文件和Singularity容器：</strong> 将文件移动到 Singularity 容器中；从容器内访问主机上的文件。</li>
 <li><strong>使用具有 Singularity 的Docker镜像：</strong>如何从Docker镜像运行 Singularity 容器。</li>
 </ol>
*第二部分：创建镜像，运行并行代码*
 <ol start="6">
   <li><strong>准备构建 Singularity 镜像</strong>：Docker Singularity 容器入门。</li>
   <li><strong>构建Singularity镜像</strong>：解释如何构建和共享您自己的Singularity镜像。</li>
   <li><strong>使用Singularity容器运行MPI并行作业</strong>：解释如何从Singularity容器中运行MPI并行代码。</li>
</ol>

# Singularity - 第1部分

## 什么是Singularity?

[Singularity](https://sylabs.io/singularity/) 是另一个容器平台。从用户的角度来看，在某些方面它与Docker相似，但在其他方面，特别是在系统架构方面，它是根本不同的。这些差异意味着Singularity特别适合在分布式、高性能计算 (HPC) 基础架构以及Linux笔记本电脑或台式机上运行！

系统管理员通常不会将Docker安装在共享计算平台上，例如实验室桌面、研究集群或HPC平台，因为Docker的设计为多用户共享平台带来了潜在的安全问题。另一方面，Singularity可以由最终用户完全在“用户空间”内运行，也就是说，不需要为用户分配特殊的管理权限，以便他们在已安装Singularity所在的平台上运行和与容器交互。

## Singularity入门
Singularity最初在研究社区内开发，是开源的，目前可在“[下一代高性能计算](https://github.com/hpcng)” 的[repository](https://github.com/hpcng/singularity) 访问源码。 本Singularity材料的第一部分旨在在预安装Singularity的远程平台上进行。

> ## 在您自己的笔记本电脑/台式机上安装 Singularity
> 如果您有一个具有管理员权限的Linux系统，并且您想在该系统上安装Singularity，请学习[Singularity 资料的第二部分]({{ page.root }}/06-singularity-images-prep/index.html#installing-singularity-on-your-local-system-optional-advanced-task）。
{: .callout}

登录到您已获得访问权限且安装了Singularity的远程平台。检查您的终端中是否有 `singularity` 命令可用：

> ## 加载模块
> HPC 系统通常使用 *modules* 来提供对系统上软件的访问，因此您可能需要使用以下命令：
> ~~~
> $ module load singularity
> ~~~
> {: .language-bash}
> before you can use the `singularity` command on the system.
{: .callout}

~~~
$ singularity --version
~~~
{: .language-bash}

~~~
singularity version 3.5.3
~~~
{: .output}

根据系统上安装的 Singularity 版本，您可能会看到不同的版本。 在撰写本文时，`v3.9.9` 是 Singularity 的最新版本。

## 镜像和容器

我们将从简要说明本课程这一部分中使用的术语开始。 我们指的是 **_镜像_** 和 **_容器_**。 这两个术语有什么区别？

**_镜像_** 是文件包，包括操作系统、软件和潜在的数据以及其他与应用程序相关的文件。它们有时可能被称为_磁盘映像_或_容器映像_，它们可能以不同的方式存储，可能作为单个文件或作为一组文件。 无论哪种方式，我们都会将此文件或文件集合称为镜像。

**_container_** 是基于镜像的虚拟环境。也就是说，正在运行的容器中可用的文件、应用程序、工具等由启动容器的映像决定。 可以从一个镜像启动多个容器实例。也许，您可以将镜像视为一种模板形式，从中可以启动正在运行的容器实例。

## 获取镜像并运行 Singularity 容器

如果你回想一下Docker的学习，Docker镜像是由一组构成完整镜像的层组成的。 当您从 Docker Hub 拉取 Docker 映像时，您会看到不同的层正在下载到您的系统中。 它们存储在系统上的本地 Docker 存储库中，您可以使用 `docker` 命令查看可用镜像的详细信息。

Singularity镜像有点不同。Singularity 使用 [Singularity Image Format (SIF)](https://github.com/sylabs/sif) 并且镜像作为单个 `SIF` 文件（带有 `.sif` 文件扩展名）提供。 Singularity镜像可以从容器镜像的注册中心 [Singularity Hub](https://singularity-hub.org/) 中提取。 Singularity 还能够基于从 [Docker Hub](https://hub.docker.com/) 和其他一些来源提取的镜像运行容器。 稍后我们将在 Singularity 材料中介绍从 Docker Hub 访问容器。

> ## Singularity镜像中心
> 请注意，除了提供可以从中提取镜像的存储库之外，[Singularity Hub](https://singularity-hub.org/) 还可以从 _**recipe**_ - 配置为您构建 Singularity 镜像 定义构建映像的步骤的文件。稍后我们将查看recipe和构建镜像。
{: .callout}

让我们首先创建一个 `test` 目录，进入它并从Singularity Hub _pulling_一个测试_Hello World_镜像：

~~~
$ mkdir test
$ cd test
$ singularity pull hello-world.sif shub://vsoch/hello-world
~~~
{: .language-bash}

~~~
INFO:    Downloading shub image
 59.75 MiB / 59.75 MiB [===============================================================================================================] 100.00% 52.03 MiB/s 1s
~~~
{: .output}

刚刚发生了什么？！ 我们使用 `singularity pull` 命令从 Singularity Hub 中提取了一个 SIF 镜像，并指示它使用名称 `hello-world.sif` 将镜像文件存储在当前目录中。 如果您运行`ls`命令，您应该会看到`hello-world.sif`文件现在存在于当前目录中。这是我们的镜像，我们现在可以基于这个镜像运行一个容器：

~~~
$ singularity run hello-world.sif
~~~
{: .language-bash}

~~~
RaawwWWWWWRRRR!! Avocado!
~~~
{: .output}

上面的命令从我们从Singularity Hub下载的镜像中运行_hello-world_容器，并显示了结果输出。

当我们运行它时，容器是如何确定要做什么的？！ 运行容器实际上做了什么导致显示的输出？

当您从 Singularity 映像运行容器而不使用任何其他命令行参数时，容器会运行嵌入在映像中的默认运行脚本。 这是一个 shell 脚本，可用于在容器启动时运行存储在映像中的命令、工具或应用程序。 我们可以使用 `singularity inspect` 命令检查镜像的运行脚本：

~~~
$ singularity inspect -r hello-world.sif
~~~
{: .language-bash}

~~~
#!/bin/sh 

exec /bin/bash /rawr.sh

~~~
{: .output}

这向我们展示了当我们使用`singularity run`命令时配置为默认运行的 `hello-world.sif` 映像中的脚本。

接下来将更详细地介绍正在运行的容器。

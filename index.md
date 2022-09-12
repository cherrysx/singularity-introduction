---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

本课介绍使用[Singularity HPC容器平台](https://github.com/hpcng/singularity)。 Singularity特别适合在用户没有管理权限的基础设施上运行容器，例如高性能计算 (HPC) 集群等共享基础设施。

本课将从头开始介绍 Singularity，向您展示如何运行一个简单的容器并构建您自己的容器并在 HPC 基础架构上运行并行科学工作负载。

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## 先决条件
>  
> 本课有两个核心要素——_运行容器_和_构建容器_。每种方法的先决条件略有不同，下面将对此进行说明。
>
> **正在运行的容器：**（第 1-5 节和第 8 节）
>  
> - 访问预先安装了 Singularity 的本地或远程平台，您可以作为用户访问（即不需要管理员/root 访问权限）。
> - 您将使用的平台还应安装 MPI（第 8 节需要）。
>
> **构建容器：**（第 6 节和第 7 节）
> 构建容器需要访问安装了 Singularity 的平台，您还可以在该平台上进行管理访问。如果您运行 Linux 并且对遵循 [Singularity 安装说明](https://sylabs.io/guides/3.5/admin-guide/installation.html)，那么可以选择直接在您的系统上安装 Singularity。但是，我们强烈建议在这部分中使用 [Docker Singularity 容器](https://quay.io/repository/singularity/singularity?tab=tags)。课程材料的相关部分提供了有关如何使用容器的详细信息。因此，为了支持构建容器，先决条件是：
>
> - 访问安装了Docker的系统，您可以在该系统上运行 Docker Singularity 容器。
>
> 或
>
> - 访问本地或远程基于 Linux 的系统，在该系统上您具有管理员 (root) 访问权限并且可以安装 Singularity 软件。
>
{: .prereq}

{% include links.md %}

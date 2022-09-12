---
title: "Singularity使用Docker镜像"
teaching: 5
exercises: 10
questions:
- "如何将Docker镜像与 Singularity 一起使用？"
objectives:
- "了解如何基于Docker镜像运行Singularity容器。"
keypoints:
- "Singularity可以从Docker镜像启动一个容器，该镜像可以直接从Docker Hub拉取。"
---

## Singularity使用Docker镜像

Singularity还可以直接从Docker镜像启动容器，从而可以访问 [Docker Hub](https://hub.docker.com/) 和其他注册表上可用的大量现有容器镜像。

虽然Singularity实际上并没有使用Docker镜像运行容器（它首先将其转换为适合 Singularity 使用的格式），但所使用的方法为最终用户提供了无缝体验。当您指示Singularity基于拉取Docker镜像运行容器时，Singularity会拉取组成Docker镜像的切片或_layers_，并将它们转换为单文件 Singularity SIF 镜像。

例如，从我们目前看到的简单的_Hello World_示例继续，让我们拉出 [官方Docker Python images](https://hub.docker.com/_/python) 之一。我们将使用带有标签“3.9.6-slim-buster”的图像，它在 Debian的[Buster](https://www.debian.org/releases/buster/) (v10) Linux 上安装了 Python 3.9.6分配：

~~~
$ singularity pull python-3.9.6.sif docker://python:3.9.6-slim-buster
~~~
{: .language-bash}

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 33847f680f63 done  
Copying blob b693dfa28d38 done  
Copying blob ef8f1a8cefd1 done  
Copying blob 248d7d56b4a7 done  
Copying blob 478d2dfa1a8d done  
Copying config c7d70af7c3 done  
Writing manifest to image destination
Storing signatures
2021/07/27 17:23:38  info unpack layer: sha256:33847f680f63fb1b343a9fc782e267b5abdbdb50d65d4b9bd2a136291d67cf75
2021/07/27 17:23:40  info unpack layer: sha256:b693dfa28d38fd92288f84a9e7ffeba93eba5caff2c1b7d9fe3385b6dd972b5d
2021/07/27 17:23:40  info unpack layer: sha256:ef8f1a8cefd144b4ee4871a7d0d9e34f67c8c266f516c221e6d20bca001ce2a5
2021/07/27 17:23:40  info unpack layer: sha256:248d7d56b4a792ca7bdfe866fde773a9cf2028f973216160323684ceabb36451
2021/07/27 17:23:40  info unpack layer: sha256:478d2dfa1a8d7fc4d9957aca29ae4f4187bc2e5365400a842aaefce8b01c2658
INFO:    Creating SIF file...
~~~
{: .output}

请注意我们如何看到Singularity说它是“_Converting OCI blobs to SIF format_”。 然后，我们看到Docker镜像的各个层被下载并解压缩并写入单个 SIF 文件。 该过程完成后，我们应该会在当前目录中看到 python-3.9.6.sif 图像文件。

我们现在可以从这个镜像运行一个容器，就像我们使用任何其他Singularity镜像一样。

> ## 运行我们刚刚从 Docker Hub 中提取的 Python 3.9.6 镜像
>
> 尝试运行 Python 3.9.6 镜像。 发生什么了？
>
> 尝试运行一些简单的 Python 语句...
> 
> > ## 运行 Python 3.9.6 镜像
> >
> > ~~~
> > $ singularity run python-3.9.6.sif
> > ~~~
> > {: .language-bash}
> > 
> > 这应该让您直接进入正在运行的容器中的 Python 交互式 shell：
> > 
> > ~~~
> > Python 3.9.6 (default, Jul 22 2021, 15:24:21) 
> > [GCC 8.3.0] on linux
> > Type "help", "copyright", "credits" or "license" for more information.
> > >>> 
> > ~~~
> > 现在尝试运行一些简单的 Python 语句：
> > ~~~
> > >>> import math
> > >>> math.pi
> > 3.141592653589793
> > >>> 
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

除了运行容器并让它运行默认运行脚本之外，您还可以启动一个运行 shell 的容器，以防您想在运行 Python 之前进行任何配置。 这将在以下练习中介绍：

> ## 在 Python容器中打开一个 shell
>
> 尝试在基于`python-3.9.6.sif` 镜像的容器中运行 shell。 也就是说，运行一个打开 shell 的容器，而不是我们上面看到的默认 Python 交互式控制台。
> 看看你是否能找到不止一种方法来实现这一目标。
>
> 在 shell 中，尝试启动 Python 交互式控制台并运行一些 Python 命令。
> 
> > ## 解决方案
> >
> > 回想一下前面的材料，我们可以使用 `singularity shell` 命令打开容器内的外壳。 要在基于 `python-3.9.6.sif` 镜像的容器中打开常规 shell，我们可以简单地运行：
> > ~~~
> > $ singularity shell python-3.9.6.sif
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Singularity> echo $SHELL
> > /bin/bash
> > Singularity> cat /etc/issue
> > Debian GNU/Linux 10 \n \l
> > 
> > Singularity> python
> > Python 3.9.6 (default, Jul 22 2021, 15:24:21) 
> > [GCC 8.3.0] on linux
> > Type "help", "copyright", "credits" or "license" for more information.
> > >>> print('Hello World!')
> > Hello World!
> > >>> exit()
> > 
> > Singularity> exit
> > $ 
> > ~~~
> > {: .output}
> > 
> > 也可以使用 `singularity exec` 命令在容器中运行可执行文件。 因此，我们可以使用 `exec` 命令运行 `/bin/bash`：
> > 
> > ~~~
> > $ singularity exec python-3.9.6.sif /bin/bash
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Singularity> echo $SHELL
> > /bin/bash
> > ~~~
> > {: .output}
> > 
> > 您只需运行“python”命令即可从容器shell运行 Python 控制台。
> {: .solution}
{: .challenge}

## References

\[1\] Gregory M. Kurzer, Containers for Science, Reproducibility and Mobility: Singularity P2. Intel HPC Developer Conference, 2017. Available at: https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf

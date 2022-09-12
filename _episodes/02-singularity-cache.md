---
title: "Singularity缓存"
teaching: 10
exercises: 0
questions:
- "为什么 Singularity 使用本地缓存？"
- "Singularity 在哪里存储镜像？"
objectives:
- "了解 Singularity 的镜像缓存。"
- "了解如何管理本地存储的 Singularity 镜像。"
keypoints:
- "Singularity 缓存下载的镜像，以便在使用“singularity pull”命令请求时不会再次下载未更改的镜像。"
- "您可以通过删除所有本地缓存的镜像或指定要删除的单个镜像来释放缓存中的空间。"
---

## Singularity的镜像缓存

虽然 Singularity 没有像 Docker 那样的本地镜像存储库，但它会缓存下载的镜像文件。正如我们在上一集中看到的，镜像只是存储在本地磁盘上的“.sif”文件。

如果您删除从远程映像存储库中拉取的本地“.sif”映像，然后再次拉取它，如果该映像与您之前拉取的版本没有变化，您将获得本地映像文件的副本缓存而不是从远程源再次下载镜像。 这消除了不必要的网络传输，对于可能需要一些时间通过网络传输的大型镜像特别有用。为了证明这一点，删除存储在 `test` 目录中的 `hello-world.sif` 文件，然后再次发出 `pull` 命令：

~~~
$ rm hello-world.sif
$ singularity pull hello-world.sif shub://vsoch/hello-world
~~~
{: .language-bash}

~~~
INFO:    Use image from cache
~~~
{: .output}

正如我们在上面的输出中看到的那样，镜像已经从缓存中返回，我们看不到我们之前看到的输出：显示正在从 Singularity Hub 下载镜像。

我们如何知道本地缓存中存储了什么？ 我们可以使用 `singularity cache` 命令找出：

~~~
$ singularity cache list
~~~
{: .language-bash}

~~~
There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

这告诉我们缓存中存储了多少容器文件以及缓存使用了多少磁盘空间，但它并没有告诉我们实际存储的是什么。 要了解更多信息，我们可以将 `-v` 详细标志添加到 `list` 命令：

~~~
$ singularity cache list -v
~~~
{: .language-bash}

~~~
NAME                     DATE CREATED           SIZE             TYPE
hello-world_latest.sif   2020-04-03 13:20:44    62.65 MB         shub

There are 1 container file(s) using 62.65 MB and 0 oci blob file(s) using 0.00 kB of space
Total space used: 62.65 MB
~~~
{: .output}

这为我们提供了一些关于存储在缓存中的实际镜像的更多有用信息。 在 `TYPE` 列中，我们可以看到我们的镜像类型是 `shub`，因为它是从 Singularity Hub 提取的 `SIF` 镜像。

> ## 清理 Singularity 镜像缓存
> 我们可以使用 `singularity cache clean` 命令从缓存中删除镜像。运行不带任何选项的命令将显示警告并要求您确认是否要从缓存中删除所有内容。
>
> 您还可以删除特定镜像或特定类型的所有镜像。查看 `singularity cache clean --help` 的输出以获取更多信息。
{: .callout}

> ## 缓存位置
> 默认情况下，Singularity 使用 `$HOME/.singularity/cache`作为缓存的位置。 您可以通过将 `SINGULARITY_CACHEDIR` 环境变量设置为您要使用的缓存位置来更改缓存的位置。
{: .callout}

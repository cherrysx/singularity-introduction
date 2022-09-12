---
title: "使用Singularity容器运行命令"
teaching: 10
exercises: 5
questions:
- "如何在容器中运行不同的命令？"
- "如何访问容器内的交互式shell？"
objectives:
- "了解如何在启动容器时运行不同的命令。"
- "了解如何在容器环境中打开交互式shell。"
keypoints:
- "`singularity exec` 是 `singularity run` 的替代方案，它允许您启动运行特定命令的容器。"
- "`singularity shell` 命令可用于启动容器并在其中运行交互式 shell。"
---

## 在容器中运行特定命令

我们之前看到，我们可以使用 `singularity inspect` 命令查看容器配置为默认运行的运行脚本。如果我们想在容器中运行不同的命令怎么办？

如果我们知道要在容器中运行的可执行文件的路径，我们可以使用`singularity exec`命令。 例如，使用我们已经从 Singularity Hub 中提取的 `hello-world.sif`容器，我们可以在 `hello-world.sif`文件所在的`test`目录中运行以下命令：

~~~
$ singularity exec hello-world.sif /bin/echo Hello World!
~~~
{: .language-bash}

~~~
Hello World!
~~~
{: .output}

在这里，我们看到一个容器已经从`hello-world.sif`镜像启动，并且`/bin/echo`命令已经在容器中运行，并传递了输入 `Hello World!`。 该命令已将提供的输入回显到控制台，并且容器已终止。

请注意，`singularity exec`的使用已经覆盖了镜像元数据中设置的任何运行脚本，并且我们指定为 `singularity exec` 的参数的命令已被运行。

> ## 基本练习：在“hello-world”容器中运行不同的命令
>
> 你能运行一个基于`hello-world.sif`镜像的容器，**打印当前日期和时间**吗？
> 
> > ## 解决方案
> >
> > ~~~
> > $ singularity exec hello-world.sif /bin/date
> > ~~~
> > {: .language-bash}
> > 
> > ~~~
> > Fri Jun 26 15:17:44 BST 2020
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

<br/>

### **`singularity run`和`singularity exec`的区别**

上面我们使用了`singularity exec`命令。 在之前，我们使用了“singularity run”。 为了澄清，这两个命令之间的区别是：

 - `singularity run`：这将根据指定的镜像运行容器的默认命令集。这个默认命令是在构建镜像时在镜像元数据中设置的（我们将在后面看到更多关于这个的内容）。使用 `singularity run` 时无需指定要运行的命令，只需指定镜像文件名即可。 正如我们之前看到的，您可以使用 `singularity inspect` 命令查看在基于映像启动新容器时默认运行的命令。

 - `singularity exec`：这将基于指定的镜像启动一个容器，并运行`singularity exec <镜像文件名>`后面的命令行提供的命令。这将覆盖在镜像元数据中指定的任何默认命令，如果您使用“singularity run”，这些默认命令将被运行。

## 在容器中打开交互式shell

如果要在容器中打开交互式 shell，Singularity 提供了`singularity shell` 命令。 同样，使用 `hello-world.sif` 映像，在我们的 `test` 目录中，我们可以在 hello-world 映像的容器中运行 shell：

~~~
$ singularity shell hello-world.sif
~~~
{: .language-bash}

~~~
Singularity> whoami
[<your username>]
Singularity> ls
hello-world.sif
Singularity> 
~~~
{: .output}

如上所示，我们在一个从 `hello-world.sif` 映像启动的新容器中打开了一个 shell。 请注意，shell 提示已更改为显示我们现在位于 Singularity 容器中。

> ## 讨论：在 Singularity 容器中运行 shell
>
> Q: 关于在 Singularity 容器 shell 中输入的上述命令的输出，您注意到了什么？
>  
> Q: 这与您在 Docker 容器中看到的不同吗？
{: .discussion}

使用 `exit` 命令退出容器shell。

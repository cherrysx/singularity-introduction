---
title: "使用Singularity容器运行MPI并行作业"
teaching: 30
exercises: 40
questions:
# - "Can I run MPI parallel codes from Singularity containers on a local/institutional/national HPC platform?"
- "如何从Singularity容器设置和运行MPI作业？"
objectives:
- "了解Singularity容器中的MPI应用程序如何在HPC平台上运行"
- "了解通过 Singularity 运行 MPI 作业时的挑战和相关的性能影响"
keypoints:
- "如果两个平台具有兼容的 MPI 实现，则可以在一个平台上构建包含 MPI 应用程序的Singularity镜像，然后在另一个平台（例如 HPC 集群）上运行。"
- "在 Singularity 容器中运行 MPI 应用程序时，使用主机系统上的 MPI 可执行文件为每个进程启动 Singularity 容器。"
- "考虑并行应用程序的性能要求以及构建/运行镜像的位置可能会影响到这一点。"
---

## 使用 Singularity 容器运行 MPI 并行代码

### MPI 概述

MPI - [消息传递接口](https://en.wikipedia.org/wiki/Message_Passing_Interface) - 是一种广泛使用的并行编程标准。 它用于在并行应用程序中的进程之间交换消息/数据。如果您曾参与开发或使用计算科学软件，您可能已经熟悉 MPI 和运行 MPI 应用程序。

在大型集群上使用 MPI 代码时，一种常见的方法是自己编译代码，在集群平台上您自己的用户目录中，针对集群上支持的 MPI 实现进行构建。 或者，如果代码在集群上被广泛使用，平台管理员可以将应用程序构建并打包为一个模块，以便集群的所有用户都可以轻松访问它。

### 带有Singularity容器的 MPI 代码

我们已经看到，如果没有 root 访问权限，构建 Singularity 容器可能是不切实际的。由于我们极不可能在大型机构、区域或国家集群上拥有 root 访问权限，因此直接在目标平台上构建容器通常不是一种选择。

如果我们的目标平台使用 [OpenMPI](https://www.open-mpi.org/)，这是两个广泛使用的源 MPI 实现之一，我们可以在本地构建平台上构建/安装兼容的 OpenMPI 版本，或者直接在镜像中作为镜像构建过程的一部分。然后，我们可以在镜像沙箱中以交互方式或通过定义文件构建需要 MPI 的代码。

如果目标平台使用基于 [MPICH](https://www.mpich.org/) 的 MPI 版本，另一个广泛使用的开源 MPI 实现，则存在 [MPICH 与其他几种 MPI 实现之间的 ABI 兼容性]（ https://www.mpich.org/abi/）。在这种情况下，您可以在本地平台、镜像沙箱中或作为镜像构建过程的一部分通过定义文件构建 MPICH 和您的代码，并且您应该能够在目标集群上成功运行基于此镜像的容器平台。

如 Singularity 的 [MPI 文档](https://sylabs.io/guides/3.5/user-guide/mpi.html) 中所述，提供了对 OpenMPI 和 MPICH 的支持。给出了通过定义文件从源代码构建相关 MPI 版本的说明，我们将在下面的示例中看到它的使用。

#### **HPC平台上的容器可移植性和性能**

虽然在本地系统上构建旨在用于远程HPC 平台的容器确实提供了一定程度的可移植性，但如果您追求最佳性能，它可能会出现一些问题。如果要实现最佳性能，则需要构建和配置容器中的 MPI 版本以支持目标平台上的硬件。如果平台具有带有专有驱动程序的专用硬件，则在具有不同硬件的不同平台上构建意味着不可能使用正确的驱动程序支持来构建以获得最佳性能。如果可用的 MPI 版本不同（但兼容），则尤其如此。 Singularity 的 [MPI 文档](https://sylabs.io/guides/3.5/user-guide/mpi.html) 重点介绍了使用 MPI 代码的两种不同模型。我们将在此处查看的 _[混合模型](https://sylabs.io/guides/3.5/user-guide/mpi.html#hybrid-model)_ 涉及使用 MPI 安装中的 MPI 可执行文件主机系统启动Singularity并在容器内运行应用程序。容器中的应用程序链接并使用容器内的 MPI 安装，而容器内的 MPI 安装又与主机系统上运行的 MPI 守护进程通信。在下一节中，我们将了解如何构建一个包含小型 MPI 应用程序的 Singularity 镜像，然后可以使用混合模型运行该应用程序。

### 为 MPI 代码构建和运行 Singularity 镜像

#### **构建和测试镜像**

此示例假设您将在本地平台上构建容器镜像，然后将其部署到具有不同但兼容的 MPI 实现的集群。有关其工作原理的更多信息，请参阅 Singularity 文档中的 [Singularity 和 MPI 应用程序](https://sylabs.io/guides/3.5/user-guide/mpi.html#singularity-and-mpi-applications)。

我们将从定义文件构建镜像。基于此镜像的容器将能够使用 [OSU Micro-Benchmarks](https://mvapich.cse.ohio-state.edu/benchmarks/) 软件运行 MPI 基准测试。

在此示例中，目标平台是使用 [Intel MPI](https://software.intel.com/content/www/us/en/develop/tools/mpi-library.html) 的远程 HPC 集群。可以通过我们在 Singularity 材料的上一集中使用的 Singularity Docker 镜像来构建容器。

首先创建一个目录，然后在该目录中从 [OSU 微基准页面](https://mvapich.cse.ohio-state.edu/benchmarks/) 下载并保存 OSU 微基准 5.7.1 版的“tarball”和来自 [MPICH 下载页面](https://www.mpich.org/downloads/) 的 MPICH 版本 3.4.2。

在同一目录下，将以下定义文件内容保存到 `.def` 文件中，例如`osu_benchmarks.def`：

~~~
Bootstrap: docker
From: ubuntu:20.04

%files
    /home/singularity/osu-micro-benchmarks-5.7.1.tgz /root/
    /home/singularity/mpich-3.4.2.tar.gz /root/

%environment
    export SINGULARITY_MPICH_DIR=/usr
    export OSU_DIR=/usr/local/osu/libexec/osu-micro-benchmarks/mpi

%post
    apt-get -y update && DEBIAN_FRONTEND=noninteractive apt-get -y install build-essential libfabric-dev libibverbs-dev gfortran
    cd /root
    tar zxvf mpich-3.4.2.tar.gz && cd mpich-3.4.2
    echo "Configuring and building MPICH..."
    ./configure --prefix=/usr --with-device=ch3:nemesis:ofi && make -j2 && make install
    cd /root
    tar zxvf osu-micro-benchmarks-5.7.1.tgz
    cd osu-micro-benchmarks-5.7.1/
    echo "Configuring and building OSU Micro-Benchmarks..."
    ./configure --prefix=/usr/local/osu CC=/usr/bin/mpicc CXX=/usr/bin/mpicxx
    make -j2 && make install

%runscript
    echo "Rank ${PMI_RANK} - About to run: ${OSU_DIR}/$*"
    exec ${OSU_DIR}/$*
~~~
{: .output}

快速概览上述定义文件的作用：

- 镜像从`ubuntu:20.04`Docker 镜像引导。
- 在 `%files` 部分：OSU Micro-Benchmarks 和 MPICH tar 文件从当前目录复制到镜像中的 `/root` 目录。
- 在 `%environment` 部分：设置几个环境变量，这些变量将在从生成的镜像运行的所有容器中可用。
- 在“%post”部分：
- Ubuntu 的 `apt-get` 包管理器用于更新包目录，然后安装 MPICH 构建所需的编译器和其他库。
- 提取 MPICH .tar.gz 文件并运行配置、构建和安装步骤。请注意使用 `--with-device` 选项来配置 MPICH 以使用正确的驱动程序来支持高性能集群上改进的通信性能。
- 提取 OSU Micro-Benchmarks .tar.gz 文件并运行配置、构建和安装步骤以从源代码构建基准代码。
- 在 `%runscript` 部分：设置了一个运行脚本，它将回显当前进程的排名号，然后运行作为命令行参数提供的命令。

_请注意，要运行的可执行文件的基本路径（`$OSU_DIR`）在运行脚本中是硬编码的_。然后，您在基于镜像运行容器实例时提供的命令行参数将添加到此基本路径中。示例命令行参数包括：`startup/osu_hello`、`collective/osu_allgather`、`pt2pt/osu_latency`、`one-sided/osu_put_latency`。

> ## 构建和测试OSU微基准镜像
>
> 使用上述定义文件，构建一个名为 `osu_benchmarks.sif` 的 Singularity 镜像。
>
> 镜像构建完成后，通过运行 `startup` 基准文件夹中的 `osu_hello` 基准对其进行测试。
>
> _注意：如果您不使用 Singularity Docker 镜像来构建您的 Singularity 镜像，则需要在定义文件的 `%files` 部分中编辑 .tar.gz 文件的路径。_
>  
> > ## 解决方案
> > 
> > 您应该能够从定义文件构建镜像，如下所示：
> > 
> > ~~~
> > $ singularity build osu_benchmarks.sif osu_benchmarks.def
> > ~~~
> > {: .language-bash}
> >
> > _请注意，如果您直接从命令行运行 Singularity Docker容器来进行构建，您需要提供 `.def`文件的完整路径**在**容器内_ - 这很可能这将与主机系统上的文件路径不同。例如，如果您已将本地系统上包含该文件的目录挂载到容器内的 `/home/singularity`，则 `.def` 文件的完整路径将是`/home/singularity/osu_benchmarks.def `。
> >
> > 假设镜像构建成功，您可以尝试在本地运行容器，并将SIF文件传输到您可以访问的集群平台（安装了 Singularity）并在那里运行它。
> >
> > 让我们从_您的本地系统_（您构建容器的位置）上的 `startup/osu_hello` 的单进程运行开始，以确保我们可以按预期运行容器。我们将使用容器内的MPI。 _请注意，当我们在HPC集群平台上运行并行作业时，我们使用集群上安装的 MPI来协调运行，所以情况有些不同......_
> >
> > 根据你的镜像在 Singularity 容器中启动一个 shell，然后通过 `mpirun` 运行一个单进程作业：
> > 
> > ~~~
> > $ singularity shell --contain /home/singularity/osu_benchmarks.sif
> > Singularity> mpirun -np 1 $OSU_DIR/startup/osu_hello
> > ~~~
> > {: .language-bash}
> > 
> > 您应该会看到类似于以下内容的输出：
> > 
> > ~~~
> > # OSU MPI Hello World Test v5.7.1
> > This is a test with 1 processes
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

#### **通过 MPI 运行 Singularity 容器**

假设上述测试有效，我们现在可以尝试在我们的容器镜像中并行运行一个 OSU 基准测试工具。

这就是事情变得有趣的地方，我们将从查看 Singularity 容器如何在 MPI 环境中运行开始。

如果您熟悉运行 MPI 代码，您就会知道您使用 `mpirun`（就像我们在前面的示例中所做的那样）、`mpiexec` 或类似的 MPI 可执行文件来启动您的应用程序。该可执行文件可以直接在您正在使用的本地系统或集群平台上运行，或者您可能需要通过提交给作业调度程序的作业脚本来运行它。您的基于 MPI 的应用程序代码将与 MPI 库链接，将对这些 MPI 库进行 MPI API 调用，这些 MPI 库又与在主机系统上运行的 MPI 守护进程对话。该守护进程处理 MPI 进程之间的通信，包括根据需要与其他节点上的守护进程对话以在不同机器上运行的进程之间交换信息。

在 Singularity 容器中运行代码时，我们不使用存储在容器中的 MPI 可执行文件（即，我们不运行 `singularity exec mpirun -np <numprocs> /path/to/my/executable`）。相反，我们使用主机系统上安装的MPI来运行 Singularity，并从每个 MPI 进程的容器内启动可执行文件的实例。如果 MPI 实现中没有 Singularity 支持，这将导致在每个进程中启动一个单独的 Singularity 容器实例。如果在主机上运行大量进程，这可能会产生一些开销。在 MPI 实现中内置Singularity支持的情况下，这可以解决这个潜在问题，并减少作为MPI作业的一部分从容器内运行代码的开销。

最终，这意味着我们从容器内正在运行的 MPI 代码的链接到 MPI 库，而这些库又与主机系统上的 MPI 守护程序通信，该主机系统是主机系统 MPI 安装的一部分。在 MPICH 的情况下，这两种 MPI 的安装可能不同，但只要您的容器镜像和主机系统安装的 MPI 版本之间存在 [ABI 兼容性](https://wiki.mpich.org/mpich/index.php/ABI_Compatibility_Initiative)，您的作业应该可以成功运行。

我们现在可以尝试运行进程2个MPI运行点对点基准测试“osu_latency”的。如果您的本地系统同时安装了MPI 和 Singularity并且具有多个内核，则可以在该系统上运行此测试。或者，您可以在集群上运行。请注意，如果您在集群上运行，您可能需要通过提交作业到调度器的作业提交脚本来提交此命令。

> ## 进行`osu_latency` 基准测试的并行运行（一般示例）
>
> 将 `osu_benchmarks.sif` Singularity镜像移动到集群（或其他合适的）平台上，您将在其中进行基准测试。
>
> 您应该能够使用类似于下图的命令运行基准测试。但是，如果您在集群上运行，此时您可能需要编写并提交作业提交脚本以启动基准测试。
>
> ~~~
> $ mpirun -np 2 singularity run osu_benchmarks.sif pt2pt/osu_latency
> ~~~
> {: .language-bash}
> 
> > ## 预期输出和讨论
> >
> > 正如您在上面显示的 mpirun 命令中看到的，我们在主机系统上调用了`mpirun`并将`singularity`可执行文件传递给MPI，其参数是镜像文件和我们想要传递给镜像的运行脚本，在这种情况下是要运行的基准可执行文件的路径/名称。
> >
> > 下面显示了您应该看到的输出示例。 您应该为最大 4MB 的消息显示延迟值。
> > 
> >~~~
> > Rank 1 - About to run: /.../mpi/pt2pt/osu_latency
> > Rank 0 - About to run: /.../mpi/pt2pt/osu_latency
> > # OSU MPI Latency Test v5.6.2
> > # Size          Latency (us)
> > 0                       0.38
> > 1                       0.34
> > ...
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}


> ## 并行运行 `osu_latency` 基准测试
>
> 这个版本的练习，用于使用包含 MPI 构建的 Singularity 容器并行运行 osu_latency 基准测试，特定于本课程的运行。
>
> 此处提供的信息专门针对您可以访问的 HPC 平台进行此教学版本的课程。
>
> 将 `osu_benchmarks.sif` Singularity镜像移动到要进行基准测试的集群上。您应该使用 `scp` 或类似的实用程序来复制文件。
>
> 为您提供访问权限的平台使用“Slurm”来安排作业在平台上运行。您现在需要创建一个“Slurm”作业提交脚本来运行基准测试。
>
> 下载此 [模板脚本]({{site.url}}{{site.baseurl}}/files/osu_latency.slurm.template) 并对其进行编辑以适合您的配置。
>
> 使用 `sbatch` 命令将修改后的作业提交脚本提交给 `Slurm` 调度程序。
> 
> ~~~
> $ sbatch osu_latency.slurm
> ~~~
> {: .language-bash}
> 
> > ## 预期输出和讨论
> >
>> 正如您将在使用提供的模板作业提交脚本的命令中看到的那样，我们在主机系统上调用了 `mpirun` 并将其传递给 MPI 的 `singularity` 可执行文件，其参数是镜像文件和我们的任何参数 想要传递给镜像的运行脚本。 在这种情况下，参数是要运行的基准可执行文件的路径/名称。
> >
> > 下面显示了您应该看到的输出示例。 您应该为最大 4MB 的消息显示延迟值。
> > 
> >~~~
> > INFO:    Convert SIF file to sandbox...
> > INFO:    Convert SIF file to sandbox...
> > Rank 1 - About to run: /.../mpi/pt2pt/osu_latency
> > Rank 0 - About to run: /.../mpi/pt2pt/osu_latency
> > # OSU MPI Latency Test v5.6.2
> > # Size          Latency (us)
> > 0                       1.49
> > 1                       1.50
> > 2                       1.50
> > ...
> > 4194304               915.44
> > INFO:    Cleaning up image...
> > INFO:    Cleaning up image...
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

这表明我们可以在 Singularity 容器中成功运行并行 MPI 可执行文件。 但是，在这种情况下，这两个进程几乎肯定会在同一个物理节点上运行，因此这并不是在测试节点之间互连的性能。

您现在可以尝试运行更大规模的测试。 您还可以尝试运行使用多个进程的基准测试，例如尝试 `collective/osu_gather`。

> ## 调查使用构建在本地系统上并在集群上运行的容器镜像时的性能
>
> 要了解 Singularity 镜像中的代码与在目标 HPC 平台上本地构建的相同代码之间的性能差异，请尝试在集群本地从源代码构建 OSU 基准。 然后尝试运行通过Singularity容器运行的相同基准。 查看运行 `collective/osu_gather` 或其他集体基准测试时获得的输出，以了解是否存在性能差异及其重要性。
>
> 尝试运行足够多的进程，使这些进程分布在不同的物理节点上，以便您利用集群的网络互连。
>
> 你看到了什么？
> 
> > ## 讨论
> >  
> > 您可能会发现直接在 HPC 平台上构建的代码版本的性能明显更好。 或者，两个版本之间的性能可能相似。
> >
> > 代码的两个构建之间的性能差异有多大？
> >
> > 什么可能导致两个版本的代码之间的性能差异？
> > 
> {: .solution}
{: .challenge}

如果您想通过 Singularity 运行代码，性能对您来说是个问题，建议您使用 _[bind 模型](https://sylabs.io/guides/3.5/user-guide /mpi.html#bind-model)_ 用于通过 Singularity 构建/运行 MPI 应用程序。

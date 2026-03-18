# 满足现状的 Distroless 容器的谬误

> 原文：[`towardsdatascience.com/the-fallacy-of-complacent-distroless-containers-8b09bd3ad55a/`](https://towardsdatascience.com/the-fallacy-of-complacent-distroless-containers-8b09bd3ad55a/)

![由 Leonardo AI 生成的图像](img/38cc1d9a8790a87a89a8a75f406b92e9.png)

由 Leonardo AI 生成的镜像

* * *

构建 [Docker 镜像](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/) 是一种 [简单易行的实践](https://medium.com/better-programming/the-whole-shebang-dockerfiles-5d59ace94d28)，然而，完美地构建它们仍然是一门难以掌握的艺术。为了追求最小、最安全且功能齐全的容器镜像，开发者们面临着涉及复杂工具、深入的分发知识以及易出错的裁剪策略的 [distroless](https://github.com/GoogleContainerTools/distroless) 实践。实际上，这些实践往往忽略了包管理器的使用，这导致了安全漏洞，因为大多数漏洞扫描器依赖于包管理器元数据来检测容器镜像中的软件组件。

## 构建容器镜像

当您构建容器镜像时，您正在将应用程序及其依赖项打包成一个可移植的软件单元，稍后可以独立部署，无需虚拟化整个操作系统。

构建容器镜像现在实际上是一种非常容易接触到的实践。有大量工具（例如 [Docker](https://docs.docker.com/reference/dockerfile/)、[Rockcraft](https://documentation.ubuntu.com/rockcraft/en/stable/)、[Buildah](https://buildah.io/)…）专门为此目的而设计。

**但是**，在打包您的应用程序及其运行所需的一切的过程中，您是否可能添加了不需要的东西？

大多数情况下，答案是 **是的**！

这里有一个非常简单的 Dockerfile：

```py
FROM ubuntu:24.04

RUN apt update &amp;&amp; apt install -y --no-install-recommends nginx 
  &amp;&amp; rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

在这个例子中，我们是在 Ubuntu 24.04 图像之上打包 [Nginx](https://nginx.org/en/)。

+   `ubuntu:24.04` 将在我们的最终镜像中。我们实际上需要它吗？很可能不需要。有了它，一大堆不必要的软件（例如 `apt` 这样的实用工具）将被保留，从而增加了镜像的攻击面；

+   尽管我们小心地没有安装推荐项并清理 `apt` 列表，但我们仍然安装了整个 Nginx 软件包及其所有依赖项。我们需要所有这些吗？这个问题比较棘手，因为它很大程度上取决于使用场景，但我们确实知道我们不想有像 Nginx 的 man 页面这样的东西。

## Distroless 容器

> "[Distroless](https://github.com/GoogleContainerTools/distroless)" 镜像仅包含您的应用程序及其运行时依赖项。

它们不包含来自 Linux 分发的典型附加库或实用工具。

这在过去 7 年的容器安全领域一直是最推崇的做法。尽管在概念上是正确的，但构建这些更小且“更安全（？）”的无发行版容器有什么成本？

+   **容易构建吗？**并不真的容易。这可能是一项需要使用专用工具并需要深入了解发行版以有效“去除发行版”的艰难工艺。

+   **容易出错吗？**是的。一些最推崇的构建无发行版镜像的策略涉及采用“自上而下”的方法——即用你的应用程序膨胀基础容器，然后**手动**将所需内容挑选到“scratch”环境中。

## 相关性不等于因果关系

并不是因为你的容器更小，它就一定会更安全！事实上，构建无发行版容器容易产生**盲点**。

Yotam Perkal 的 [2022 Rezilion 报告](https://www.linkedin.com/posts/yotam-perkal_scanning-the-scanners-vulnerability-scanner-activity-7086727226372587521-pd94?utm_source=share&utm_medium=member_desktop) 通过扫描 20 个最受欢迎的容器镜像并比较结果漏洞报告，测试了不同漏洞扫描器的可靠性和一致性。除了大量的高和关键误识别外，报告还显示这些工具的平均精确度为 82%，其中很大一部分结果由**假阳性**和**假阴性**组成。

说实话，我对**假阳性**可以接受——就像被告知你生病了，而实际上只是检查错误——虽然很可怕，但并不真正危险。

另一方面，**假阴性**要糟糕得多！就像有一个你不知道的问题——**一个盲点**！

## 安全盲点的根本原因

防漏洞扫描器无法检测某些漏洞的主要原因之一是，它们中的大多数**依赖于包元数据**，因此**无法检测到由包管理器未管理的软件组件**。

你不相信我？让我来给你展示。

为了演示目的，我们只需从 Docker Hub 选取一个流行且存在漏洞的 Docker 镜像，以及一个流行的漏洞扫描器。

让我们说：

+   [Trivy](https://trivy.dev/latest/) 作为扫描器，

+   `[ubuntu:lunar](https://hub.docker.com/layers/library/ubuntu/lunar/images/sha256-ea1285dffce8a938ef356908d1be741da594310c8dced79b870d66808cb12b0f)` 作为 Docker 镜像。

在撰写本文时，所选的 Docker 镜像已经过时，并且存在漏洞。根据 Trivy 的分析，这个镜像总共有 11 个 CVE：

```py
$ trivy image ubuntu:lunar
...
ubuntu:lunar (ubuntu 23.04)

Total: 11 (UNKNOWN: 0, LOW: 2, MEDIUM: 9, HIGH: 0, CRITICAL: 0)
```

但是，这是一个基于 Debian 的容器镜像，那么如果我们删除镜像的包元数据，Trivy 会说什么呢？让我们看看…

```py
$ echo '''
FROM ubuntu:lunar

# Whiteout the dpkg status file
RUN rm /var/lib/dpkg/status
''' | docker build -t ubuntu:lunar-tampered -
```

鼓掌吧… 🥁

```py
$ trivy image ubuntu:lunar-tampered
...
ubuntu:lunar-tampered (ubuntu 23.04)

Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```

零，零，什么都没有…没有漏洞！看起来是这样。但我们知道仍有 11 个 CVE。我们只是删除了 Trivy 扫描所依赖的包元数据。

* * *

## 现在怎么办？

漏洞扫描器的工作方式不同，它们可能依赖于容器镜像本身的信息来生成准确的报告！

因此，这里有一个你可以使用的清单，以确保你构建和使用的容器没有携带隐藏的漏洞：

1.  并不是因为容器小巧且无分发系统，你计划使用的容器就是安全的。

1.  看看超越漏洞扫描器。正如我们在上面的例子中看到的，一个缺失的文件可能导致扫描器无法识别 CVE。所以，不要对此视而不见！是的，使用扫描器！但也要尝试寻找可能存在的盲点的线索。如何？

    +   一些扫描器，如 Trivy，实际上会在它们依赖的文件（如上面的`dpkg/status`）缺失时发出警告。例如，Trivy 会说：| *WARN 没有检测到操作系统包。请确保您没有删除包含有关已安装包信息的任何文件 | WARN 例如，位于"/lib/apk/db/"、"/var/lib/dpkg/"和"/var/lib/rpm"下的文件 –* 一些工具，包括 Trivy，也可以生成 SBOM。这是一种更用户友好的方式来双重检查镜像的软件组件。所以尝试生成一个 SBOM（例如，`trivy image --format spdx-json --output result.json <yourImage>`）。这个 SBOM 是空的吗？它是否缺少您期望在该镜像中看到的组件？如果是这样，那么漏洞扫描器很可能也无法生成准确的报告。

1.  漏洞扫描器各不相同，所以不要只依赖一个扫描器。尝试选择那些对您想要使用的镜像中包含的软件生态系统有更好支持的扫描器。

1.  在构建容器时避免“死点”。也就是说，选择使您的应用程序工作的最小文件集可能听起来很有吸引力，但您可能无意中遗漏了扫描器正常工作的必要元数据。

1.  关于上述内容，优先考虑使用包管理器。是的，其中一些可能并不真正适合与最小化容器一起工作，但正如我们上面看到的，它们产生的某些元数据对于进行适当的安全扫描是至关重要的。

* * *

在即将发表的这篇文章中，我将探讨一些构建最小化容器镜像的技术，从典型的多阶段方法到对无分发系统友好的工具，如[Chisel](https://ubuntu.com/containers/chiselled)。

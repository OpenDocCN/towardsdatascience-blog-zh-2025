# Kubernetes——有效理解和利用探测

> 原文：[`towardsdatascience.com/kubernetes-understanding-and-utilizing-probes-effectively/`](https://towardsdatascience.com/kubernetes-understanding-and-utilizing-probes-effectively/)

## 简介

让我们谈谈 Kubernetes 探测以及为什么它们在您的部署中很重要。在管理面向生产的容器化应用程序时，即使是小的优化也可能带来巨大的好处。

为了减少部署时间，使您的应用程序更好地响应扩展事件，并管理运行 Pod 的健康状况，需要微调您的容器生命周期管理。这正是为什么对 Kubernetes 探测的正确配置和实施对于任何关键部署至关重要。它们帮助您的集群就流量路由、重启和资源分配做出智能决策。

正确配置的探测器可以显著提高应用程序的可靠性，减少部署停机时间，并优雅地处理意外错误。在本文中，我们将探讨 Kubernetes 中可用的三种探测类型以及如何利用它们来配置更健壮的系统。

## 快速回顾

精确了解每个探测的作用以及一些常见的配置模式是至关重要的。它们在容器生命周期中各自发挥作用，当一起使用时，它们为维护应用程序的可用性和性能提供了一个坚如磐石的基础框架。

### 启动：优化启动时间

当由于扩展事件或新的部署启动新 Pod 时，启动探测器会被评估一次。它充当其余容器检查和微调的守门人，这将帮助您的应用程序更好地处理增加的负载或服务降级。

样例配置：

```py
startupProbe:
  httpGet:
    path: /health
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

**关键要点**：

+   保持`periodSeconds`较低，以便探测频繁触发，快速检测成功的部署。

+   将`failureThreshold`增加到足够高的值，以适应最坏情况的启动时间。

启动探测将通过查询配置的路径来检查您的容器是否已启动。它还将**停止触发存活和就绪性探测**，直到成功为止。

### 存活性：检测已死亡的容器

您的存活探测回答了一个非常简单的问题：“这个 Pod 是否仍在正常运行？”如果不是，K8s 将重启它。

样例配置：

```py
livenessProbe:
  httpGet:
    path: /health
    port: 80
  periodSeconds: 10
  failureThreshold: 3
```

**关键要点**：

+   由于 K8s 将完全重启您的容器并启动一个新的容器，因此添加一个`failureThreshold`来对抗间歇性异常。

+   避免使用`initialDelaySeconds`，因为它过于限制性——使用启动探测器代替。

请注意，失败的存活探测将使您当前正在运行的 Pod 崩溃并启动一个新的 Pod，因此请避免使其过于激进——这是下一个要考虑的。

### 就绪性：处理意外错误

就绪性探针确定是否应该开始或继续接收流量。在您的容器失去数据库连接或过度使用且不应接收新请求的情况下，它非常有用。

样本配置：

```py
readinessProbe:
  httpGet:
    path: /health
    port: 80
  periodSeconds: 3
  failureThreshold: 1
  timeoutSeconds: 1
```

**关键要点**：

+   由于这是你阻止流量到达不健康目标的第一道防线，因此使探针更具侵略性并减少`periodSeconds`。

+   将`failureThreshold`保持在最小值，你希望快速失败。

+   超时期间也应保持在最小值，以处理较慢的容器。

+   通过拥有较长的运行`livenessProbe`来给`readinessProbe`充足的时间进行恢复。

就绪性探针确保流量不会到达未准备好的容器，因此它是堆栈中最重要的一项。

## 将所有内容整合在一起

如您所见，尽管所有探针都有其独特的用途，但提高您应用程序弹性策略的最佳方式是同时使用它们。

你的启动探针将帮助你进行扩展场景和新部署，使你的容器能够快速启动。它们只会触发一次，并且只有在成功完成之前才会停止其他探针的执行。

活跃性探针有助于处理因不可恢复错误而死亡的容器，并告诉集群为你启动一个新的、全新的 pod。

就绪性探针是告诉 K8s 何时一个 pod 应该接收流量或不接收流量的探针。在处理间歇性错误或高资源消耗导致响应时间较慢的情况下，它可以非常有用。

## 其他配置

探针可以进一步配置，在检查时使用命令而不是 HTTP 请求，同时为容器安全终止提供充足的时间。虽然这些在更具体的场景中很有用，但了解如何扩展您的部署配置可能是有益的，因此如果您处理的是独特的用例，我建议进行一些额外的阅读。

*进一步阅读：*

[活跃性、就绪性和启动探针](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/)

[配置活跃性、就绪性和启动探针](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

# 使用 Roofline 建模理解应用性能

> 原文：[`towardsdatascience.com/understanding-application-performance-with-roofline-modeling/`](https://towardsdatascience.com/understanding-application-performance-with-roofline-modeling/)

<mdspan datatext="el1750375931063" class="mdspan-comment">计算应用性能时常见的挑战</mdspan>是实际性能和理论性能可能不同。随着高性能需求的产品生态系统（如高性能计算（HPC）、游戏或当前环境中的大型语言模型（LLMs））不断增长，准确计算应用性能变得至关重要。

简单地测量理论上的每秒浮点运算次数（GFLOPs）是不够的，因为应用很少在现实世界中达到这些最大值。这就是 Roofline 模型发挥作用的地方，它提供了一种清晰的可视化方法来估计应用性能，并突出了硬件特定优化的关键作用。

## 为什么简单的指标不够

当我们思考性能测量时，会想到几个指标：

+   **执行时间：** 这告诉你任务花费了多长时间，但无法提供关于*为什么*的见解。

+   **每条指令的周期数（CPI）：** 这仅衡量处理器的计算性能。

+   **串行与并行执行：** 该指标衡量计算性能时忽略了任何硬件优化。

+   **每秒浮点运算次数（FLOP/s）：** 这仅代表一个理论上的最大值，在现实场景中往往难以实现。

虽然这些是好的指标，但它们通常提供的信息不足。例如，使用每秒浮点运算次数是一个理论极限，通常在现实场景中难以实现。因此，仅使用该指标作为*唯一*的指标是不够的，因为它忽略了常见的性能限制因素——数据移动。

## Roofline 建模

Roofline 模型是一个强大的工具，它将应用性能与特定硬件架构（如 CPU 或 GPU）的能力进行可视化映射。该模型的名字来源于它产生的图形的形状，该图形具有一个由斜线和水平线组成的“屋顶”。这种形状代表了硬件强加的最终性能限制。

从这种建模技术中，有两个参数定义了硬件可实现的极限：

+   **数据移动：** 移动数据所需的时间，计算为总数据大小除以系统的峰值内存带宽。

+   **计算：** 计算所需的时间，通过将总浮点运算次数除以系统的峰值计算性能（通常以 GFLOP/s 衡量）来确定。

应用程序的总执行时间由这两个值中的较大者决定：`max {data_movement, computation}`。

尽管硬件具有更好的计算性能，但数据移动往往可能成为瓶颈。Roofline 建模引入了**算术强度（AI）**的概念。AI 是每移动每字节数据所执行的浮点运算的比率。

+   具有高算术强度的算法被认为是计算饥渴型。其性能受限于计算速度。

+   具有低算术强度的算法被认为是数据饥渴型。其性能受限于数据移动速度。

### 理解图表

![Roofline 模型图](img/e003094fea28314733ba1bf8ba308bab.png)

[`commons.wikimedia.org/wiki/File:Example_of_a_naive_Roofline_model.svg`](https://commons.wikimedia.org/wiki/File:Example_of_a_naive_Roofline_model.svg)

[创意共享](https://en.wikipedia.org/wiki/en:Creative_Commons) [署名-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by-sa/4.0/deed.en)

Roofline 图将可达到的 FLOP/s（y 轴）与算术强度（x 轴）进行对比。屋顶本身显示了硬件的限制。屋顶的斜面部分代表峰值数据带宽（以 GB/s 计），而平坦部分代表峰值计算性能（以 GFLOPS 计）。请注意，图像中的所有内容都是对数刻度。

+   **低于屋顶的点**：表示次优性能，表明改进的范围。

+   **击中斜线的点**：数据饥渴型应用程序。其性能受限于数据带宽。

+   **击中平坦线的点**：计算饥渴型应用程序。它正在使用处理器的全部计算能力。

## 为什么 Roofline 建模很重要？

Roofline 建模提供了一种直观的视觉方式来理解应用程序性能，展示了关键特性如操作强度、GPU 能力以及可达到的 FLOP/s。这种建模有助于程序员针对他们使用且能获得更好结果的硬件对应用程序进行有针对性的优化。

+   **瓶颈分析**：拥有视觉辅助工具使得开发者能够轻松地找出瓶颈所在——是内存还是性能。如果应用程序对内存需求较大，开发者可以专注于通过缓存或循环填充等技术来提高数据局部性。如果计算密集，则可以转向启用更多并行计算或利用编译器优化。

+   **硬件和软件设计**：软件工程师不应害怕底层硬件。相反，应该拥抱并优化硬件设计。软件工程师可以使用 Roofline 建模的见解来拥抱并针对他们使用的特定架构进行优化。

## Roofline 建模实践

为了执行 Roofline 模型，我们需要分析应用程序以了解性能。从分析中，我们可以获得如浮点运算（FLOPs）和内存带宽使用等指标，这些对于 Roofline 模型都是必需的。本文探讨了这些工具中的两个——Nvidia 的 `ncu`，它是用于 GPU 分析的 Nsight Compute CLI，以及 PyTorch 的分析器，特别是用于使用 PyTorch 的应用程序。

对于详细的 CUDA 内核优化和精确的 FLOP/byte 计算，`ncu` 提供了直接的 GPU 硬件计数器信息。相比之下，`torch.profiler.profile` 在 PyTorch 中提供了一个更高级别的视角，有助于理解操作级别的性能、张量内存使用以及包含 CPU 和 GPU 活动的整体应用程序行为。

### 使用 ncu 进行分析

`ncu` 是用于分析 CUDA 内核的命令行界面 [2]。它可以直接在终端显示结果或将它们保存到日志文件以供后续分析。为了构建 Roofline 模型，我们需要捕获将允许我们计算算术强度的特定指标。

我们将使用 PyTorch ImageNet 存储库 [3] 作为我们的示例。这是一个不错的选择，因为它易于理解，由 PyTorch 良好地记录，并且与他们的分析器兼容，因此我们可以真正深入性能分析。

#### 第 1 步：运行 ncu 命令以收集指标

第一步是通过 ncu 运行应用程序以收集必要的硬件级数据。命令看起来像这样：

```py
ncu --log-file <log_file_name> \
    --metrics <list_of_metrics_separated_by_comma> \
    --target-processes all \
    python3 <your_application.py application_arguments>
```

+   log-file: 我们想要存储结果的日志文件。

+   指标：这是最重要的参数，描述了我们想要捕获的指标。为了计算算术强度，我们考虑：

    +   `dram__sectors_write.sum` : 写入的 DRAM 扇区求和

    +   `dram__sectors_read.sum` : 读取的 DRAM 扇区求和

    +   `smsp__sass_thread_inst_executed_op_fadd_pred_on.sum` : 浮点加法求和

    +   `smsp__sass_thread_inst_executed_op_fmul_pred_on.sum` : 浮点乘法求和

    +   `smsp__sass_thread_inst_executed_op_ffma_pred_on.sum` : 浮点融合乘加操作的求和

+   target-process: `all` 标志确保我们分析整个应用程序。

我们的 ncu 命令变为：

```py
ncu --log-file logs_example --metrics dram__sectors_write.sum, \
dram__sectors_read.sum, \
smsp__sass_thread_inst_executed_op_fadd_pred_on.sum, \ 
smsp__sass_thread_inst_executed_op_fmul_pred_on.sum, \
smsp__sass_thread_inst_executed_op_ffma_pred_on.sum \
--target-processes all python3 \
main.py /imagenet --arch resnet50 --epochs 1 --batch-size 10 \
--print-freq 10 --seed 42
```

#### 第 2 步：从指标计算 FLOPs

一旦分析器运行完毕，我们可以汇总收集到的指标来计算总的浮点运算。公式是：

\[FLOPs = 2 * FMA\_count + FADD\_count + FMUL\_count\]

+   **FLOPs:** 浮点运算的计数。

+   **FMA_count:** 融合乘加（FMA）操作通常计为 2 个 FLOPs（一个乘法和一次加法）。这由 `smsp__sass_thread_inst_executed_op_ffma_pred_on.sum` 指标表示。

+   **FADD_count:** 这由 `smsp__sass_thread_inst_executed_op_fadd_pred_on.sum` 指标表示。

+   **FMUL_count:** 这由 `smsp__sass_thread_inst_executed_op_fmul_pred_on.sum` 指标表示。

#### 第 3 步：计算传输的字节数

接下来，我们计算传输到和从 DRAM 的总数据量。ncu 指标提供了读取和写入 DRAM 扇区的数量。假设现代 GPU 的扇区大小为 32 字节：

\[Total\_DRAM\_bytes = (dram\_\_sectors\_read.sum + dram\_\_sectors\_write.sum) * 32\]

#### 第 4 步：计算算术强度

使用 FLOPs 和总字节数，我们现在可以计算算术强度：

\[AI = FLOPs / Total\_DRAM\_Bytes\]

#### 第 5 步：计算执行时间

要找到应用程序在 FLOP/s 中的性能，我们还需要执行时间。为此，我们可以使用 NVIDIA Nsight Systems (nsys)，这是一个系统级的分析器，可以精确地测量应用程序段运行时间。我们再次运行我们的应用程序，这次使用 nsys 来生成基于时间的报告。从这份报告中，我们可以提取总的 GPU 运行时间。

```py
nsys profile -f true -o <your_nsys_output_file.qdrep> python3 \
<your_application.py application_arguments>
```

我们的 nsys 命令变为：

```py
nsys profile -f true -o time.qdrep python3 main.py /imagenet \
--arch resnet50 --epochs 1 --batch-size 10 --print-freq 10 \
--seed 42
```

运行此命令后，我们可以获取`GPU_RUNNING_TIME`。

#### 第 6 步：计算应用程序性能

最后，我们通过将总 FLOPs 除以执行时间来计算实现的性能（FLOP/s）：

\[FLOP/s = FLOPs / GPU\_RUNNING\_TIME\]

这个值给出了我们可以在 Roofline 图上绘制的“可达到的 FLOP/s”。

### 使用 torch 进行性能分析

对于用 PyTorch 编写的应用程序，内置的`torch.profiler.profile`提供了一个用户友好的方式来收集性能数据。提供了两个选项供开发者选择：

+   使用分析器上下文管理器

+   针对特定神经网络层的性能分析

#### 分析器上下文管理器

我们想要分析代码的部分可以包裹在`torch.profiler.profile()`上下文管理器中。在`with`语句中，你可以定义要跟踪的活动（CPU、CUDA 或两者），设置一个`schedule`来分析特定的训练步骤，并选择是否记录张量形状、内存使用情况或 FLOPs。一旦进入上下文，必须在每个迭代的末尾调用`prof.step()`来通知分析器前进，尤其是在使用 schedule 的情况下。

```py
with profile(
    activities=<arguments>,
    schedule=torch.profiler.schedule(<arguments>),
    record_shapes=<True|False>,
    profile_memory=<True|False>,
    with_flops=<True|False>
) as prof:

    ....
    prof.step()
```

+   **activities**: 指定是否分析 CPU、CUDA 或两者。

+   **schedule:** 对于在训练循环中分析多个步骤非常有用。如果使用 schedule 参数，分析器需要调用 prof.step()来移动到下一个步骤。

+   **record_shapes:** 是否记录张量的形状。

+   **profile_memory:** 用于捕获内存使用情况

+   **with_flops:** 这是一个实验性功能，用于使用算子计算 FLOPs。

我们的分析器命令变为：

```py
with profile(
    activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
    schedule=torch.profiler.schedule(wait=1, warmup=1, active=3, repeat=2),
    record_shapes=True,
    profile_memory=True,
    with_flops=True
) as prof:
```

#### 针对特定神经网络层的性能分析

分析器也可以更具体地用于分析神经网络的具体层。这有助于检查某些特定层是否比其他层对性能的贡献更大，从而给开发者提供修改特定层的选项。虽然使用起来非常简单，但在大多数情况下，第一个选项效果更好。PyTorch 分析器的结果也可以导出并在 TensorBoard 上可视化。

```py
profiler.start()
self.conv2(x)
profiler.stop()
```

## LLMs 和 Roofline 建模

来到大家一直期待的话题——屋顶线建模是否有助于 LLM 性能计算？简短的答案是肯定的。

LLMs 是具有数十亿参数的复杂神经网络架构，以及它们处理的庞大数据集。虽然训练是一个非常资源密集的任务，但推理和微调模型也需要高效。

+   **瓶颈：** 在推理过程中，LLM 可能会因为处理的大量参数而遭受瓶颈。这些参数是模型的权重，它们导致内存带宽问题。使用屋顶线建模，可以针对瓶颈进行精确的层分析。

+   **硬件选择：** 由于大多数组织更倾向于微调现有模型而不是从头开始训练，因此选择正确的基础设施对于管理成本至关重要。这强调了选择最佳基础设施进行训练的重要性。例如，根据您的 LLM 架构选择硬件或优化模型以在特定架构上运行可以降低训练和推理成本。

## 结论

屋顶线模型提供了对应用程序性能优化的强大可视化分析。通过可视化应用程序在内存和计算方面的性能，提供了明确的指导，以选择最佳的方法来接近优化。虽然这篇文章只考虑了简单的屋顶线模型，但还有更高级的技术，例如分层屋顶线模型或为特定的计算优化添加天花板。

## 参考文献

[1] [`docs.nersc.gov/tools/performance/roofline/`](https://docs.nersc.gov/tools/performance/roofline/)

[2] [`docs.nvidia.com/nsight-compute/NsightComputeCli/index.html`](https://docs.nvidia.com/nsight-compute/NsightComputeCli/index.html)

[3] [`github.com/pytorch/examples/tree/main/imagenet`](https://github.com/pytorch/examples/tree/main/imagenet)

[4] [`developer.nvidia.com/nsight-systems`](https://developer.nvidia.com/nsight-systems)

# SENet 论文解读：通道注意力

> 原文：[`towardsdatascience.com/the-channel-wise-attention/`](https://towardsdatascience.com/the-channel-wise-attention/)

## <mdspan datatext="el1754434926679" class="mdspan-comment">引言</mdspan>

当我们谈论计算机视觉中的注意力时，首先可能想到的是在 Vision Transformer (ViT) 架构中使用的注意力。事实上，我们用于图像数据的注意力机制不止这一种。实际上还有一种叫做 Squeeze and Excitation Network (SENet) 的机制。如果 ViT 中的注意力在空间上操作，即分配权重给图像的不同区域，那么 SENet 中提出的注意力机制在通道上操作，即分配权重给不同的通道。— 在这篇文章中，我们将讨论 Squeeze and Excitation 架构的工作原理，如何从头开始实现它，以及如何将网络集成到 ResNeXt 模型中。

* * *

## Squeeze 和 Excitation 模块

SENet，首次在名为“*Squeeze-and-Excitation Networks*”的论文中提出，由 Hu 等人 [1] 完成，它不是一个像 VGG、Inception 或 ResNet 这样的独立网络。相反，它实际上是一个可以放置在现有网络上的构建块。在基于 CNN 的模型中，我们假设空间上彼此靠近的像素具有高度相关性，这就是我们使用小尺寸核来捕获这些相关性的原因。这种假设基本上是 CNN 的 *归纳偏差*。另一方面，SENet 引入了一种新的归纳偏差，其中作者假设每个图像通道对预测特定类别有不同的贡献。通过将 SE 模块应用于 CNN，模型不仅依赖于空间模式，还捕获了每个通道的重要性。为了更好地说明这一点，我们可以想象一幅火焰的图像，其中红色通道理论上对最终预测的贡献将高于蓝色和绿色通道。

SE 模块的结构如图 1 所示。正如网络名称所暗示的，在这个模块中执行了两个主要步骤：*squeeze* 和 *excitation*。*squeeze* 部分对应于标记为 *F_sq* 的操作，而 *excitation* 部分包括 *F_ex* 和 *F_scale*。另一方面，*F_tr* 操作实际上不是 SE 模块的一部分。相反，它代表了一个原本属于应用 SE 模块模型的转换函数。例如，如果我们把此 SE 模块放在 ResNet 上，*F_tr* 操作指的是瓶颈块内的卷积层堆叠。

![](img/6f3707c370be15b9612d05c980e2a15f.png)

图 1. Squeeze 和 Excitation 模块的结构 [1]。

更具体地来说，关于 *F_sq* 操作，它本质上是通过利用全局平均池化机制来工作的，其中它被用来从每个通道的整个空间维度中捕获信息。通过这样做，输入张量的每个通道都将由一个单一的数字来表示，这基本上就是对应通道的平均值。作者将这个操作称为 *全局信息嵌入*。从数学的角度讲，这可以正式地用图 2 中所示的方程来表示，其中我们基本上将高度 *H* 和宽度 *W* 上的所有值相加，然后再除以该通道内的像素数（*H×W*）。

![图片](img/8bba73d8e9294418e34b497dc3066c45.png)

图 2. SE 模块中全局平均池化机制的数学表达式 [1]。

同时，兴奋和缩放操作都被称为 *自适应重校准*，因为它们本质上是根据其重要性动态调整输入张量中每个通道的权重。实际上，图 1 中的图表并没有完全描绘整个 SENet 架构。你可以看到在图中，*F_ex* 似乎是一个单一的操作，但实际上它由两个线性层和每个线性层后的一个激活函数组成。请参见下面的图 3 以获取详细信息。

![图片](img/97fd36e02969b2300d21ba47484c8679.png)

图 3. ***F_ex*** 操作的数学公式 [1]。

两个线性层分别表示为 *W_1* 和 *W_2*，而 *δ* 和 *σ* 分别代表 ReLU 和 sigmoid 激活函数。因此，根据这个数学定义，在实现过程中我们基本上需要将张量 *z*（平均池化后的张量）通过第一个线性层，然后是 ReLU 激活函数，第二个线性层，最后是 sigmoid 激活函数。记住，sigmoid 函数将输入值归一化到 0 到 1 的范围内。在这种情况下，我们将结果输出视为每个通道的权重，其中接近 1 的值表示相应的通道包含重要信息，因此我们允许模型更多地关注该通道。否则，如果结果数字接近 0，则表示相应的通道对输出的贡献不大。

为了利用这些通道权重，我们可以执行 *F_scale* 操作，这基本上就是原始张量 *u* 和权重张量 *s* 的乘积，如图 4 所示。通过这样做，我们本质上保留了重要通道中的值，同时抑制了不重要通道的值。

![图片](img/fe20e2d621bb99fd8990aab72b9a4fa5.png)

图 4. 缩放过程仅仅是原始张量和权重张量的乘积 [1]。

顺便说一句，很抱歉这里有点过于数学化，哈哈。但我相信这会帮助你在实现部分的代码理解上有所帮助。

### SE 模块放置的位置

在像 VGG 这样的平面 CNN 模型上应用 SE 模块很容易，因为我们只需在每个卷积层之后简单地放置它。然而，由于 Inception 或 ResNet 中存在并行分支，这可能并不直接。为了解决这种混淆，作者提供了在如图 5 所示的两种模型上具体实现 SE 模块的指南。

![图片](img/b9c3bfb0d861ec84df2d92d7589d7f41.png)

图 5. SE 模块在 Inception 和 ResNet 中的放置位置[1]。

对于 Inception 模型，我们不是在每个卷积层之后立即放置 SE 模块，而是将输入张量通过整个 Inception 块（包括所有内部分支）传递，然后附加 SE 模块。同样的方法也适用于 ResNet，但请注意，在 SE 模块处理主张量之后，跳过连接中的张量与主流之间的求和操作发生。

如我之前所述，激励阶段本质上由两个线性层组成。如果我们仔细观察上述结构，我们可以看到第一个线性层的输出形状是 1×1×C/*r*。变量*r*被称为*缩减比*，它在最终通过第二个线性层将权重张量投影回 1×1×C 之前，减少了权重张量的维度。第一层所做的维度缩减充当*瓶颈*操作，这对于限制模型复杂性和提高泛化能力是有用的。作者对不同的*r*值进行了实验，并发现*r* = 16 在准确性和复杂性之间达到了最佳平衡。

![图片](img/77a6a6596efa7e9fdb69cb85f7d695ca.png)

图 6. 在 ResNet 中附加 SE 模块的几种可能方式[1]。

除了在 ResNet 中实现 SE 模块外，如图 6 所示，实际上我们还可以遵循几种方法来实现。根据图 7 中的实验结果，标准 SE、SE-PRE 和 SE-Identity 块获得了相似的结果，同时它们都显著优于 SE-POST。这表明 SE 模块的位置会影响模型性能，尤其是在准确性方面。基于这些发现，作者认为只要我们在元素级求和操作之前应用 SE 模块，我们就能获得良好的结果。在编码部分稍后，我将演示如何实现标准 SE 块。

![图片](img/a64873b8293e9fd408a81069a0343623.png)

图 7. 不同 SE 模块集成策略的实验结果[1]。

### 更多实验结果

论文中实际上讨论了更多实验结果。其中之一是当 SE 模块应用于现有的基于 CNN 的模型时，显示准确度得分提高的表格。我提到的表格显示在下面的图 8 中。

![](img/8d99583bb113d860dc379dfc524d0554.png)

图 8.在不同模型上应用 SE 模块的实验结果[1][2]。

蓝色高亮的列代表每个模型的错误率，粉色列表示以 GFLOPs 为单位的计算复杂度。*重新实现*列指的是作者自己实现的简单模型，而*SENet*列表示配备了 SE 模块的相同模型。表格清楚地显示，当应用 SE 模块时，top-1 和 top-5 错误率都会降低。重要的是要知道，尽管添加 SE 模块会导致 GFLOPs 增加，但与错误率的降低相比，这种增加是相当微小的。

接下来，我们实际上可以通过打印出推理阶段 SE 模块中包含的值来揭示一些有趣的见解。让我们看一下下面的图 9 中的图表，以更好地说明这一点。这些图表的*x*轴表示通道号，*y*轴表示每个通道根据其重要性所拥有的权重，而线条的颜色表示被预测的类别。

![](img/abd2c5f964c605eeced22cf8a53688fe.png)

图 9. SE 模块在不同网络深度下的激活情况[1]。

在较浅的层中，SE 模块捕获的特征是*类别无关的*，这基本上意味着它捕获了预测所有类别所需的通用信息。被称为(a)和(b)的图表，即 ResNet 阶段 2 和 3 的 SE 模块，显示不同类别之间的通道活动没有太大差异，这表明这两个模块没有捕获特定类别的信息。实际上，这与深层层的 SE 模块不同，即阶段 4(c)和阶段 5(d)的 SE 模块。我们可以看到，这两个模块根据被预测的类别调整通道权重的方式不同。这就是深层层的 SE 模块被称为*类别特定*的基本原因。然而，作者承认，在阶段 5 的第二个块(e)中的一些 SE 模块中可能发生异常行为。在这里，SE 模块没有显示出有意义的通道重新校准行为，这表明它不像我们之前讨论的那样有贡献。

### 详细架构

在本文中，我们将实现*SE-ResNeXt-50（32×4d）*模型，如图 10 中右侧列所示。ResNeXt 模型本身与 ResNet 相似，只是每个块中第二卷积层的组参数被设置为 32。如果你熟悉 ResNeXt，这实际上是实现所谓的*基数*最简单且有效的方法。如果你还不熟悉 ResNeXt，我建议你阅读我之前关于 ResNeXt 的文章，文章末尾的参考文献[3]提供了链接。

仔细观察架构，*SE-ResNet-50*与*ResNet-50*的区别仅在于 SE 模块的存在。同样的情况也适用于*SE-ResNeXt-50（32×4d）*与*ResNeXt-50（32×4d）*（在表中未显示）。注意下面图中，带有 SE 模块的模型在每个块的最后卷积层之后附加了一个*fc*层，相应的两个数字表示 SE 模块内部的第一和第二全连接层。

![](img/be2ce08efb4941d6185632f05da70418.png)

图 10. ResNet-50、SE-ResNet-50 和 SE-ResNeXt-50（32×4d）的完整架构[1]。

* * *

## 从零开始实现

记住，这里我们即将在 ResNeXt 中集成 SE 模块，因此我们需要从头开始实现它们。从技术上讲，实际上可以直接从 PyTorch 中获取 ResNeXt 架构，然后手动将其 SE 模块附加到上面。然而，这里我决定使用我之前文章中的 ResNeXt 实现，因为它比我从 PyTorch 中获取的实现更容易理解。请注意，这里我将专注于构建 SE 模块以及如何将其附加到 ResNeXt 模型上，而不是解释 ResNeXt 本身，因为我已经在文章[3]中介绍过了。

现在我们开始编写代码，导入所需的模块。

```py
# Codeblock 1
import torch
import torch.nn as nn
```

### 挤压和激励模块

以下 SE 模块实现遵循图 5（右）所示的图示。值得注意的是，下面的`SEModule`类不包括跳过连接（曲线箭头），因为整个 SE 模块是在初始分支之后但在合并（求和）之前应用的。

这个类的`__init__()`方法接受两个参数：`num_channels`和`r`，如代码块 2a 中的行`#(1)`所示。我们肯定希望这个 SE 模块在整个网络中都是可用的。因此，我们需要将`num_channels`参数设置为可调整的，因为不同阶段的 ResNeXt 块中输出通道的数量是不同的，如图 10 所示。同时，尽管我们在整个网络中的 SE 模块通常使用相同的缩减比*r*，但从技术上讲，我们可以为不同的阶段使用不同的`r`，这可能会是一个有趣的实验对象。所以，这也是我也将`r`参数设置为可调整的原因。

```py
# Codeblock 2a
class SEModule(nn.Module):
    def __init__(self, num_channels, r):                     #(1)
        super().__init__()

        self.global_pooling = nn.AdaptiveAvgPool2d(output_size=(1,1))  #(2)
        self.fc0 = nn.Linear(in_features=num_channels,       #(3)
                             out_features=num_channels//r, 
                             bias=False)
        self.relu = nn.ReLU()                                #(4)
        self.fc1 = nn.Linear(in_features=num_channels//r,    #(5)
                             out_features=num_channels, 
                             bias=False)
        self.sigmoid = nn.Sigmoid()                          #(6)
```

在`__init__()`方法内部，我们需要初始化 5 层。我按照图 5 中给出的顺序写下它们，即全局平均池化层（`#(2)`）、线性层（`#(3)`）、ReLU 激活函数（`#(4)`）、另一个线性层（`#(5)`）和 sigmoid 激活函数（`#(6)`）。在这里，你可以看到第一个线性层负责通过将通道数从`num_channels`缩减到`num_channels//r`来执行降维，然后第二个线性层将其扩展回`num_channels`。请注意，我们将两个线性层的偏置项设置为`False`，这实际上意味着我们只将利用权重张量。两层中偏置项的缺失迫使 SE 模块学习一个通道与其他通道之间的相关性，而不仅仅是添加固定的调整。

仍然使用`SEModule`类，现在让我们继续到`forward()`方法来定义网络的流程。你可以在代码块 2b 中的行`#(1)`看到，我们从单个输入`x`开始，在 ResNeXt 的情况下，它本质上是由同一 ResNeXt 块内的第三个卷积层产生的张量。如图 5 所示，接下来我们需要分支出网络。在这里，我们直接使用`global_pooling`层处理分支，我将得到的张量命名为`squeezed`（`#(2)`）。原始输入张量`x`本身将保持不变，因为我们不会在缩放阶段对其进行任何操作。接下来，我们需要使用`torch.flatten()`（`#(3)`）去除`squeezed`张量的空间维度。这基本上是因为我们想要进一步使用行`#(4)`和`#(5)`中的线性层进行处理，这些层只能处理一维张量。空间维度然后在行`#(6)`再次引入，允许我们在行`#(7)`处执行`x`（原始张量）和`excited`（通道权重）之间的乘法。这个过程产生了一个重新校准的`x`版本，我们称之为`scaled`。在这里，我在每个步骤后打印出张量维度，以便你更好地理解这个 SE 模块的流程。

```py
# Codeblock 2b
    def forward(self, x):                                  #(1)
        print(f'original\t\t: {x.size()}')

        squeezed = self.global_pooling(x)                  #(2)
        print(f'after avgpool\t\t: {squeezed.size()}')

        squeezed = torch.flatten(squeezed, 1)              #(3)
        print(f'after flatten\t\t: {squeezed.size()}')

        excited = self.relu(self.fc0(squeezed))            #(4)
        print(f'after fc0-relu\t\t: {excited.size()}')

        excited = self.sigmoid(self.fc1(excited))          #(5)
        print(f'after fc1-sigmoid\t: {excited.size()}')

        excited = excited[:, :, None, None]                #(6)
        print(f'after reshape\t\t: {excited.size()}')

        scaled = x * excited                               #(7)
        print(f'after scaling\t\t: {scaled.size()}')

        return scaled
```

现在我们将通过传递一个虚拟张量通过它来检查我们是否正确实现了网络。在下面的代码块 3 中，我初始化了一个 SE 模块，并配置它接受一个 512 通道的图像张量，并且具有 16 的缩减比例（`#(1)`）。如果你看一下图 10 中的 SE-ResNeXt 架构，这个 SE 模块基本上对应于第三阶段中的那个（其输出大小为 28×28）。因此，在行`#(2)`我们需要相应地调整虚拟张量的形状。然后我们使用行`#(3)`中的代码将这个张量输入到网络中。

```py
# Codeblock 3
semodule = SEModule(num_channels=512, r=16)    #(1)
x = torch.randn(1, 512, 28, 28)                #(2)

out = semodule(x)      #(3)
```

下面是打印函数给出的结果。

```py
# Codeblock 3 Output
original          : torch.Size([1, 512, 28, 28])    #(1)
after avgpool     : torch.Size([1, 512, 1, 1])      #(2)
after flatten     : torch.Size([1, 512])            #(3)
after fc0-relu    : torch.Size([1, 32])             #(4)
after fc1-sigmoid : torch.Size([1, 512])            #(5)
after reshape     : torch.Size([1, 512, 1, 1])      #(6)
after scaling     : torch.Size([1, 512, 28, 28])    #(7)
```

你可以看到原始张量的形状与我们的虚拟张量完全匹配，即 1×512×28×28（`#(1)`）。顺便说一下，我们可以忽略 0 轴上的数字 1，因为它本质上表示批处理大小，在这种情况下，我假设我们只在一个批处理中得到了一张图像。经过池化后，空间维度折叠到 1×1，因为现在每个通道都由一个单独的数字表示（`#(2)`）。我之前解释的扁平化操作的目的就是删除两个空轴（`#(3)`），因为后续的线性层只能处理单维张量。在这里，你可以看到第一个线性层通过我们之前设置的缩减比例 16 将张量维度减少到 32（`#(4)`）。然后，第二个线性层将这个张量的长度扩展回 512（`#(5)`）。接下来，我们解压缩张量，以便我们恢复 1×1 的空间维度（`#(6)`），这样我们就可以将其与输入张量相乘（`#(7)`）。基于这个详细的流程，你可以看到 SE 模块基本上保留了原始张量的维度，这证明了该模块可以附加到任何基于 CNN 的模型上，而不会破坏网络的原始流程。

#### ResNeXt

既然我们已经理解了如何从头开始实现 SE 模块，现在我将向你展示我们如何将其附加到 ResNeXt 模型上。在这样做之前，我们需要初始化实现 ResNeXt 架构所需的参数。在下面的代码块 4 中，前四个变量是根据*ResNeXt-50 (32×4d)*变体确定的，而最后一个变量（`R`）代表 SE 模块的缩减比例。

```py
# Codeblock 4
CARDINALITY  = 32
NUM_CHANNELS = [3, 64, 256, 512, 1024, 2048]
NUM_BLOCKS   = [3, 4, 6, 3]
NUM_CLASSES  = 1000
R = 16
```

在代码块 5a 和 5b 中定义的`Block`类是我之前文章中的 ResNeXt 块。实际上，我们在`__init__()`方法中做了很多事情，但总体思路是在初始化 SE 模块之前，初始化三个称为`conv0`（`#(1)`）、`conv1`（`#(2)`）和`conv2`（`#(3)`）的卷积层。稍后，我们将根据图 10 中显示的 SE-ResNeXt 架构配置这些层。

```py
# Codeblock 5a
class Block(nn.Module):
    def __init__(self, 
                 in_channels,
                 add_channel=False,
                 channel_multiplier=2,
                 downsample=False):
        super().__init__()

        self.add_channel = add_channel
        self.channel_multiplier = channel_multiplier
        self.downsample = downsample

        if self.add_channel:
            out_channels = in_channels*self.channel_multiplier
        else:
            out_channels = in_channels

        mid_channels = out_channels//2

        if self.downsample:
            stride = 2
        else:
            stride = 1

        if self.add_channel or self.downsample:
            self.projection = nn.Conv2d(in_channels=in_channels,
                                        out_channels=out_channels, 
                                        kernel_size=1, 
                                        stride=stride, 
                                        padding=0, 
                                        bias=False)
            nn.init.kaiming_normal_(self.projection.weight, nonlinearity='relu')
            self.bn_proj = nn.BatchNorm2d(num_features=out_channels)

        self.conv0 = nn.Conv2d(in_channels=in_channels,       #(1)
                               out_channels=mid_channels,
                               kernel_size=1, 
                               stride=1, 
                               padding=0, 
                               bias=False)
        nn.init.kaiming_normal_(self.conv0.weight, nonlinearity='relu')
        self.bn0 = nn.BatchNorm2d(num_features=mid_channels)

        self.conv1 = nn.Conv2d(in_channels=mid_channels,      #(2)
                               out_channels=mid_channels, 
                               kernel_size=3, 
                               stride=stride,
                               padding=1, 
                               bias=False, 
                               groups=CARDINALITY)
        nn.init.kaiming_normal_(self.conv1.weight, nonlinearity='relu')
        self.bn1 = nn.BatchNorm2d(num_features=mid_channels)

        self.conv2 = nn.Conv2d(in_channels=mid_channels,      #(3)
                               out_channels=out_channels,
                               kernel_size=1, 
                               stride=1, 
                               padding=0, 
                               bias=False)
        nn.init.kaiming_normal_(self.conv2.weight, nonlinearity='relu')
        self.bn2 = nn.BatchNorm2d(num_features=out_channels)

        self.relu = nn.ReLU()

        self.semodule = SEModule(num_channels=out_channels, r=R)    #(4)
```

`forward()` 方法本身通常也与原始的 ResNeXt 模型相同，只是在这里我们需要将 SE 模块放在元素级求和之前，如下面的代码块 5b 中的行 `#(1)` 所示。请记住，这种实现遵循图 6 (b) 中的标准 SE 模块架构。

```py
# Codeblock 5b
    def forward(self, x):
        print(f'original\t\t: {x.size()}')

        if self.add_channel or self.downsample:
            residual = self.bn_proj(self.projection(x))
            print(f'after projection\t: {residual.size()}')
        else:
            residual = x
            print(f'no projection\t\t: {residual.size()}')

        x = self.conv0(x)
        x = self.bn0(x)
        x = self.relu(x)
        print(f'after conv0-bn0-relu\t: {x.size()}')

        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        print(f'after conv1-bn1-relu\t: {x.size()}')

        x = self.conv2(x)
        x = self.bn2(x)
        print(f'after conv2-bn2\t\t: {x.size()}')

        x = self.semodule(x)      #(1)
        print(f'after semodule\t\t: {x.size()}')

        x = x + residual
        x = self.relu(x)
        print(f'after summation\t\t: {x.size()}')

        return x
```

在上述实现中，每次我们实例化一个 `Block` 对象时，我们都会有一个已经配备了 SE 模块的 ResNeXt 模块。现在我们将测试上述类，看看我们是否正确实现了它。在这里，我将在第三阶段模拟一个 ResNeXt 模块。`add_channel` 和 `downsample` 参数设置为 `False`，因为我们想保留输入张量的通道数和空间维度。

```py
# Codeblock 6
block = Block(in_channels=512, add_channel=False, downsample=False)
x = torch.randn(1, 512, 28, 28)

out = block(x)
```

下面是输出看起来像什么。您可以看到，我们的第一个卷积层成功地将通道数从 512 减少到 256 (`#(1)`)，然后由第三个卷积层将其扩展回原始维度 (`#(2)`)。之后，张量通过 SE 模块，输出大小与输入相同，就像我们在代码块 3 中看到的那样 (`#(3)`)。SE 模块的处理完成后，我们最终可以在主分支的张量和跳过连接的张量之间执行元素级求和 (`#(4)`)。

```py
original             : torch.Size([1, 512, 28, 28])
no projection        : torch.Size([1, 512, 28, 28])
after conv0-bn0-relu : torch.Size([1, 256, 28, 28])    #(1)
after conv1-bn1-relu : torch.Size([1, 256, 28, 28])
after conv2-bn2      : torch.Size([1, 512, 28, 28])    #(2)
after semodule       : torch.Size([1, 512, 28, 28])    #(3)
after summation      : torch.Size([1, 512, 28, 28])    #(4)
```

下面是如何实现整个架构。我们本质上需要做的就是根据图 10 中的架构堆叠多个 SE-ResNeXt 模块。实际上，代码块 7 中的 `SEResNeXt` 类与我之前文章 [3] 中的 `ResNeXt` 类完全相同（我实际上复制粘贴了它），因为使 SE-ResNeXt 与原始 ResNeXt 区别开来的只是我们之前讨论的 `Block` 类中存在 SE 模块。

```py
# Codeblock 7
class SEResNeXt(nn.Module):
    def __init__(self):
        super().__init__()

        # conv1 stage
        self.resnext_conv1 = nn.Conv2d(in_channels=NUM_CHANNELS[0],
                                       out_channels=NUM_CHANNELS[1],
                                       kernel_size=7,
                                       stride=2,
                                       padding=3, 
                                       bias=False)
        nn.init.kaiming_normal_(self.resnext_conv1.weight, 
                                nonlinearity='relu')
        self.resnext_bn1 = nn.BatchNorm2d(num_features=NUM_CHANNELS[1])
        self.relu = nn.ReLU()
        self.resnext_maxpool1 = nn.MaxPool2d(kernel_size=3,
                                             stride=2, 
                                             padding=1)

        # conv2 stage
        self.resnext_conv2 = nn.ModuleList([
            Block(in_channels=NUM_CHANNELS[1],
                  add_channel=True,
                  channel_multiplier=4,
                  downsample=False)
        ])
        for _ in range(NUM_BLOCKS[0]-1):
            self.resnext_conv2.append(Block(in_channels=NUM_CHANNELS[2]))

        # conv3 stage
        self.resnext_conv3 = nn.ModuleList([Block(in_channels=NUM_CHANNELS[2],
                                                  add_channel=True, 
                                                  downsample=True)])
        for _ in range(NUM_BLOCKS[1]-1):
            self.resnext_conv3.append(Block(in_channels=NUM_CHANNELS[3]))

        # conv4 stage
        self.resnext_conv4 = nn.ModuleList([Block(in_channels=NUM_CHANNELS[3],
                                                  add_channel=True, 
                                                  downsample=True)])

        for _ in range(NUM_BLOCKS[2]-1):
            self.resnext_conv4.append(Block(in_channels=NUM_CHANNELS[4]))

        # conv5 stage
        self.resnext_conv5 = nn.ModuleList([Block(in_channels=NUM_CHANNELS[4],
                                                  add_channel=True, 
                                                  downsample=True)])

        for _ in range(NUM_BLOCKS[3]-1):
            self.resnext_conv5.append(Block(in_channels=NUM_CHANNELS[5]))

        self.avgpool = nn.AdaptiveAvgPool2d(output_size=(1,1))

        self.fc = nn.Linear(in_features=NUM_CHANNELS[5],
                            out_features=NUM_CLASSES)

    def forward(self, x):
        print(f'original\t\t: {x.size()}')

        x = self.relu(self.resnext_bn1(self.resnext_conv1(x)))
        print(f'after resnext_conv1\t: {x.size()}')

        x = self.resnext_maxpool1(x)
        print(f'after resnext_maxpool1\t: {x.size()}')

        for i, block in enumerate(self.resnext_conv2):
            x = block(x)
            print(f'after resnext_conv2 #{i}\t: {x.size()}')

        for i, block in enumerate(self.resnext_conv3):
            x = block(x)
            print(f'after resnext_conv3 #{i}\t: {x.size()}')

        for i, block in enumerate(self.resnext_conv4):
            x = block(x)
            print(f'after resnext_conv4 #{i}\t: {x.size()}')

        for i, block in enumerate(self.resnext_conv5):
            x = block(x)
            print(f'after resnext_conv5 #{i}\t: {x.size()}')

        x = self.avgpool(x)
        print(f'after avgpool\t\t: {x.size()}')

        x = torch.flatten(x, start_dim=1)
        print(f'after flatten\t\t: {x.size()}')

        x = self.fc(x)
        print(f'after fc\t\t: {x.size()}')

        return x
```

当整个 *SE-ResNeXt-50 (32×4d)* 架构完成后，现在我们将通过传递一个大小为 1×3×224×224 的张量通过网络来测试它，模拟一个 224×224 大小的单个 RGB 图像。您可以在下面的代码块 8 的输出中看到，模型似乎工作正常，因为张量成功通过了 `seresnext` 模型中的所有层而没有返回任何错误。因此，我相信这个模型现在可以开始训练了。顺便说一句，如果您真的想训练这个模型，别忘了根据您数据集中的类别数量更改输出通道中的神经元数量。

```py
# Codeblock 8
seresnext = SEResNeXt()
x = torch.randn(1, 3, 224, 224)

out = seresnext(x)
```

```py
# Codeblock 8 Output
original               : torch.Size([1, 3, 224, 224])
after resnext_conv1    : torch.Size([1, 64, 112, 112])
after resnext_maxpool1 : torch.Size([1, 64, 56, 56])
after resnext_conv2 #0 : torch.Size([1, 256, 56, 56])
after resnext_conv2 #1 : torch.Size([1, 256, 56, 56])
after resnext_conv2 #2 : torch.Size([1, 256, 56, 56])
after resnext_conv3 #0 : torch.Size([1, 512, 28, 28])
after resnext_conv3 #1 : torch.Size([1, 512, 28, 28])
after resnext_conv3 #2 : torch.Size([1, 512, 28, 28])
after resnext_conv3 #3 : torch.Size([1, 512, 28, 28])
after resnext_conv4 #0 : torch.Size([1, 1024, 14, 14])
after resnext_conv4 #1 : torch.Size([1, 1024, 14, 14])
after resnext_conv4 #2 : torch.Size([1, 1024, 14, 14])
after resnext_conv4 #3 : torch.Size([1, 1024, 14, 14])
after resnext_conv4 #4 : torch.Size([1, 1024, 14, 14])
after resnext_conv4 #5 : torch.Size([1, 1024, 14, 14])
after resnext_conv5 #0 : torch.Size([1, 2048, 7, 7])
after resnext_conv5 #1 : torch.Size([1, 2048, 7, 7])
after resnext_conv5 #2 : torch.Size([1, 2048, 7, 7])
after avgpool          : torch.Size([1, 2048, 1, 1])
after flatten          : torch.Size([1, 2048])
after fc               : torch.Size([1, 1000])
```

此外，我们还可以使用以下代码打印出这个模型所拥有的参数数量。在这里，你可以看到代码块返回了 27,543,848。这个参数数量略高于原始的 ResNeXt 模型对应版本，正如我在之前的文章以及官方 PyTorch 文档[4]中提到的，原始 ResNeXt 模型只有 25,028,904 个参数。这种模型大小的增加确实是有道理的，因为整个网络中的 ResNeXt 块现在由于 SE 模块的存在而拥有更多的层。

```py
# Codeblock 9
def count_parameters(model):
    return sum([params.numel() for params in model.parameters()])

count_parameters(seresnext)
```

```py
# Codeblock 9 Output
27543848
```

* * *

## 结束

这就是关于 Squeeze and Excitation 模块的全部内容。我确实鼓励你从这里开始，通过在自己的数据集上训练这个模型来探索，这样你将看到论文中提出的发现是否也适用于你的情况。不仅如此，我认为如果你尝试自己实现 SE 模块在其他神经网络架构如 VGG 或 Inception 上，也会很有趣。

我希望你在今天学到了一些新东西。感谢阅读！

*顺便说一下，你还可以在我的 GitHub 仓库[5]中找到这篇文章中使用的代码。*

* * *

[1] Jie Hu *et al.* Squeeze and Excitation Networks. Arxiv. [`arxiv.org/abs/1709.01507`](https://arxiv.org/abs/1709.01507) [访问日期：2025 年 3 月 17 日].

[2] 图片由作者原创。

[3] 将 ResNet 提升到下一个层次。Towards Data Science. [`towardsdatascience.com/taking-resnet-to-the-next-level/`](https://towardsdatascience.com/taking-resnet-to-the-next-level/) [访问日期：2025 年 7 月 22 日].

[4] Resnext50_32x4d. PyTorch. [`pytorch.org/vision/main/models/generated/torchvision.models.resnext50_32x4d.html#torchvision.models.resnext50_32x4d`](https://pytorch.org/vision/main/models/generated/torchvision.models.resnext50_32x4d.html#torchvision.models.resnext50_32x4d) [访问日期：2025 年 3 月 17 日].

[5] MuhammadArdiPutra. The Channel-Wise Attention — Squeeze and Excitation. GitHub. [`github.com/MuhammadArdiPutra/medium_articles/blob/main/The%20Channel-Wise%20Attention%20-%20Squeeze%20and%20Excitation.ipynb`](https://github.com/MuhammadArdiPutra/medium_articles/blob/main/The%20Channel-Wise%20Attention%20-%20Squeeze%20and%20Excitation.ipynb) [访问日期：2025 年 4 月 7 日].

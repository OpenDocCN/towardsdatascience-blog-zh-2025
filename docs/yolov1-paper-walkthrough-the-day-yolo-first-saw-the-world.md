# YOLOv1 论文解读：YOLO 首次亮相的那一天

> 原文：[`towardsdatascience.com/yolov1-paper-walkthrough-the-day-yolo-first-saw-the-world/`](https://towardsdatascience.com/yolov1-paper-walkthrough-the-day-yolo-first-saw-the-world/)

## <mdspan datatext="el1764722956521" class="mdspan-comment">引言</mdspan>

如果我们谈论目标检测，第一个可能出现在我们脑海中的模型很可能是 YOLO——至少对我来说，多亏了它在计算机视觉领域的普及。

这个模型的第一个版本，被称为 YOLOv1，于 2015 年在名为“*You Only Look Once: Unified, Real-Time Object Detection”* [1]的研究论文中发布。在 YOLOv1 发明之前，用于执行目标检测的最先进算法之一是 R-CNN（基于区域的卷积神经网络），其中它使用多阶段机制来完成这项任务。它最初使用选择性搜索算法来创建区域提议，然后使用基于 CNN 的模型从所有这些区域中提取特征，并最终使用 SVM [2]对检测到的对象进行分类。在这里，你可以清楚地想象出仅对单张图像进行目标检测的过程需要多长时间。

YOLO 最初的动力是提高速度。实际上，不仅实现了低计算复杂度，作者还证明他们提出的深度学习模型也能达到高精度。截至本文撰写时，YOLOv13 已经发布几天了[3]。但让我们现在就先谈谈它的第一个祖先，这样你可以从它首次出现时开始看到这个模型的美。本文将讨论 YOLOv1 是如何工作的，以及如何使用 PyTorch 从头开始构建这个神经网络架构。

* * *

## YOLOv1 背后的基本理论

在我们深入了解架构之前，最好先了解 YOLOv1 背后的想法。让我们从一个例子开始。假设我们有一张猫的图片，我们打算将其用作 YOLOv1 模型的训练样本。因此，我们需要为它创建一个真实情况。原始论文中提到，我们需要定义参数 *S*，它表示我们将沿每个空间维度将图像分成多少个网格单元。默认情况下，此参数设置为 7，因此我们将有 7×7=49 个单元。请查看下面的图 1 以更好地理解这个想法。

![](img/ab6d65bc897d0a9d891e9ebce8d7abbd.png)

图 1. 图像被分成均匀大小的网格单元后的样子。单元（3, 3）负责存储关于猫的信息[4]。

接下来，我们需要确定哪个单元格对应于物体的中心点。在上面的例子中，猫几乎位于图像的中心，因此中心点必须位于单元格(3, 3)。在推理阶段稍后，我们可以将这个单元格视为负责预测猫的单元格。现在仔细观察这个单元格，我们需要确定中点的确切位置。在这里，你可以看到沿着垂直轴它正好位于中间，但在水平轴上它稍微向左偏离了中间。所以，如果我要近似，坐标将是(0.4, 0.5)。这个坐标值是相对于单元格的，并且归一化到 0 到 1 的范围。可能值得注意，中点的(*x*, *y*)坐标既不应小于 0 也不应大于 1，因为超出这个范围的值意味着中点位于另一个单元格中。同时，边界框的宽度*w*和高度*h*大约是 2.4 和 3.2，这些数字是相对于单元格大小的，这意味着如果物体比单元格大，那么这个值将大于 1。稍后，如果我们创建一个图像的 ground truth，我们需要将这些*x*，*y*，*w*和*h*信息存储在所谓的*目标向量*中。

### 目标向量

对于每个单元格，目标向量的长度是 25，其中前 20 个元素（索引 0 到 19）以 one-hot 编码的形式存储物体的类别。这本质上是因为 YOLOv1 最初是在 PASCAL VOC 数据集上训练的，该数据集有那么多类别。接下来，索引 20 用于存储边界框预测的置信度，在训练阶段，只要单元格内有物体中心点，这个置信度就设置为 1。最后，中点的(*x*, *y*)坐标放在索引 21 和 22，而*w*和*h*存储在索引 23 和 24。下面图 2 的插图显示了单元格(3, 3)的目标向量看起来是什么样子。

![图片](img/84cc2b6282bbf1aa3f854f2473c872a0.png)

图 2. 图 1 中图像的单元格(3, 3)的目标向量 [5]。

再次提醒，上述目标向量仅对应单个单元格。为了创建整个图像的 ground truth，我们需要将多个类似的向量连接起来，形成所谓的*目标张量*，如图 3 所示。请注意，来自所有其他单元格的类别概率以及边界框的置信度、位置和大小都设置为 0，因为没有其他物体出现在图像中。

![图片](img/38d0466674642c3967b4c5be543b8c8f.png)

图 3. 每个单元格中所有目标向量的连接方式。这个整个张量将作为单个图像的 ground truth [5]。

### 预测向量

*预测向量*相当不同。如果目标向量由 25 个元素组成，预测向量则由 30 个元素组成。这是因为 YOLOv1 在推理过程中默认为同一对象预测两个边界框。因此，我们需要额外的 5 个元素来存储模型生成的第二个边界框的信息。尽管预测了两个边界框，但后来我们只会选择置信度更高的那个。

![](img/19c4cf0336b91ba0083474cd39db9cdd.png)

图 4. 预测向量有 5 个额外的元素专门用于第二个边界框（橙色突出显示）[5]。

这种独特的目标和预测向量维度要求作者重新思考损失函数。对于回归问题，我们通常使用 MAE、MSE 或 RMSE，而对于分类任务，我们通常使用交叉熵损失。但 YOLOv1 不仅仅是回归和分类问题，考虑到我们在向量表示中既有连续值（边界框）又有离散值（类别）。正因为如此，作者创建了一个针对此模型的新损失函数，如图 5 所示。这个损失函数相当复杂（你看，对吧？），所以我决定写一篇单独的文章来解释它——敬请期待，我很快就会发布。

![](img/8edfa7d1228c33f4847535d4e0cb93ec.png)

图 5. YOLOv1 的损失函数[1]。

* * *

## YOLOv1 架构

就像典型的早期计算机视觉模型一样，YOLOv1 使用基于 CNN 的架构作为模型的骨干。它由 24 个卷积层组成，按照图 6 中的结构堆叠。如果你仔细观察这张图，你会注意到输出层产生一个形状为 30×7×7 的张量。这个维度表明每个单元格都有其对应的长度为 30 的预测向量，其中包含检测到的对象的类别和边界框信息，这与我们之前的讨论完全一致。

![](img/7b6f8a46e0a180ea3041caff85fbf742.png)

图 6. YOLOv1 的架构[1]。

好吧，我想我已经涵盖了 YOLOv1 的所有基础知识，现在让我们从头开始使用 PyTorch 实现架构。在开始之前，我们首先需要导入所需的模块并初始化参数*S*、*B*和*C*。请参见下面的代码块 1。

```py
# Codeblock 1
import torch
import torch.nn as nn

S = 7
B = 2
C = 20
```

我上面初始化的三个参数是论文中给出的默认值，其中`S`代表水平和垂直轴上的网格单元格数，`B`表示每个单元格生成的边界框数，而`C`是数据集中可用的类别数。由于我们使用`S=7`和`B=2`，我们的 YOLOv1 将为每张图像总共生成 7×7×2=98 个边界框。

### 建筑块

接下来，我们将创建`ConvBlock`类，其中包含一个卷积层（行`#(1)`）、一个 leaky ReLU 激活函数（`#(2)`）以及一个可选的最大池化层（`#(3)`），如代码块 2 所示。

```py
# Codeblock 2
class ConvBlock(nn.Module):
    def __init__(self, 
                 in_channels, 
                 out_channels, 
                 kernel_size, 
                 stride, 
                 padding, 
                 maxpool_flag=False):
        super().__init__()
        self.maxpool_flag = maxpool_flag

        self.conv = nn.Conv2d(in_channels=in_channels,       #(1)
                              out_channels=out_channels, 
                              kernel_size=kernel_size, 
                              stride=stride, 
                              padding=padding)
        self.leaky_relu = nn.LeakyReLU(negative_slope=0.1)   #(2)

        if self.maxpool_flag:
            self.maxpool = nn.MaxPool2d(kernel_size=2,       #(3)
                                        stride=2)

    def forward(self, x):
        print(f'original\t: {x.size()}')

        x = self.conv(x)
        print(f'after conv\t: {x.size()}')

        x = self.leaky_relu(x)
        print(f'after leaky relu: {x.size()}')

        if self.maxpool_flag:
            x = self.maxpool(x)
            print(f'after maxpool\t: {x.size()}')

        return x
```

在现代架构中，我们通常使用*Conv-BN-ReLU*结构，但在 YOLOv1 创建的时候，批归一化层似乎还没有那么流行，因为它是在 YOLOv1 几个月前才出现的。所以，我想这可能就是作者没有使用这个归一化层的原因。相反，它只在整个网络中使用一系列卷积和 leaky ReLUs。

快速回顾一下，leaky ReLU 是一种与标准 ReLU 类似的激活函数，除了负值被乘以一个小数而不是被置零。在 YOLOv1 的情况下，我们将乘数设置为 0.1（`#(2)`），以便它仍然可以保留一些负输入数字中包含的信息。

![图片](img/8c131e3981a660c22224ce943e14c287.png)

图 7. ReLU 与 Leaky ReLU 激活函数[6]。

由于`ConvBlock`类已经定义，现在我将对其进行测试，以检查其是否正常工作。在下面的代码块 3 中，我尝试实现网络中的第一层，并通过一个虚拟张量传递。您可以在代码块中看到，`in_channels`被设置为 3（`#(1)`），`out_channels`被设置为 64（`#(2)`），因为我们希望这个初始层接受 RGB 图像作为输入，并返回一个 64 通道的图像。内核的大小是 7×7（`#(3)`），因此我们需要将填充设置为 3（`#(5)`）。通常，这种配置可以让我们保留图像的空间维度，但由于我们使用`stride=2`（`#(4)`），这个填充大小确保图像正好减半。接下来，如果您回到图 6，您会注意到一些卷积层后面跟着一个最大池化层，而另一些则没有。由于第一个卷积使用了最大池化层，我们需要将`maxpool_flag`参数设置为`True`（`#(6)`）。

```py
# Codeblock 3
convblock = ConvBlock(in_channels=3,       #(1)
                      out_channels=64,     #(2)
                      kernel_size=7,       #(3)
                      stride=2,            #(4)
                      padding=3,           #(5)
                      maxpool_flag=True)   #(6)
x = torch.randn(1, 3, 448, 448)            #(7)
out = convblock(x)
```

之后，我们可以简单地生成一个维度为 1×3×448×448 的随机值张量（`#(7)`），这模拟了一个大小为 448×448 的单个 RGB 图像的批次，然后通过网络传递。您可以在下面的输出结果中看到，我们的卷积层成功地将通道数增加到 64，并将空间维度减半到 224×224。减半是通过最大池化层再次完成的，一直减半到 112×112。

```py
# Codeblock 3 Output
original         : torch.Size([1, 3, 448, 448])
after conv       : torch.Size([1, 64, 224, 224])
after leaky relu : torch.Size([1, 64, 224, 224])
after maxpool    : torch.Size([1, 64, 112, 112])
```

### 主干

接下来，我们将创建一系列`ConvBlocks`来构建网络的整个主干。如果您还不熟悉术语*主干*，在这种情况下，它基本上是两个全连接层之前的所有内容（参见图 6）。现在请看下面的代码块 4a 和 4b，以了解我是如何定义`Backbone`类的。

```py
# Codeblock 4a
class Backbone(nn.Module):
    def __init__(self):
        super().__init__()
        # in_channels, out_channels, kernel_size, stride, padding, maxpool_flag
        self.stage0 = ConvBlock(3, 64, 7, 2, 3, maxpool_flag=True)      #(1)
        self.stage1 = ConvBlock(64, 192, 3, 1, 1, maxpool_flag=True)    #(2)

        self.stage2 = nn.ModuleList([
            ConvBlock(192, 128, 1, 1, 0), 
            ConvBlock(128, 256, 3, 1, 1), 
            ConvBlock(256, 256, 1, 1, 0),
            ConvBlock(256, 512, 3, 1, 1, maxpool_flag=True)      #(3)
        ])

        self.stage3 = nn.ModuleList([])
        for _ in range(4):
            self.stage3.append(ConvBlock(512, 256, 1, 1, 0))
            self.stage3.append(ConvBlock(256, 512, 3, 1, 1))

        self.stage3.append(ConvBlock(512, 512, 1, 1, 0))
        self.stage3.append(ConvBlock(512, 1024, 3, 1, 1, maxpool_flag=True))  #(4)

        self.stage4 = nn.ModuleList([])
        for _ in range(2):
            self.stage4.append(ConvBlock(1024, 512, 1, 1, 0))
            self.stage4.append(ConvBlock(512, 1024, 3, 1, 1))

        self.stage4.append(ConvBlock(1024, 1024, 3, 1, 1))
        self.stage4.append(ConvBlock(1024, 1024, 3, 2, 1))    #(5)

        self.stage5 = nn.ModuleList([])
        self.stage5.append(ConvBlock(1024, 1024, 3, 1, 1))
        self.stage5.append(ConvBlock(1024, 1024, 3, 1, 1))
```

在上面的代码块中，我们根据论文中给出的架构实例化`ConvBlock`实例。这里有几个要点我想强调。首先，我在代码中使用的术语*阶段*在论文中没有明确提及。然而，我决定使用这个词来描述图 6 中的六个卷积层组。其次，请注意，我们需要将第一个四组中的最后一个`ConvBlock`的`maxpool_flag`设置为`True`以执行空间下采样（`#(1–4)`）。对于第五组，下采样是通过将最后一个卷积层的步长设置为 2（`#(5)`）来完成的。第三，图 6 没有提到卷积层的填充大小，因此我们需要手动计算它们。确实有一个特定的公式可以根据给定的核大小找到填充大小。然而，我觉得记住它更容易。只需记住，如果我们使用 7×7 大小的核，那么我们需要将填充设置为 3 以保持空间维度。同时，对于 5×5、3×3 和 1×1 核，填充应分别设置为 2、1 和 0。

由于骨干网络中的所有层都已经实例化，我们现在可以使用下面的`forward()`方法将它们全部连接起来。我认为这里不需要解释什么，因为基本上它只是通过按顺序通过输入张量`x`通过层来实现。

```py
# Codeblock 4b
    def forward(self, x):
        print(f'original\t: {x.size()}\n')

        x = self.stage0(x)
        print(f'after stage0\t: {x.size()}\n')

        x = self.stage1(x)
        print(f'after stage1\t: {x.size()}\n')

        for i in range(len(self.stage2)):
            x = self.stage2i
            print(f'after stage2 #{i}\t: {x.size()}')

        print()
        for i in range(len(self.stage3)):
            x = self.stage3i
            print(f'after stage3 #{i}\t: {x.size()}')

        print()
        for i in range(len(self.stage4)):
            x = self.stage4i
            print(f'after stage4 #{i}\t: {x.size()}')

        print()
        for i in range(len(self.stage5)):
            x = self.stage5i
            print(f'after stage5 #{i}\t: {x.size()}')

        return x
```

现在我们通过运行以下测试代码来验证我们的实现是否正确。

```py
# Codeblock 5
backbone = Backbone()
x = torch.randn(1, 3, 448, 448)
out = backbone(x)
```

如果你尝试运行上面的代码块，屏幕上应该出现以下输出。在这里，你可以看到图像的空间维度在经过每个阶段的最后一个`ConvBlock`后正确地减少了。这个过程一直持续到最后一个阶段，最终我们得到了一个大小为 1024×7×7 的张量，这与图 6 中的插图完全匹配。

```py
# Codeblock 5 Output
original        : torch.Size([1, 3, 448, 448])

after stage0    : torch.Size([1, 64, 112, 112])

after stage1    : torch.Size([1, 192, 56, 56])

after stage2 #0 : torch.Size([1, 128, 56, 56])
after stage2 #1 : torch.Size([1, 256, 56, 56])
after stage2 #2 : torch.Size([1, 256, 56, 56])
after stage2 #3 : torch.Size([1, 512, 28, 28])

after stage3 #0 : torch.Size([1, 256, 28, 28])
after stage3 #1 : torch.Size([1, 512, 28, 28])
after stage3 #2 : torch.Size([1, 256, 28, 28])
after stage3 #3 : torch.Size([1, 512, 28, 28])
after stage3 #4 : torch.Size([1, 256, 28, 28])
after stage3 #5 : torch.Size([1, 512, 28, 28])
after stage3 #6 : torch.Size([1, 256, 28, 28])
after stage3 #7 : torch.Size([1, 512, 28, 28])
after stage3 #8 : torch.Size([1, 512, 28, 28])
after stage3 #9 : torch.Size([1, 1024, 14, 14])

after stage4 #0 : torch.Size([1, 512, 14, 14])
after stage4 #1 : torch.Size([1, 1024, 14, 14])
after stage4 #2 : torch.Size([1, 512, 14, 14])
after stage4 #3 : torch.Size([1, 1024, 14, 14])
after stage4 #4 : torch.Size([1, 1024, 14, 14])
after stage4 #5 : torch.Size([1, 1024, 7, 7])

after stage5 #0 : torch.Size([1, 1024, 7, 7])
after stage5 #1 : torch.Size([1, 1024, 7, 7])
```

### 完全连接层

骨干网络完成后，我们现在可以继续到完全连接的部分，我在下面的代码块 6 中写到了这部分。这部分网络非常简单，因为它主要只由两个线性层组成。关于细节，论文中提到作者在第一个（`#(1)`）和第二个（`#(4)`）线性层之间应用了一个丢弃层，丢弃率为 0.5（`#(3)`）。需要注意的是，漏激活函数（leaky ReLU）仍然被使用（`#(2)`），但只在第一个线性层之后使用。这是因为第二个层作为输出层，因此它不需要应用任何激活函数。

```py
# Codeblock 6
class FullyConnected(nn.Module):
    def __init__(self):
        super().__init__()

        self.linear0 = nn.Linear(in_features=1024*7*7, out_features=4096)   #(1)
        self.leaky_relu = nn.LeakyReLU(negative_slope=0.1)                  #(2)
        self.dropout = nn.Dropout(p=0.5)                                    #(3)
        self.linear1 = nn.Linear(in_features=4096, out_features=(C+B*5)*S*S)#(4)

    def forward(self, x):
        print(f'original\t: {x.size()}')

        x = self.linear0(x)
        print(f'after linear0\t: {x.size()}')

        x = self.leaky_relu(x)
        x = self.dropout(x)

        x = self.linear1(x)
        print(f'after linear1\t: {x.size()}')

        return x
```

运行下面的代码块 7，以查看张量在通过线性层堆栈处理时的变换情况。

```py
# Codeblock 7
fc = FullyConnected()
x = torch.randn(1, 1024*7*7)
out = fc(x)
```

```py
# Codeblock 7 Output
original      : torch.Size([1, 50176])
after linear0 : torch.Size([1, 4096])
after linear1 : torch.Size([1, 1470])
```

我们可以在上面的输出中看到，`fc`块接收一个形状为 50176 的输入，这实际上是 1024×7×7 张量的展平形式。`linear0`层通过将这个输入映射到 4096 维向量来工作，然后`linear1`层最终将其进一步映射到 1470。在后续的后处理阶段，我们需要将其重塑为 30×7×7，这样我们就可以轻松地获取边界框和对象分类结果。从技术上讲，这个重塑过程可以在模型内部或外部完成。为了简化，我决定保留输出为展平形式，这意味着重塑将由外部处理。

### 将 FC 部分连接到主干

到目前为止，我们已经完成了主干和全连接层的构建。因此，它们现在可以组装起来构建整个 YOLOv1 架构。关于下面的代码，我无法解释太多，因为我们在这里所做的只是实例化这两部分，并在`forward()`方法中将它们连接起来。只是不要忘记将`backbone`的输出展平（`#(1)`），以便与`fc`块的输入兼容。

```py
# Codeblock 8
class YOLOv1(nn.Module):
    def __init__(self):
        super().__init__()

        self.backbone = Backbone()
        self.fc = FullyConnected()

    def forward(self, x):
        x = self.backbone(x)
        x = torch.flatten(x, start_dim=1)    #(1)
        x = self.fc(x)

        return x
```

为了测试我们的模型，我们可以简单地实例化`YOLOv1`模型，并传递一个模拟 448×448 大小 RGB 图像的虚拟张量（`#(1)`）。在将张量输入到网络中（`#(2)`）后，我还尝试通过将输出张量重塑为 30×7×7 来模拟后处理步骤，如图`#(3)`所示。

```py
# Codeblock 9
yolov1 = YOLOv1()
x = torch.randn(1, 3, 448, 448)      #(1)

out = yolov1(x)                      #(2)
out = out.reshape(-1, C+B*5, S, S)   #(3)
```

下面是上述代码运行后的输出。在这里，你可以看到我们的输入张量成功流经整个网络的所有层，这表明我们的 YOLOv1 模型工作正常，因此可以准备训练。

```py
# Codeblock 9 Output
original        : torch.Size([1, 3, 448, 448])

after stage0    : torch.Size([1, 64, 112, 112])

after stage1    : torch.Size([1, 192, 56, 56])

after stage2 #0 : torch.Size([1, 128, 56, 56])
after stage2 #1 : torch.Size([1, 256, 56, 56])
after stage2 #2 : torch.Size([1, 256, 56, 56])
after stage2 #3 : torch.Size([1, 512, 28, 28])

after stage3 #0 : torch.Size([1, 256, 28, 28])
after stage3 #1 : torch.Size([1, 512, 28, 28])
after stage3 #2 : torch.Size([1, 256, 28, 28])
after stage3 #3 : torch.Size([1, 512, 28, 28])
after stage3 #4 : torch.Size([1, 256, 28, 28])
after stage3 #5 : torch.Size([1, 512, 28, 28])
after stage3 #6 : torch.Size([1, 256, 28, 28])
after stage3 #7 : torch.Size([1, 512, 28, 28])
after stage3 #8 : torch.Size([1, 512, 28, 28])
after stage3 #9 : torch.Size([1, 1024, 14, 14])

after stage4 #0 : torch.Size([1, 512, 14, 14])
after stage4 #1 : torch.Size([1, 1024, 14, 14])
after stage4 #2 : torch.Size([1, 512, 14, 14])
after stage4 #3 : torch.Size([1, 1024, 14, 14])
after stage4 #4 : torch.Size([1, 1024, 14, 14])
after stage4 #5 : torch.Size([1, 1024, 7, 7])

after stage5 #0 : torch.Size([1, 1024, 7, 7])
after stage5 #1 : torch.Size([1, 1024, 7, 7])

original        : torch.Size([1, 50176])
after linear0   : torch.Size([1, 4096])
after linear1   : torch.Size([1, 1470])

torch.Size([1, 30, 7, 7])
```

* * *

## 结束

值得注意的是，我在整篇文章中展示的所有代码都是针对基础 YOLOv1 架构的。论文中提到，作者还提出了这个模型的轻量级版本，他们称之为*Fast YOLO*。这个更小的 YOLOv1 版本提供了更快的计算时间，因为它只包含 9 个卷积层，而不是 24 个。不幸的是，论文没有提供实现细节，所以我无法演示如何实现它。

我鼓励你尝试玩一下上面的代码。从理论上讲，我们可以用其他深度学习模型替换基于 CNN 的主干，例如 ResNet、ResNeXt、ViT 等。你所需要做的只是确保主干输出形状与全连接部分的输入形状相匹配。不仅如此，我还想让你尝试从头开始训练这个模型。但如果你决定这样做，你可能需要通过减少模型的深度（卷积层的数量）或宽度（核的数量）来缩小这个模型。这是因为作者提到，他们需要大约一周的时间来在 ImageNet 数据集上进行预训练，更不用说在目标检测任务上进行微调的时间了。

好吧，我认为这就是我能解释的关于 YOLOv1 的工作原理及其架构的几乎所有内容了。如果您在这篇文章中发现了任何错误，请告诉我。谢谢！

*顺便说一下，本文中使用的代码也可在我的 GitHub 仓库[7]中找到。*

* * *

## 参考文献

[1] Joseph Redmon 等人. 你只需看一次：统一、实时目标检测。Arxiv. [`arxiv.org/pdf/1506.02640`](https://arxiv.org/pdf/1506.02640) [访问日期：2025 年 7 月 5 日].

[2] Ross Girshick 等人. 用于精确目标检测和语义分割的丰富特征层次结构。Arxiv. [`arxiv.org/pdf/1311.2524`](https://arxiv.org/pdf/1311.2524) [访问日期：2025 年 7 月 5 日].

[3] Mengqi Lei 等人. YOLOv13：使用超图增强自适应视觉感知的实时目标检测。Arxiv. [`arxiv.org/abs/2506.17733`](https://arxiv.org/abs/2506.17733) [访问日期：2025 年 7 月 5 日].

[4] 由作者使用 Gemini 生成并编辑的图像。

[5] 原始图像由作者创建。

[6] Bing Xu 等人. 卷积网络中修正激活的实证评估。Arxiv. [`arxiv.org/pdf/1505.00853`](https://arxiv.org/pdf/1505.00853) [访问日期：2025 年 7 月 5 日].

[7] MuhammadArdiPutra. YOLO 首次窥见世界的日子—YOLOv1. GitHub. [`github.com/MuhammadArdiPutra/medium_articles/blob/main/The%20Day%20YOLO%20First%20Saw%20the%20World%20-%20YOLOv1.ipynb`](https://github.com/MuhammadArdiPutra/medium_articles/blob/main/The%20Day%20YOLO%20First%20Saw%20the%20World%20-%20YOLOv1.ipynb) [访问日期：2025 年 7 月 7 日].

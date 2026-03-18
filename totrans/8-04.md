# 使用 Blender 扩展分割：如何自动化数据集创建

> 原文：[`towardsdatascience.com/scaling-segmentation-with-blender-how-to-automate-dataset-creation-73aa38967599/`](https://towardsdatascience.com/scaling-segmentation-with-blender-how-to-automate-dataset-creation-73aa38967599/)

![由 Lina Trochez 在 Unsplash 上拍摄的照片](img/14369f318a959c27866662a50758f356.png)

由[Lina Trochez](https://unsplash.com/@lmtrochezz?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)拍摄的照片

如果你曾经为新的项目训练过分割模型，你可能知道这不仅仅关于模型。这是关于数据。

收集图像通常很简单；你通常可以在像[Unsplash](https://unsplash.com/)这样的平台上找到很多，甚至可以使用像 Stable Diffusion 这样的生成式 AI 工具来生成更多：

> [**如何在没有训练数据的情况下训练实例分割模型**](https://towardsdatascience.com/how-to-train-an-instance-segmentation-model-with-no-training-data-190dc020bf73)

主要挑战通常在于标注。为分割标注图像是非常耗时的。即使有像 Meta 的 SAM2 这样的高级工具，创建一个完全标注、稳健且多样化的数据集仍然需要相当多的时间。

在这篇文章中，我们将探讨另一个经常被忽视的选项：使用 3D 工具，例如[Blender](https://www.blender.org/)。确实，3D 引擎越来越强大和逼真。此外，它们提供了一个引人注目的优势：在创建数据集的同时自动生成标签，消除了手动标注的需求。

在这篇文章中，我们将概述创建手部分割模型的完整解决方案，分为以下关键部分：

+   使用 Blender 生成手部图像以及如何获得手部姿势、位置和肤色的多样性

+   使用生成的 Blender 图像和选定的背景图像生成数据集，使用[OpenCV](https://opencv.org/)

+   使用 PyTorch 训练和评估模型

当然，本文中使用的所有代码都是完全可用和可重用的，在以下[GitHub](https://github.com/vincent-vdb/medium_posts)仓库中。

## 生成手部图像

要生成手部图像，让我们使用 Blender。我不是这类工具的专家，但它为我们提供了许多高度有用的功能：

+   它是免费的——没有商业许可，任何人都可以立即下载和使用

+   有一个庞大的社区，可以在网上找到许多模型，有些是免费的，有些则不是

+   最后，它包括一个 Python API，使图像生成的自动化具有各种功能

正如我们将看到的，这些功能非常有用，将使我们能够轻松地生成合成数据。为了确保足够的多样性，我们将探讨如何自动随机化以下参数在我们的生成手中：

+   手指位置：我们希望有各种不同位置的手部图像

+   相机位置：我们希望从各种角度拍摄手部图像

+   肤色：我们希望肤色具有多样性，使模型足够健壮

*注意：这里提出的方法并非没有基于肤色的潜在偏见，也不声称是无偏见的。任何基于此方法的产品都必须仔细评估其伦理偏见。*

在深入这些步骤之前，我们需要一个手部的 3D 模型。在诸如[Turbosquid](https://www.turbosquid.com/)等网站上有很多模型，但我使用了一个可以在此找到的免费手部模型。如果你用 Blender 打开这个文件，你会得到以下截图类似的效果。

![在 Blender 中打开的手部模型截图。图片由作者提供。](img/b8e12c0e9f25168555c7a799954f9b41.png)

在 Blender 中打开的手部模型截图。图片由作者提供。

如所示，该模型不仅包括手部的形状和纹理，还包括骨骼结构，这使得手部运动模拟成为可能。让我们从这个基础出发，通过调整手指位置、肤色和相机位置来获得一组多样化的手部。

### 修改手指位置

第一步是确保一组多样化且逼真的手指位置。不深入太多细节（因为这更多与 Blender 本身相关），我们需要创建控制器以进行移动并施加允许的移动限制。基本上，我们不希望手指向后折叠或以不现实的方向弯曲。更多关于这些步骤的细节，请参考[这个 YouTube 教程](https://www.youtube.com/watch?v=wBfSA1mDATY)，它帮助我以最小的努力实现了这些功能。

一旦 Blender 文件设置得当，并有了正确的限制，我们就可以使用 Python 脚本来自动化任何手指位置：

如我们所见，我们只是随机更新控制器的位置，允许在限制下移动手指。有了正确的限制集，我们得到的手指位置看起来如下：

![随机手指位置生成的图像样本。图片由作者提供。](img/4085ce22e66c4a926a85fcf2d38cbcaf.png)

随机手指位置生成的图像样本。图片由作者提供。

这产生了逼真且多样的手指位置，最终能够生成一系列多样的手部图像。现在，让我们来调整肤色。

### 修改肤色

当创建一个包含人物的新的图像数据集时，最具挑战性的方面之一可能是实现足够广泛的肤色代表性。确保模型在所有肤色上都能高效工作且无偏见是一个关键优先事项。尽管我并不声称解决任何偏见，但我提出的方法允许通过自动改变肤色来找到一个解决方案。

*注意：这种方法并不声称使模型完全摆脱任何伦理偏见。任何用于生产的模型都必须经过公平性评估的仔细测试。可以参考谷歌为他们的面部检测模型所做的工作[what has been done by Google for their face detection models](https://storage.googleapis.com/mediapipe-assets/MediaPipe%20BlazeFace%20Model%20Card%20(Short%20Range).pdf)作为一个例子。*

我在这里对图像进行的是纯图像处理计算。想法很简单：给定一个目标颜色和渲染手部的平均颜色，我将简单地计算这两种颜色之间的差异。然后，我将把这个差异应用到渲染的手部上，以得到新的肤色：

因此，它给出了以下手部图像：

![随机肤色生成图像样本。图由作者提供。](img/cc3d2d284d9403cc592277d8a004aefe.png)

随机肤色生成图像样本。图由作者提供。

虽然结果并不完美，但它们使用简单的图像处理产生了具有多样肤色的合理逼真图像。只剩下一步，那就是拥有足够多样的图像集：渲染视角。

### 调整相机位置

最后，让我们调整相机位置以从多个角度捕捉手部。为了实现这一点，相机被放置在围绕手部的球体上的一个随机点上。这可以通过简单地调整球坐标的两个角度轻松实现。在下面的代码中，我生成了一个球体上的随机位置：

然后，通过添加一些关于球面位置的约束，我可以用 Blender 更新手部的相机位置：

因此，我们现在得到了以下图像样本：

![随机手指位置、肤色和相机位置生成图像样本。图由作者提供。](img/f328f24464ad812d67ee371e5148a3b2.png)

随机手指位置、肤色和相机位置的生成图像样本。图由作者提供。

现在我们有了具有不同手指位置、肤色和不同视角的手部图像。在训练分割模型之前，下一步是实际生成各种背景和情境下的手部图像。

## 生成训练数据

为了生成多样且足够逼真的图像，我们将把生成的手部图像与一组选定的背景图像混合。

我在 Unsplash 上获取了图像，作为背景图像，这些图像是免费使用的，并且不包含手部。然后，我将随机将这些 Blender 生成的手部图像添加到这些背景图像上：

这个函数虽然很长，但只执行简单的操作：

+   加载一个随机的手部图像和遮罩

+   加载一个随机的背景图像

+   调整背景图像的大小

+   在背景图像中随机选择一个位置放置手部

+   计算新的遮罩

+   计算背景和手部的混合图像

因此，生成数百甚至数千张图像及其标签以供分割任务使用相当容易。以下是生成图像的样本：

![一张带有背景和 Blender 生成的手的生成图像样本。图由作者提供。](img/79826084bb48d38475b04a9ff590e45c.png)

一张带有背景和 Blender 生成的手的生成图像样本。图由作者提供。

使用这些生成的图像和掩码，我们现在可以继续下一步：训练一个分割模型。

## 训练和评估分割模型

现在我们已经正确生成了数据，让我们在它上面训练一个分割模型。让我们首先谈谈训练流程，然后评估使用这些生成数据的益处。

### 训练模型

我们将使用 PyTorch 来训练模型，以及[Segmentation Models Pytorch](https://github.com/qubvel-org/segmentation_models.pytorch)库，它允许轻松训练许多分割模型。

以下代码片段允许模型训练：

这段代码执行了模型训练的典型步骤：

+   实例化训练和验证数据集，以及数据加载器

+   实例化模型本身

+   定义损失函数和优化器

+   训练模型并保存

模型本身有几个输入参数：

+   编码器，从[这个列表](https://github.com/qubvel-org/segmentation_models.pytorch?tab=readme-ov-file#encoders)中选择已实现的模型，例如我这里使用的[MobileNetV3](https://arxiv.org/abs/1905.02244)

+   在[ImageNet 数据集](https://www.image-net.org/)上的初始化权重

+   输入通道的数量，这里为 3，因为我们使用彩色图像

+   输出通道的数量，这里为 1，因为只有一个类别

+   输出激活函数：这里的 sigmoid，因为只有一个类别

如果你想了解更多信息，完整的实现可以在 GitHub 上找到。

### 评估模型

为了评估模型以及混合图像的改进，让我们进行以下比较：

+   在[Ego Hands 数据集](http://vision.soic.indiana.edu/projects/egohands/)上训练和评估一个模型

+   在 Ego Hands 数据集上，添加我们的混合生成的数据到训练集，训练和评估相同的模型

在这两种情况下，我将在 Ego Hands 数据集的相同子集上评估模型。作为一个评估指标，我将使用[交并比 (IoU)](https://en.wikipedia.org/wiki/Jaccard_index)（也称为 Jaccard 指数）。以下是结果：

+   仅在 Ego Hands 数据集上，经过 20 个 epoch 后：**IoU = 0.72**

+   在 Ego Hands 数据集和 Blender 生成的图像上，经过 20 个 epoch 后：**IoU = 0.76**

如我们所见，我们可以通过使用由 Blender 生成的图像组成的数据集，将 IoU 从 0.72 提高到 0.76，这是一个显著的提升。

### 测试模型

对于任何愿意在自己的计算机上尝试此模型的人，我还添加了一个 GitHub 脚本，以便它在实时摄像头流中运行。

由于我训练了一个相对较小的模型（MobileNetV3 Large 100），大多数现代笔记本电脑都应能有效运行此代码。

## 结论

让我们以几个关键要点来结束这篇文章：

+   Blender 是一款强大的工具，它允许你在各种条件下生成逼真的图像：光线、相机位置、变形等……

+   利用 Blender 生成合成数据可能最初需要一些时间，但可以使用 Python API 完全自动化

+   使用生成的数据提高了语义分割任务的模型性能：它将 IoU 从 0.72 提高到 0.76

+   为了获得更加多样化的数据集，可以使用更多的 Blender 手部模型：更多的手部形状，更多的纹理可以帮助分割模型更好地泛化

最后，如果你成功构建了一个工作模型，并想找到最佳的部署策略，你可以查看以下指南：

> [**如何选择最佳的机器学习部署策略：云端与边缘**](https://towardsdatascience.com/how-to-choose-the-best-ml-deployment-strategy-cloud-vs-edge-7b62d9db9b20)

作为旁注，虽然这篇文章主要关注语义分割，但这种方法可以适应其他计算机视觉任务，包括实例分割、分类和地标预测。我很乐意听到其他可能的我可能遗漏的 Blender 的潜在用途。

## 参考文献

这里有一些参考文献，尽管它们已经在文章中提到过：

+   [OpenCV](https://opencv.org/)

+   [MobileNetV3](https://arxiv.org/abs/1905.02244)

+   [用于训练的分割模型 PyTorch](https://github.com/qubvel-org/segmentation_models.pytorch)

+   [Blender](https://www.blender.org/) 及其 [Python API](https://docs.blender.org/api/current/index.html)

+   [Ego Hands 数据集](http://vision.soic.indiana.edu/projects/egohands/)

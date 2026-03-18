# PyTorch 中叶子张量的真正含义及其梯度

> 原文：[`towardsdatascience.com/what-pytorch-really-means-by-a-leaf-tensor-2/`](https://towardsdatascience.com/what-pytorch-really-means-by-a-leaf-tensor-2/)

<mdspan datatext="el1750293464837" class="mdspan-comment">这篇帖子</mdspan>并不是对链式法则的另一种解释。它是对 autograd 奇特一面的游览——在这里，梯度服务于物理，而不仅仅是权重。

我最初在攻读博士学位的第一年写了这个教程，当时我在 PyTorch 中导航梯度计算的复杂性。大部分内容显然是针对标准反向传播设计的——这是可以的，因为这是大多数人需要的。

但物理信息神经网络（PINN）是一个情绪化的生物，它需要不同类型的梯度逻辑。我花了一些时间来喂养它，并认为将发现与社区分享可能是有价值的，特别是与 PINN 实践者——也许它能节省某人一些头疼。但如果你从未听说过 PINNs，不用担心！这篇帖子对你来说仍然适用——特别是如果你对梯度、梯度等的有趣内容感兴趣。

## 基本术语

在计算机世界中，**张量**意味着一个多维数组，即一组由一个或多个整数索引的数字。更准确地说，也存在零维张量，它们只是单个数字。有些人说张量是矩阵在超过两个维度上的推广。

如果你之前研究过广义相对论，你可能听说过数学张量有协变和逆变指标这样的东西。但忘掉它吧——在 PyTorch 中，张量只是多维数组。这里没有技巧。

**叶子张量**是一个计算图中的叶子（在图论意义上的叶子）。我们将在下面查看这些内容，所以这个定义将更有意义。

张量的`requires_grad`属性告诉 PyTorch 是否应该记住这个张量在后续计算中的使用情况。目前，可以将`requires_grad=True`的张量视为变量，而将`requires_grad=False`的张量视为常量。

## 叶子张量

让我们从创建几个张量并检查它们的属性`requires_grad`和`is_leaf`开始。

```py
import torch

a = torch.tensor([3.], requires_grad=True)
b = a * a

c = torch.tensor([5.])
d = c * c

assert a.requires_grad is True and a.is_leaf is True
assert b.requires_grad is True and b.is_leaf is False
assert c.requires_grad is False and c.is_leaf is True
assert d.requires_grad is False and d.is_leaf is True  # sic!
del a, b, c, d
```

`a`是一个叶子张量，正如预期的那样，而`b`不是，因为它是一个乘法的结果。`a`被设置为需要梯度，所以自然地`b`继承了这一属性。

`c`显然是一个叶子张量，但为什么`d`是叶子张量呢？`d.is_leaf`为真的原因源于一个特定的约定：所有`requires_grad`设置为 False 的张量都被认为是叶子张量，如[PyTorch 文档](https://pytorch.org/docs/stable/generated/torch.Tensor.is_leaf.html)中所述：

> 所有具有`requires_grad`（[PyTorch 文档](https://docs.pytorch.org/docs/stable/generated/torch.Tensor.requires_grad.html#torch.Tensor.requires_grad)）属性为`False`的张量按惯例都是叶子张量。

虽然在数学上 `d` 不是一个叶子（因为它是由另一个操作 `c * c` 得出的），梯度计算永远不会超出它。换句话说，不会有关于 `c` 的导数。这允许 `d` 被视为一个叶子。

简而言之，在 PyTorch 中，叶子张量可以是：

+   直接输入（即不是从其他张量计算得出）并且有 `requires_grad=True`。例如：随机初始化的神经网络权重。

+   完全不需要梯度，无论它们是直接输入还是计算得出。在 autograd 的眼中，这些只是常数。例如：

    +   任何神经网络输入数据，

    +   一个输入图像在去除均值或其他操作后的情况，这仅涉及不要求梯度的张量。

一个小备注，对于那些想了解更多的人来说。`requires_grad` 属性如下所示继承：

```py
a = torch.tensor([5.], requires_grad=True)
b = torch.tensor([5.], requires_grad=True)
c = torch.tensor([5.], requires_grad=False)

d = torch.sin(a * b * c)

assert d.requires_grad == any((x.requires_grad for x in (a, b, c)))
```

*代码备注：所有代码片段都应该自包含，除了我首次包含的导入。我省略它们是为了最小化样板代码。我相信读者能够轻松处理这些内容。*

## 梯度保留

一个单独的问题是梯度保留。计算图中所有节点，即所有使用的张量，如果需要梯度，都会计算梯度。然而，只有叶子张量保留这些梯度。这很有道理，因为梯度通常用于更新张量，只有叶子张量在训练过程中会进行更新。非叶子张量，如第一个例子中的 `b`，不会直接更新；它们会随着 `a` 的变化而变化，因此它们的梯度可以被丢弃。然而，在某些情况下，尤其是在物理信息神经网络（PINNs）中，你可能希望保留这些中间张量的梯度。在这种情况下，你需要显式标记非叶子张量以保留它们的梯度。让我们看看：

```py
a = torch.tensor([3.], requires_grad=True)
b = a * a
b.backward()

assert a.grad is not None
assert b.grad is None  # generates a warning
```

你可能刚刚看到了一个警告：

```py
UserWarning: The .grad attribute of a Tensor that is not a leaf Tensor is being 
accessed. Its .grad attribute won't be populated during autograd.backward(). 
If you indeed want the .grad field to be populated for a non-leaf Tensor, use 
.retain_grad() on the non-leaf Tensor. If you access the non-leaf Tensor by 
mistake, make sure you access the leaf Tensor instead. 
See github.com/pytorch/pytorch/pull/30531 for more informations. 
(Triggered internally at aten\src\ATen/core/TensorBody.h:491.)
```

因此，让我们通过强制 `b` 保留其梯度来修复它。

```py
a = torch.tensor([3.], requires_grad=True)
b = a * a
b.retain_grad()  # <- the difference
b.backward()

assert a.grad is not None
assert b.grad is not None
```

## 梯度的奥秘

现在让我们看看著名的梯度本身。它是什么？它是一个张量吗？如果是，它是一个叶子张量吗？它需要或保留梯度吗？

```py
a = torch.tensor([3.], requires_grad=True)
b = a * a
b.retain_grad()
b.backward()

assert isinstance(a.grad, torch.Tensor)
assert a.grad.requires_grad is False and a.grad.retains_grad is False and a.grad.is_leaf is True
assert b.grad.requires_grad is False and b.grad.retains_grad is False and b.grad.is_leaf is True
```

显然：

**–** 梯度本身是一个张量，

**–** 梯度是一个叶子张量，

**–** 梯度不需要梯度。

它保留梯度吗？这个问题没有意义，因为它最初就不需要梯度。我们将在下一秒回到梯度是否是叶子张量的问题，但现在我们将测试一些事情。

### 多次反向传播和 `retain_graph`

当我们两次计算相同的梯度时会发生什么？

```py
a = torch.tensor([3.], requires_grad=True)
b = a * a
b.retain_grad()
b.backward()
try:
    b.backward()
except RuntimeError:
    """
    RuntimeError: Trying to backward through the graph a second time (or 
    directly access saved tensors after they have already been freed). Saved 
    intermediate values of the graph are freed when you call .backward() or 
    autograd.grad(). Specify retain_graph=True if you need to backward through 
    the graph a second time or if you need to access saved tensors after 
    calling backward.
    """
```

错误信息解释了一切。这应该可以工作：

```py
a = torch.tensor([3.], requires_grad=True)
b = a * a
b.retain_grad()

b.backward(retain_graph=True)
print(a.grad)  # prints tensor([6.])

b.backward(retain_graph=True)
print(a.grad)  # prints tensor([12.])

b.backward(retain_graph=False)
print(a.grad)  # prints tensor([18.])

# b.backward(retain_graph=False)  # <- here we would get an error, because in 
# the previous call we did not retain the graph.
```

侧（但很重要）注：你还可以观察梯度如何在 `a` 中累积：每次迭代都会增加。

### 强大的 `create_graph` 参数

如何使梯度需要梯度？

```py
a = torch.tensor([5.], requires_grad=True)
b = a * a
b.retain_grad()
b.backward(create_graph=True)

# Here an interesting thing happens: now a.grad will require grad! 
assert a.grad.requires_grad is True
assert a.grad.is_leaf is False

# On the other hand, the grad of b does not require grad, as previously. 
assert b.grad.requires_grad is False
assert b.grad.is_leaf is True
```

上述内容非常有用：`a.grad` 在数学上表示 \[\frac{\partial b}{\partial a}\] 不再是一个常数（叶子节点），而是计算图中的一员，可以进一步使用。我们将在第二部分中使用这个事实。

为什么 `b.grad` 不需要求导？因为 `b` 对 `b` 的导数简单地为 1。

如果你现在觉得 `backward` 感觉不太直观，不要担心。我们很快就会切换到另一种称为 nomen omen `grad` 的方法，它允许我们精确选择导数的成分。在此之前，有两个旁注：

**旁注 1**：如果你将 `create_graph` 设置为 True，它也会将 `retain_graph` 设置为 True（如果未明确设置）。在 PyTorch 代码中，它看起来完全像

这：

```py
 if retain_graph is None:
        retain_graph = create_graph
```

**旁注 2**：你可能看到了这样的警告：

```py
 UserWarning: Using backward() with create_graph=True will create a reference 
    cycle between the parameter and its gradient which can cause a memory leak. 
    We recommend using autograd.grad when creating the graph to avoid this. If 
    you have to use this function, make sure to reset the .grad fields of your 
    parameters to None after use to break the cycle and avoid the leak. 
    (Triggered internally at C:\cb\pytorch_1000000000000\work\torch\csrc\autograd\engine.cpp:1156.)
      Variable._execution_engine.run_backward(  # Calls into the C++ engine to 
    run the backward pass
```

我们将遵循建议，现在使用 `autograd.grad`。

## 使用 `autograd.grad` 函数求导

现在让我们从某种程度上的高级 `.backward()` 方法转向更底层的 `grad` 方法，该方法明确地计算一个张量相对于另一个张量的导数。

```py
from torch.autograd import grad

a = torch.tensor([3.], requires_grad=True)
b = a * a * a
db_da = grad(b, a, create_graph=True)[0]
assert db_da.requires_grad is True
```

同样，与 `backward` 类似，`b` 对 `a` 的导数可以被视为一个函数并进一步求导。所以换句话说，`create_graph` 标志可以理解为：*在计算梯度时，保留它们是如何计算的记录，这样我们可以将它们视为需要求导的非叶张量，并进一步使用。*

特别是，我们可以计算二阶导数：

```py
d2b_da2 = grad(db_da, a, create_graph=True)[0]
# Side note: the grad function returns a tuple and the first element of it is what we need.
assert d2b_da2.item() == 18
assert d2b_da2.requires_grad is True
```

正如之前所说：这实际上是允许我们使用 PyTorch 进行 PINN 的关键属性。

## 总结

大多数关于 PyTorch 梯度的教程都集中在经典监督学习中的反向传播。这个教程探索了一个不同的视角——一个由 PINNs 和其他对梯度有强烈需求的生物的需求所塑造的视角。

我们学习了 PyTorch 森林中的叶子是什么，为什么默认情况下只保留叶节点的梯度，以及如何在需要时为其他张量保留它们。我们看到了 `create_graph` 如何将梯度转化为 autograd 世界中的可导公民。

但仍有许多东西需要揭示——特别是为什么非标量函数的梯度需要额外小心，如何在不使用整个 RAM 的情况下计算二阶导数，以及为什么在需要元素级梯度时切片输入张量是个坏主意。

所以让我们在第二部分见面，我们将更仔细地看看 `grad` 👋

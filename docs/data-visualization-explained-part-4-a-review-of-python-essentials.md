# 数据可视化解析（第四部分）：Python 基础知识回顾

> 原文：[`towardsdatascience.com/data-visualization-explained-part-4-a-review-of-python-essentials/`](https://towardsdatascience.com/data-visualization-explained-part-4-a-review-of-python-essentials/)

*<mdspan datatext="el1761243970023" class="mdspan-comment">这是我的数据可视化系列中的第四篇文章。请参阅以下内容：*</mdspan>

+   *[第一部分：“数据可视化解析：它是什么以及为什么它很重要”](https://towardsdatascience.com/data-visualization-explained-what-it-is-and-why-it-matters/)*“

+   *[第二部分：“数据可视化解析：视觉变量的介绍”](https://towardsdatascience.com/data-visualization-explained-part-2-an-introduction-to-visual-variables/)*。”

+   *[第三部分：“数据可视化解析：颜色的作用。”](https://towardsdatascience.com/data-visualization-explained-part-3-the-role-of-color/)*

在我的数据可视化系列中，到目前为止，我已经涵盖了可视化设计的基石。在真正设计和构建可视化之前，理解这些原则是至关重要的，因为它们确保了底层数据得到公正的处理。如果你还没有这样做，我强烈建议你阅读我之前的文章（如上链接）。

到目前为止，你已经准备好开始构建我们自己的可视化。我将在未来的文章中介绍各种实现方法——并且本着数据科学的精神，许多这些方法将需要编程。为了确保你为这一步做好准备，这篇文章将包括对 Python 基础知识的简要回顾，然后讨论它们在编写数据可视化代码中的相关性。

## 基础知识——表达式、变量、函数

表达式、变量和函数是所有 Python 代码——以及任何语言代码——的主要构建块。让我们看看它们是如何工作的。

### 表达式

**表达式**是一个返回某个值的语句。最简单的表达式是任何类型的常量值。例如，下面有三个简单的表达式：第一个是整数，第二个是字符串，第三个是浮点数。

```py
7
'7'
7.0
```

更复杂的表达式通常由数学运算组成。我们可以添加、减去、乘以或除以各种数字：

```py
3 + 7
820 - 300
7 * 53
121 / 11
6 + 13 - 3 * 4
```

根据定义，这些表达式由 Python 评估为单个值，遵循由缩写词 [PEMDAS](https://www.khanacademy.org/math/cc-sixth-grade-math/x0267d782:cc-6th-exponents-and-order-of-operations/cc-6th-order-of-operations/v/more-complicated-order-of-operations-example)（括号、指数、乘法、除法、加法、减法）概述的数学运算顺序。例如，上面的最终表达式评估为数字 `7.0`。（你明白为什么吗？）

### 变量

表达式很棒，但它们本身并不是非常有用。在编程时，您通常需要保存某些表达式的值，以便您可以在程序的后续部分中使用它们。**变量** 是一个容器，它保存表达式的值，并允许您稍后访问它。以下是与上面第一个示例中完全相同的表达式，但这次它们的值被保存在各种变量中：

```py
int_seven = 7
text_seven = '7'
float_seven = 7.0
```

Python 中的变量有几个重要的属性：

+   变量的 **名称**（等号左边的单词）必须是一个单词，并且不能以数字开头。如果您需要在变量名称中包含多个单词，惯例是使用下划线（如上面的示例所示）将它们分开。

+   在我们使用 Python 中的变量时，您不需要指定数据类型，就像您在用其他语言编程时可能习惯做的那样。这是因为 Python 是一种 *动态类型* 语言。

+   一些其他编程语言区分变量的 **声明** 和 **赋值**。在 Python 中，我们只是在声明变量的同一行中赋值，因此没有必要区分。

当声明变量时，Python 总会将等号右侧的表达式评估为单个值，然后再将其分配给变量。（这回到了 Python 如何评估复杂表达式的方式）。以下是一个示例：

```py
yet_another_seven = (2 * 2) + (9 / 3)
```

上面的变量被分配了值 `7.0`，而不是复合表达式 `(2 * 2) + (9 / 3)`。

### 函数

您可以将 **函数** 视为一种机器。它接受一些（或多个）东西，运行一些代码来转换您传递进来的对象（或多个对象），并输出一个精确的值。在 Python 中，函数主要用于两个主要原因：

1.  为了操作感兴趣的输入变量并得出所需的输出（就像数学函数一样）。

1.  为了避免代码重复。通过将代码包装在函数中，我们可以在需要运行该代码时随时调用该函数（而不是一次又一次地编写相同的代码）。

理解如何在 Python 中定义函数的最简单方法就是查看一个示例。下面，我们编写了一个简单的函数，该函数将数字的值加倍：

```py
def double(num):
    doubled_value = num * 2
    return doubled_value

print(double(2))    # outputs 4
print(double(4))    # outputs 8
```

关于上面的示例，您应该确保理解以下几个重要点：

+   `def` 关键字告诉 Python 您想要定义一个函数。`def` 后面的单词是函数的名称，因此上面的函数被称为 `double`。

+   在名称之后，有一组括号，您可以在其中放置函数的参数（这是一个术语，意思是函数的输入）。**重要提示**：如果您的函数不需要任何参数，您仍然需要包括括号——只是不要在它们里面放任何内容。

+   在`def`语句的末尾必须使用冒号，否则 Python 将不会高兴（即，它会抛出一个错误）。整个带有`def`语句的行被称为**函数签名**。

+   `def`语句之后的所有行都包含构成函数的代码，缩进一级。这些行共同构成了**函数体**。

+   上面的函数的最后一行是**返回语句**，它使用`return`关键字指定函数的输出。返回语句不一定是函数的最后一行，但在遇到它之后，Python 将退出函数，并且不会运行更多的代码行。更复杂的函数可能有多个返回语句。

+   你通过写出函数的名称，并在括号中放入所需的输入来调用一个函数。如果你调用一个没有输入的函数，你仍然需要包括括号。

## Python 和数据可视化

现在让我们来回答你可能正在问自己的问题：为什么一开始要进行所有的 Python 复习？毕竟，你可以用很多种方式来可视化数据，而且它们肯定不是都受限于对 Python 的了解，甚至不是受限于一般的编程知识。

这一点是正确的，但作为一个数据科学家，你很可能会在某些时候需要编程——在编程中，你使用的语言极有可能是 Python。当你刚刚从团队的数据工程师那里接手一个数据清洗和分析流程时，了解如何快速有效地将其转化为一系列可操作和可展示的视觉洞察力是非常有用的。

Python 对于数据可视化来说非常重要，原因有几个：

+   它是一种易于使用的语言。如果你刚刚开始转向数据科学和可视化工作，用 Python 编写可视化程序会比使用像 JavaScript 中的[D3](https://d3js.org/)这样的底层工具要容易得多。

+   Python 中有许多不同且流行的库，它们都能提供使用代码来可视化数据的能力，这些代码直接建立在上面我们学到的 Python 基础知识之上。例如包括[Matplotlib](https://matplotlib.org/)、[Seaborn](https://seaborn.pydata.org/)、[Plotly](https://plotly.com/)和[Vega-Altair](https://altair-viz.github.io/)（之前仅被称为 Altair）。我将在未来的文章中探讨其中的一些，特别是 Altair。

+   此外，上述所有库都无缝集成到 Python 的基础数据科学库 pandas 中。pandas 中的数据可以直接集成到这些库的代码逻辑中，以构建可视化；你通常甚至不需要在开始可视化之前导出或转换它。

+   **本文讨论的基本原则可能看起来很基础，但它们对于实现数据可视化起到了很重要的作用：**

    +   正确计算表达式和理解他人编写的表达式对于确保你能够可视化数据的准确表示至关重要。

    +   你通常会需要存储特定的值或值集以供以后在可视化中使用——你需要变量来做到这一点。

        +   有时，你甚至可以将整个*可视化*存储在变量中以供以后使用或显示。

    +   更高级的库，如 Plotly 和 Altair，允许你调用内置（有时甚至是用户自定义）的函数来自定义可视化。

    +   基本的 Python 知识将使你能够将你的可视化集成到简单的应用程序中，这些应用程序可以与他人共享，使用工具如[Plotly Dash](https://dash.plotly.com/tutorial)和[Streamlit](https://docs.streamlit.io/get-started)。这些工具旨在简化对于编程新手的数据科学家构建应用程序的过程，而本文中涵盖的基础概念将足以让你开始使用它们。

如果这还不足以说服你，我强烈建议你点击上面的链接之一，开始探索这些可视化工具。一旦你开始看到你可以用它们做什么，你就不会回头了。

对于我来说，我将在下一篇文章中回来展示我自己的可视化教程。（这些工具中的一个或多个可能会出现。）在此之前！

## 参考文献

+   [`www.khanacademy.org/math/cc-sixth-grade-math/x0267d782:cc-6th-exponents-and-order-of-operations/cc-6th-order-of-operations/v/more-complicated-order-of-operations-example`](https://www.khanacademy.org/math/cc-sixth-grade-math/x0267d782:cc-6th-exponents-and-order-of-operations/cc-6th-order-of-operations/v/more-complicated-order-of-operations-example)

+   [`dash.plotly.com/tutorial`](https://dash.plotly.com/tutorial)

+   [`docs.streamlit.io/get-started`](https://docs.streamlit.io/get-started)

# PyScript vs. JavaScript：网络巨头的对决

> 原文：[`towardsdatascience.com/pyscript-vs-javascript-a-battle-of-web-titans/`](https://towardsdatascience.com/pyscript-vs-javascript-a-battle-of-web-titans/)

<mdspan datatext="el1743466709098" class="mdspan-comment">我们今天深入探讨前端网络开发，你可能想知道：这与数据科学有什么关系？为什么 Towards Data Science 会发布一篇与网络开发相关的帖子？</mdspan>

好吧，因为数据科学不仅仅是构建强大的模型、进行高级分析或清理和转换数据——**展示结果是我们的工作关键部分**。而且有几种方式可以做到这一点：PowerPoint 演示文稿、交互式仪表板（如 Tableau），或者，正如你所猜到的，通过网站。

从个人经验来说，我每天都在开发我们用来展示数据驱动结果的网站。使用网站而不是 PowerPoint 或 Tableau 有许多优点，其中**自由度和定制性**是最大的。

尽管我已经（某种程度上）喜欢上了 JavaScript，但它永远不会与 Python 编码的乐趣相匹配。幸运的是，在 FOSDEM 上，我了解了**PyScript**，令我惊讶的是，它并不像最初我想的那样处于 alpha 阶段。

但这足够称之为 JavaScript 的潜在替代品吗？这正是我们今天要探讨的。

JavaScript 已经数十年来是网络开发的王者。它无处不在：从简单的按钮点击到像 Gmail 和 Netflix 这样的复杂网络应用。但现在，有一个挑战者进入了这个圈子——**PyScript**，这是一个框架，允许你在浏览器中运行**Python**而无需后端。听起来像梦一样，对吧？让我们通过这两项网络技术之间的有趣**面对面对决**来分析一下，看看 PyScript 是否是一个真正的竞争对手！

## 第一轮：它们是什么？

这就像杰克·保罗与迈克·泰森的较量：新挑战者（PyScript）与老牌冠军（JS）。不用担心，我今天并不是说这场战斗会令人失望。

让我们从老将开始：**JavaScript**。

+   创建于 1995 年，JavaScript 是网络开发的**骨架**。

+   在浏览器中本地运行，控制从用户交互到动画的一切。

+   由**React、Vue、Angular**和庞大的框架生态系统支持。

+   可以直接**操作 DOM**，使网页动态化。

现在轮到新手：**PyScript**。

+   基于**Pyodide**（一个 Python 到 WebAssembly 的项目），PyScript 允许你在 HTML 文件中编写**Python**。

+   不需要**后端服务器**——你的 Python 代码直接在浏览器中运行。

+   可以导入 Python 库，如 NumPy、Pandas 和 Matplotlib。

+   **但是**，它仍在发展，并且有局限性。

最后这个*但是*很重要，所以 JavaScript 赢得了第一轮！

## 第二轮：性能对决

当谈到速度时，JavaScript 就像尤塞恩·博尔特——优化且 **飞速**。它原生运行在浏览器中，并针对性能进行了微调。另一方面，PyScript 通过 WebAssembly 运行 Python，这意味着 **额外的开销**。

让我们用一个真正的迷你项目：一个简单的计数器应用程序。我们将使用这两种替代方案来构建它，并看看哪一个表现更好。

### JavaScript

```py
<button onclick="increment()">Click Me</button>
<p id="count">0</p>
<script>
  let count = 0;
  function increment() {
    count++;
    document.getElementById("count").innerText = count;
  }
</script> 
```

### PyScript

```py
<py-script>
from pyscript import display
count = 0

def increment():
    global count
    count += 1
    display(count, target="count")
</py-script>
<button py-click="increment()">Click Me</button>
<p id="count">0</p>
```

将它们付诸实践：

+   JavaScript 运行即时。

+   PyScript 有明显的延迟。

本轮结束：JavaScript 增加了优势，以 2-0 领先！

## 第三轮：易用性与可读性

这两种语言都不是完美的（例如，它们都不包括静态类型），但它们的语法非常不同。**JavaScript** 可能相当混乱：

```py
const numbers = [1, 2, 3];
const doubled = numbers.map(num => num * 2);
```

而 **Python** 则容易得多：

```py
numbers = [1, 2, 3]
doubled = [num * 2 for num in numbers]
```

PyScript 允许我们使用 Python 语法的事实无疑使其成为本轮的赢家。尽管我明显偏向 Python，但它的入门友好性以及通常比 JS 更简洁和简单的事实使其在可用性方面表现更好。

对于 PyScript 来说的问题是，JavaScript 已经深深集成到浏览器中，使其更加实用。尽管如此，PyScript 仍然赢得了这一轮，使得比分变为 2-1。

还有一轮...

## 第四轮：生态系统与库

JavaScript 有无数的框架，如 React、Vue 和 Angular，使其成为构建动态 Web 应用的强大工具。其库专门针对 Web 进行优化，提供从 UI 组件到复杂动画的各种工具。

另一方面，PyScript 得益于 Python 在科学计算和数据科学库方面的庞大生态系统，例如 NumPy、Pandas 和 Matplotlib。虽然这些工具在数据可视化和分析方面非常出色，但它们并不是为前端 Web 开发优化的。此外，PyScript 需要解决方案来与 DOM 交互，而 JavaScript 可以原生且高效地处理。

虽然 PyScript 是将 Python 嵌入 Web 应用程序的激动人心的工具，但它仍处于早期阶段。JavaScript 仍然是通用 Web 开发的更实际选择，而 PyScript 在需要浏览器内 Python 计算能力的场景中表现优异。

下面是一个总结一些关键组件的表格

| 功能 | JavaScript | PyScript |
| --- | --- | --- |
| DOM 控制 | 直接且即时 | 需要 JavaScript 解决方案 |
| 性能 | 优化浏览器 | WebAssembly 开销 |
| 生态系统 | 巨大（React、Vue、Angular） | 有限，仍在增长 |
| 库 | Web 相关（Lodash、D3.js） | Python 相关（NumPy、Pandas） |
| 用例 | 全栈 Web 应用 | 数据密集型应用，交互式小部件 |

本轮结论：JavaScript 在通用 Web 开发中占主导地位，但 PyScript 在以 Python 为中心的项目中表现突出。

## 最终结论

这是一场短暂的战斗！但我们仍然不知道谁赢了...

是时候揭晓了：

+   如果你正在构建全栈 Web 应用，JavaScript 是明显的赢家。

+   如果你正在添加 Python 驱动的交互性（例如，数据可视化），PyScript 可能很有用。

话虽如此，可以说**JavaScript（及其衍生品）仍然是网页前端的最佳选择**。然而，PyScript 的未来值得关注：如果性能得到提升并且与浏览器的集成更加完善，PyScript 可能会成为愿意在前端整合更多数据相关任务的 Python 开发者们的强大混合工具。

**赢家：JavaScript**。

# 从 Power BI 到 Looker 的转换经验（作为一名高级数据分析师）

> 原文：[`towardsdatascience.com/my-experience-switching-from-power-bi-to-looker-as-a-senior-data-analyst-d244cfe62613/`](https://towardsdatascience.com/my-experience-switching-from-power-bi-to-looker-as-a-senior-data-analyst-d244cfe62613/)
> 
> ***⚠️ 注意：** 这篇文章是在比较**Power BI**和**Looker**（不是 Looker Studio）。*

![从 Power BI 过渡到 Looker（来源：自行生产）](img/e3a7dffea82c9f9a6352ce284247557b.png)

从 Power BI 过渡到 Looker（来源：自行生产）

当我开始在我目前的工作中使用 Looker 时，我惊讶地发现关于它的有用信息在网上非常少。

搜索“Looker”，你大多会找到关于 Looker Studio 的内容，这是一个完全不同的工具。

> 我无法找到一篇真实且实用的关于比较 Power BI 和 Looker（不是 Looker Studio）的评论，所以我决定基于我对两者的经验来创建一篇。

在之前在[Zensai](https://zensai.com/)（原名 LMS365）的职位中使用 Power BI 的经验，以及在我目前在[Trustpilot](https://business.trustpilot.com/jobs)的职位中过渡到 Looker，我想记录一些你在未来可能自己进行这种转变时需要了解的重要技术细节。

在这篇文章中，我将：

+   讨论 Power BI 和 Looker 之间的主要区别，

+   讨论每个工具的优点和缺点，

+   分享两个平台之间平稳过渡的技巧，以减少痛苦，

+   突出 Looker 的独特功能，这些是我所喜爱的，

+   并提及我是否更喜欢 Power BI 或 Looker 作为高级数据分析师在商业与营销方面的选择。

* * *

## **Looker 的可视化下集成表格视图可以节省时间**。

> *(Looker **1:0** Power BI)*

我在 Looker 中非常欣赏的一个功能是在探索阶段，可视化之下显示的集成数据表。

![数据表始终位于您的可视化之下（来源：自行生产）](img/662cb81aa944db30bf99ca77b54e2285.png)

数据表始终位于您的可视化之下（来源：自行生产）

我无法计算有多少次在 Power BI 中构建图表、调试和检查列时，不得不在报告视图标签页和数据视图标签页之间来回切换。

> 在 Power BI 中，你可以将可视化显示为表格——你点击三个点并选择“显示为表格”。但是，这里有个问题：表格并不是你所说的完全交互式的。你不能编辑它，旋转或自定义它，就像在 Looker 中那样。它更像是一个快照而不是一个工具。如果你想进行更改，你必须进入 Power Query。你无法在编辑时看到可视化——这是一个断开的过程。

![Power BI 中的表格不像 Looker 那样可定制（旋转、计算新列、度量、维度、隐藏列等）（来源：本人生产）](img/a7757996ae0fb65619959e1d9b396d2d.png)

Power BI 中的表格不像 Looker 那样可定制（旋转、计算新列、度量、维度、隐藏列等）（来源：本人生产）

在 Looker 中始终在可视化下方有一个可定制的表格视图可以节省大量时间，这确实是一个你很快就会习惯的实用功能。

## **Looker 表达式缺乏 DAX 公式的强大功能和功能。**

> *(Looker **1:1** Power BI)*

然而，我不喜欢 Looker 表达式。

与 [DAX（数据分析表达式）](https://learn.microsoft.com/en-us/dax/) 相比，它们在函数数量方面的发展程度并不相同。

DAX 提供了一个丰富的函数库，用于复杂计算和时间智能，这对于高级分析至关重要。

在 Power BI 中，我最喜欢的 DAX 函数无疑是 [=CALCULATE 函数](https://learn.microsoft.com/en-us/dax/calculate-function-dax)。`CALCULATE` 函数通过更改筛选条件来提供你想要的结果——这使得它在基于时间或特定类别的计算等方面非常实用：

```py
Previous_Month_Gross_Volume = 
CALCULATE(
    [gross_volume], 
    DATEADD(dim_date[Date], -1, MONTH),
    dim_product[Category] = "Electronics",
    dim_region[Region] = "North America"
)
```

虽然 Looker 提供了一系列用于数据操作的表达式，它们称之为 [Looker 表达式](https://cloud.google.com/looker/docs/functions-and-operators)（有时也称为 Lexp），但它们的深度和多功能性并不及 Power BI 中的 DAX。

> 与 DAX 相比，Looker 的表达式被更少地采用，在在线论坛、文档和教育资源中的存在感也较小。这也归因于 DAX 还被用于 Analysis Services 和 Excel 中的 Power Pivot。

我还注意到，生成式 AI 模型有时会难以处理 Looker 表达式的确切结构/公式，因为它们是在 DAX 之后引入的——即使为模型设置了正确的上下文和额外的参数。

> 需要记住的另一件事是，Looker 表达式与 [LookML 语言](https://cloud.google.com/looker/docs/write-lookml-intro) 不相同——这可能是某些人产生混淆的地方。
> 
> Looker 表达式用于执行计算，并且它们使用你在 LookML 中定义的字段。

例如，当你将一个字段添加到表达式中时，**Looker 使用字段的 LookML 标识符**，其形式类似于 `${view_name.field_name}`。

![在 Looker 中创建一个新的计算列（来源：本人生产）](img/c2261f2d853039483d678b99055f51af.png)

在 Looker 中创建一个新的计算列（来源：本人生产）

Looker 表达式用于创建：

+   自定义维度

+   自定义计算列

+   自定义度量

+   以及自定义筛选

…**在** Looker 探索界面中。

![通过仅使用数据表中存在的字段创建计算列 - 之后您可以隐藏它们，如果您不想在图表上显示它们（来源：自产）](img/64dc539ca3f090e7d5c77eaa4e7b8bbf.png)

您可以通过仅使用数据表中存在的字段来创建计算列 - 之后您可以隐藏它们，如果您不想在图表上显示它们（来源：自产）

这看起来很简单 - 但如果您结合了***多个*** **维度**、***度量值***、***计算*** **列**和***数据透视***，您最终可以得到一个非常强大的数据表。

![使用多个维度、度量值和计算列创建队列时的交叉数据表（来源：自产）](img/5558a79a72aaa5923d7d874c1d252cba.png)

使用多个维度、度量值和计算列创建队列时的交叉数据表（来源：自产）

## **Power BI 中的 Power Query 编辑器使数据操作变得简单易行。**

> *(Looker **1:2** Power BI)*

Power BI 有 Power Query 编辑器，类似于 Excel 中的编辑器。

![在 Power BI 中启动 Power Query 编辑器（来源：自产）](img/8a59487a0f46d55ac78ad76917a3e5ca.png)

在 Power BI 中启动 Power Query 编辑器（来源：自产）

您可以通过几步点击创建计算列、删除首行、将首行设置为标题，以及更多操作***只需几步点击***。

![Power BI 中的 Power Query 编辑器（来源：自产）](img/e264437179fc20f6e0198326ccc59c20.png)

Power BI 中的 Power Query 编辑器（来源：自产）

这个 Power Query 编辑器在 Looker 中不存在 - 尽管您可以在 Looker 的 Explore 中非常容易地创建自定义度量值、自定义维度和自定义计算列。

然而，对于在 Power BI/Excel 中非常有用的其他转换，您需要编写 LookML 代码。

正如您将在本文后面发现的那样，这也是 Looker 由于数据集中化而具有的一个重大优点。

## **Looker 仅基于网页。**

> *(Looker **1:3** Power BI)*

我喜欢 Power BI 是桌面应用程序，这让我可以离线使用***本地*** ***数据集***，并利用我本地机器的处理能力！

![Power BI 是桌面应用程序，Looker 不是（来源：自产）](img/e4a3000e5a5436e4d21aad182c92f2ca.png)

Power BI 是桌面应用程序，Looker 不是（来源：自产）

Looker 不是桌面应用程序 - 它仅基于网页。如果您更喜欢 Mac，基于网页是一个优势 - 尤其考虑到 Power BI 的桌面应用程序仅适用于 Windows。

> 我必须说，在 Looker 中我喜欢的一点是，您可以使用浏览器上的前后按钮非常容易地导航您的更改。这是因为 Looker explore 中的每个更改都会创建一个新的唯一 URL 链接。当您修改查询或调整设置时，URL 会更新以反映当前的状态。这与 Looker 的缓存功能相辅相成，允许您再次访问查询和可视化，而无需再次运行它们，因此它们可以立即加载（默认缓存保留策略为[一小时](https://cloud.google.com/looker/docs/caching-and-datagroups#:~:text=Warning%3A%20For%20queries%20run%20on,retention%20policy%20is%20one%20hour.)）。

虽然***Power BI 服务*** – 用户在此处上传和编辑报告、协作和创建应用程序，也是基于 Web 的，但 Looker 完全基于浏览器的方法完全消除了操作系统限制。

## **Power BI 简化了星型模式建模，而 Looker 则需要学习 LookML**。

> *(Looker **1:4** Power BI)*

在 Power BI 的**建模选项卡**中建模和创建您的星型模式非常简单。

![在 Power BI 中使用建模选项卡创建星型模式（来源：自行制作）](img/d8f0d1b940ba4359b624ddc37e91b120.png)

在 Power BI 中使用建模选项卡创建星型模式（来源：自行制作）

您可以享受一个非常大的画布，其中包含所有导入的表，并可以使用外键（FK）和主键（PK）通过事实表和维度表之间的一对多关系链接您的表。

> 例如，来到您业务的潜在客户可以存储在事实表中，而地区、联系表单、潜在客户来源和渠道则是具有唯一值的单独维度表。

这有助于您非常快且高效地查询事实表。此外，使用 Power BI 的**点击选择**功能调整关系非常简单。

![通过双击关系线，您可以通过几步点击调整您的关系（来源：自行制作）](img/171c9a46d3b2dff6bc413040a7164433.png)

通过双击关系线，您可以通过几步点击调整您的关系（来源：自行制作）

> 在 Looker 中，这不像点击并使用几步点击建立关系那样简单。

您需要学习 LookML 来在 explores 中连接您不同的视图（= 视图反映了从您的数据仓库加载到 Looker 的表）并使用 LookML 语言定义您的关系和键。

![在 Looker 中使用 LookML 代码创建星型模式（来源：自行制作）](img/2605643f6cc80193f46d230ed3b00af2.png)

在 Looker 中使用 LookML 代码创建星型模式（来源：自行制作）

## Power BI 在图表选项方面优于 Looker。

> *(Looker **1:5** Power BI)*

使用 Power BI，您有更多默认图表可供选择。

![Power BI 中的智能图表，包括三指数平滑预测（Holt-Winters）和关键影响因素图（来源：自行制作）](img/c0890729395aee49af3b32d217e7783a.png)

Power BI 中的智能图表，包括三指数平滑预测（Holt-Winters）和关键影响因素图表（来源：自产内容）

我确实很怀念那个基本为您在单个视觉中构建简单回归模型的**关键影响因素图表**！

![使用 Power BI 中的“顶级细分”可视化进行简单的细分（来源：自产内容）](img/8d36939d821c895d8689485f51d3fa3f.png)

使用 Power BI 中的“顶级细分”可视化进行简单的细分（来源：自产内容）

虽然如此，好在 Looker 也有图表的市场，尽管许多图表**并非官方支持的 Google 产品**。

## **Looker 基本上是一个庞大且透明的 SQL 运行机**。

> *(Looker **2:5** Power BI)*

我很喜欢 Looker 总是运行并轻松显示创建您在可视化下看到的表的 BQ SQL 查询，然后使用它来创建您的可视化（尽管如果处理不当，运行这些查询可能会变得非常昂贵）。

![SQL 选项卡显示您的查询直接在视觉下方运行在您的数据仓库中（来源：自产内容）](img/010049751864cd5cd12fa0700dde8098.png)

SQL 选项卡显示您的查询直接在视觉下方运行在您的数据仓库中（来源：自产内容）

> 您可以非常容易地在 Looker 中点击 SQL 选项卡，并查看针对您的数据仓库运行的精确查询，这对于调试和查看用于构建可视化的数据仓库中的哪些表非常有帮助。
> 
> 我调试的小贴士是复制您视觉的 SQL 查询，并使用生成式 AI（如 Claude 或 ChatGPT）来询问潜在的问题。

在 Power BI 中，访问底层查询需要额外的步骤，例如使用 ***性能分析器*** 或像 ***DAX Studio*** 这样的外部工具。

## **在 Looker 中格式化图表有点繁琐**。

> *(Looker **2:6** Power BI)*

对于图表的格式化选项，与 Power BI 相比并不那么强大。

> 微软在 Power BI 中创建图表的定制方面一直在提升，提供了许多不同的 **点击并选择** 下拉菜单，您在构建图表时为图表设置，几乎可以针对您能想到的任何内容。

![在 Power BI 中编辑任何图表都非常简单，只需点击并选择下拉菜单即可（来源：自产内容）](img/8179408b445bf93e44612892d6244ff1.png)

在 Power BI 中编辑任何图表都非常简单，只需点击并选择下拉菜单即可（来源：自产内容）

我并不是说在 Power BI 中能做的事情，在 Looker 中做不到——而是在 Power BI 中，条件格式化、目标、目标设定、坐标轴、字体、标题以及任何视觉的所有方面都可以通过简单的下拉菜单进行调整。

你还可以调整 Power BI 可视化中***特定指标***的外观，而无需改变整个图表或表格的显示效果 – 这允许你仅对关注的指标进行精确编辑。

![在 Power BI 中，条件格式化、目标和目标设置可以通过下拉菜单简单编辑（来源：自产）](img/4d6325b24aa69b770522d9036d929d18.png)

在 Power BI 中，条件格式化、目标和目标设置可以通过下拉菜单简单编辑（来源：自产）

![在 Power BI 中，条件格式化、平滑线条、所有类型的网格线（如虚线、点或实线）也可以轻松编辑（来源：自产）](img/e32f714d5774b61a198f9a6654dfae59.png)

在 Power BI 中，条件格式化、平滑线条、所有类型的网格线（如虚线、点或实线）也可以轻松编辑（来源：自产）

对于 Looker，我不得不写下 JSON 代码片段`{"backgroundColor": "#code"}`来更改图表的背景颜色，这随后也导致了在 Looker 中显示的此通知：

> *"⚠️ 图表配置编辑器中已进行更改。编辑可视化设置可能会导致意外行为。"*

![在 Looker 中使用 json 代码片段调整图表背景时出现错误（来源：自产）](img/771fa6459e902f97d5eb6df09fb4f556.png)

在 Looker 中使用 json 代码片段调整图表背景时出现错误（来源：自产）

## **Looker 的 git 版本控制是内置的，易于使用。**

> *(Looker **3:6** Power BI)*

我非常喜欢 Looker 内置的版本控制功能，因为它非常直观且易于使用 – 你不需要离开 Looker。

如果你决定建模、编辑或创建新的维度和度量标准，你首先需要开启***开发模式*** – 这将允许你设置自己的开发区域，这样你不会影响到生产中的任何内容。

然后，你可以在 Looker 中直接创建自己的个人（或共享）分支，并在此处编写 LookML 代码。

![Git 版本控制在 Looker 中是内置的，易于使用（来源：自产）](img/43ad2d6833ca017dd0857697f313c5fe.png)

Git 版本控制在 Looker 中是内置的，易于使用（来源：自产）

之后，你可以简单地发布你的分支到远程仓库（GitHub），然后创建 PR 以合并你的新分支到主分支。

## **使用 LookML 进行数据建模确保了组织内度量标准和维度的统一。**

> *(Looker **4:6** Power BI)*

Looker 非常出色，我喜欢它使用专有的 LookML 语言进行集中式数据建模的方法。

> LookML 代码允许您在组织内保持定义的一致性。在实践中这意味着，因为我们所有人都是作为数据分析师/科学家用一种语言编写一切，我们最终都会使用由我们的团队定义和批准的相同度量标准和维度。简单来说，Looker 让您能够在数据仓库中的预建模数据之上构建内容。

![在您的视图中（=表）中定义维度和度量，使用 LookML 确保在不同模型中只有一个定义（来源：自产）](img/9cdcce28aae7868711df8778359ea8b5.png)

在您的视图中（=表）中定义维度和度量，使用 LookML 确保在不同模型中只有一个定义（来源：自产）

您的平均员工无法定义内部的数据模型，但将能够自由地使用这些数据。这旨在创建组织内每个人都想要的单一事实来源。

> 技术上，当您更改业务中指标的度量定义时，您应该能够在一个地方编辑您的 LookML，并且使用该指标的所有探索、查看和仪表板都将自动更新。

我还喜欢 Looker，因为它允许您在创建视图时使用 LookML 中的`sql:`参数进行 SQL 编辑（视图反映了从您的数据仓库加载到 Looker 的表）。

## **Looker 的工作流程促进在构建仪表板之前进行数据探索。**

> *(Looker **5:6** Power BI)*

虽然这种比较不是 1:1 的，Power BI 的开发流程阶段：

+   **报告 → 仪表板 → 应用**

…将与 Looker 相似：

+   **探索 → Look → 仪表板。**

我发现 Looker 的方法更直观，因为它允许在构建仪表板之前更好地探索数据 ***之前***：

1.  在 Looker 中，您从***探索***开始，这允许使用即席查询进行交互式 EDA。

![Looker 中的探索示例 – 在“探索”部分，您可以决定是否将您的视图保存为 Look 或直接保存到仪表板中（来源：自产）](img/e35b97f6268dab40a97451f32632c47c.png)

Looker 中的探索示例 – 在“探索”部分，您可以决定是否将您的视图保存为 Look 或直接保存到仪表板中（来源：自产）

1.  然后，您可以将使用这些探索构建的图表保存为***Look*** – 这些****是可重用的，并且可以将它们附加到任何仪表板。

![Looker 中的 Look 示例 – Look 是任何仪表板的基本构建块（来源：自产）](img/0acd33715ce3360222e6f05ef2275fd2.png)

Looker 中的 Look 示例 – Look 是任何仪表板的基本构建块（来源：自产）

1.  这些包括图表的 Look 可以之后拼接到***仪表板***中，以全面了解您的关键指标。

![Looker 中的简单仪表板示例（来源：自产）](img/0a168078b42547366519c3c41828ab75.png)

Looker 中的简单仪表板示例（来源：自产）

> 我喜欢 Looker 中的 Explorers，因为它们赋予了组织中的任何人深入数据的能力，而无需了解 SQL。当你作为一个 Looker 开发者准备 Explorers 以供使用时，它使得财务专家、市场营销经理、收入总监 - 实际上任何人 - 都可以与该 Explorers 中的准备视图进行交互。他们不需要技术专业知识或 SQL 知识 - 他们可以提出自己的问题，并通过旋转、切片和过滤来获得即时答案。

在仪表板投入生产之前存在于 Looker 中的这一层，与 Power BI 相比，无疑是 Looker 的一个重大优势。

在 Power BI 中，虽然用户可以与报告交互并应用过滤器，但创建新的分析通常需要构建新的报告或拥有更高级的技能。

当使用 Looker 时，任何人的即席探索选项是组织中的一个非常有用的工具。

## Power BI 可以免费使用 - Looker 则不行。

> *(Looker **5:7** Power BI)*

尽管有一个名为 ***Looker Studio*** 的免费工具，但它与功能齐全的 ***Looker 企业 BI 平台*** 有显著差异 - 这使得学习 Looker 非常具有挑战性。

然而，我发现对于获得 Looker 的实际操作经验来说，[互动式 Google Cloud 实验室](https://www.cloudskillsboost.google/paths/28)（这些实验室允许你在模拟、时间有限的会话中尝试 Looker - 此外，完成这些实验室还可以获得 Google 徽章！）非常有帮助。

> Power BI Desktop 可以作为独立的桌面应用程序免费下载，并且其付费的企业版本与 Looker 相比更具成本效益，因为随着分析团队的增长，成本可能会螺旋式上升。

## Gartner 表示 Power BI 是当前的选择。

> *(Looker **5:8** Power BI)*

根据 Gartner 的 2024 年 [分析和商业智能平台魔力象限](https://www.gartner.com/doc/reprints?id=1-2HVUGEM6&ct=240620&st=sb)，微软的 Power BI 在其与微软生态系统的集成、功能性和性价比方面脱颖而出。

![分析和商业智能平台魔力象限将微软评为领导者（来源：自产；评分来自 Gartner, Inc 2024；链接这里）](img/42a95c916bf56700df14a488186643f9.png)

分析和商业智能平台魔力象限将微软评为领导者（来源：自产；评分来自 Gartner, Inc 2024；链接[这里](https://www.gartner.com/doc/reprints?id=1-2HVUGEM6&ct=240620&st=sb)）

## 作为一名高级数据分析师，我的偏好。

我已经是一名重度 Power BI 用户 4-5 年了，我认为它是一个出色的工具。它易于使用，功能丰富，非常适合想要立即开始构建的人。**在功能方面，我更喜欢 Power BI**。

但现在我已经转向 Looker 并花时间了解它是如何工作的，**我开始看到 Looker 有可能成为我未来将使用的 #1 BI 工具**。

为什么我会说即使我在功能上给 Power BI 评分更高呢？***这全部取决于公司的背景（正如 Gartner 研究也说的那样）。***

如果更多公司开始采用 BQ 并其云市场份额增长，它会产生一种锁定效应。即使你换工作，你也可能会再次遇到 Looker。所以你的 #1 BI 工具将由你决定（当然，也可以反过来适用于 Azure 和 PowerBI）。

在此之前，我曾在 [Zensai](https://zensai.com/)（原名 LMS365）工作，[那里](https://cloud.google.com/customers/trustpilot) 我们严重依赖微软生态系统——甚至我们的产品都集成到了微软中。但现在在 [Trustpilot](https://business.trustpilot.com/jobs) 上，Looker 更有意义，因为我们使用的是 Google Cloud Platform（GCP）——您可以在这里了解更多关于我们如何使用 GCP 的信息。

> 如果您的公司已经使用 GCP 和 BigQuery，Looker 的集成非常完美。它简化了所有操作——从探索数据、创建图表到调试问题。它并不完美（我可以不需要缺少定制选项、随机错误、基于网页的缓慢以及几乎需要为所有事情编写 LookML 代码的需求），但在 Google Cloud 环境中，我认为 Looker 是一个很好的选择，因为它变成了一种非常简单的工具，用于创建图表、在构建仪表板之前探索您的数据，以及快速调试——这些是我最看重的任何可视化工具的品质。

话虽如此，Power BI 和 Looker 之间的选择实际上取决于您的公司使用什么作为其 1) ***数据仓库***，2) ***人力资源***，3) ***前员工经验***，以及您的 4) ***财务资源***。

* * *

**PS:** 这里有一个带有清单格式的决策框架表格，以指导您在 ***Power BI*** 和 ***Looker*** 之间进行选择。

![检查您是否应该在您的组织中获取 Power BI 或 Looker（来源：自产内容）](img/5e0c0ef25e568c72dc3a521da6f5a1e9.png)

检查您是否应该在您的组织中获取 Power BI 或 Looker（来源：自产内容）

* * *

感谢您阅读。如果您喜欢这篇评论或学到了新东西，请 ***鼓掌*** 并关注我以获取更多内容。随时 ***联系*** 并在 [LinkedIn](https://www.linkedin.com/in/tomas-jancovic/) 上与我联系。

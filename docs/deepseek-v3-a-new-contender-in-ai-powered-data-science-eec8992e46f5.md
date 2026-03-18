# DeepSeek V3：AI 驱动数据科学的新竞争者

> 原文：[`towardsdatascience.com/deepseek-v3-a-new-contender-in-ai-powered-data-science-eec8992e46f5/`](https://towardsdatascience.com/deepseek-v3-a-new-contender-in-ai-powered-data-science-eec8992e46f5/)

1 月 27 日星期一，中国初创公司 DeepSeek 发布其新 AI 模型后，英伟达股价下跌超过 15%。该模型[性能与 ChatGPT、Llama 和 Claude 相当](https://www.deepseek.com/)，但成本却低得多。[据《Wired》报道](https://www.wired.com/story/openai-ceo-sam-altman-the-age-of-giant-ai-models-is-already-over/)，OpenAI 花费超过 1 亿美元来训练 GPT-4。但 DeepSeek 的 V3 模型仅花费了 560 万美元。这种成本效益也反映在 API 成本上——对于每 1000 万个令牌，deepseek-chat 模型（V3）的成本为 0.14 美元，而 deepseek-reasoner 模型（R1）的成本仅为 0.55 美元（[DeepSeek API 定价](https://api-docs.deepseek.com/quick_start/pricing/)）。同时，gpt-4o API 的成本为每 1000 万个输入令牌 2.50 美元，o1 API 的成本为每 1000 万个输入令牌 15.00 美元（[OpenAI API 定价](https://openai.com/api/pricing/)）。

始终对新兴的大型语言模型（LLMs）及其在数据科学中的应用感到好奇，我决定对 DeepSeek 进行测试。**我的目标是看看其聊天机器人（V3）模型在协助或甚至取代数据科学家日常任务方面表现如何**。我使用了之前文章系列中相同的标准，评估了 ChatGPT-4o、Claude 3.5 Sonnet 和 Gemini Advanced 在[SQL 查询](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-1-821086810318)、[数据探索分析（EDA）](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-2-whos-the-best-at-eda-6ed5a4a6f008)和[机器学习（ML）](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-3-best-ai-assistant-for-machine-learning-a2078793e4fa)方面的性能。

![Image created by DALL·E](img/ed987e3260382606d8cefd0d063a8c03.png)

Image created by DALL·E

* * *

## I. 初次印象

在我对 DeepSeek 网络聊天机器人 UI 的初步探索中，以下是一些快速观察：

+   **界面**：聊天机器人 UI 看起来与 ChatGPT 相似，所有过去的聊天都列在左侧，当前对话在主面板中。

+   **聊天标签**：ChatGPT 通常用简短的摘要标记您过去的聊天。然而，默认情况下，DeepSeek 使用您提示的前几个词来标记旧聊天。对于同一网络会话中的新聊天，它只显示“新聊天”，当有多个时可能会造成混淆。但当然，您可以为任何聊天重命名以增加清晰度。

+   **模型选项**：在消息输入框中，您可以选择使用推理模型（R1）或启用网络搜索功能。

+   **格式化**：聊天机器人将关键词和代码片段整齐地格式化，便于阅读。

![DeepSeek UI (image by author)](img/0c85e1209c08ff8b5230547df729fc72.png)

DeepSeek UI (image by author)

* * *

## II. SQL

### 1. 问题解决（3/3）

我首先通过测试 DeepSeek 解决三个具有挑战性的 LeetCode SQL 问题（[262](https://leetcode.com/problems/trips-and-users/description/)、[185](https://leetcode.com/problems/department-top-three-salaries/description/)和[1341](https://leetcode.com/problems/movie-rating/description/)）来测试 DeepSeek 的问题解决技能，这些问题具有低接受率。这些问题有清晰的描述，包括输入和输出表结构，并且类似于面试问题。

DeepSeek 使用聚合、过滤器、窗口函数等解决了所有三个问题。它还提供了逐步分解和清晰的解释。

![](img/e22847832501e2a282e6f3422ae55d73.png)![](img/b3c8be4814863597a5656df3e1ec7307.png)![DeepSeek 对 LeetCode SQL 问题的响应（图片由作者提供）](img/d66d7be96fab41bdc2f3042dd21e1412.png)

DeepSeek 对 LeetCode SQL 问题的响应（图片由作者提供）

### 2. 业务逻辑（3.5/4）

接下来，我上传了四个合成数据集来模拟现实世界场景，在这些场景中，表描述通常不完整，你必须根据数据的外观来假设信息。

尽管四个 CSV 文件的总大小仅为约 300KB，但我收到了错误消息“*DeepSeek 只能读取所有文件的 40%。请尝试用较小的摘录替换附加的文件。*”所以我将每个文件的大小裁剪到只有 30KB，即截取每个文件的前 100 行 - 我不再收到上述错误消息，但这次它说“*哎呀！DeepSeek 目前正经历高流量。请稍后再查看。*”最终，我转向上传每个数据集顶部行的截图，这成功了。

![DeepSeek 读取表格截图并编写 SQL 查询（图片由作者提供）](img/dab10f3ea61db1cf43cce4f7ea40353f.png)

DeepSeek 读取表格截图并编写 SQL 查询（图片由作者提供）

数据集包括：

1.  **users**：包含人口统计信息的用户级别数据。

1.  **products**：产品级别数据。

1.  **orders**：包含支付信息的订单级别数据。

1.  **ordered_products**：连接订单和产品的表。

我要求 DeepSeek 编写查询，以获取诸如“美国用户按月总订单金额”、“每月新用户数量”、“前 5 个最佳销售产品类别”和“每月用户留存率”等指标。这些是在电子商务公司中通常会跟踪的常见指标。它能够为前三个指标生成正确的查询，但在留存率问题上出现了错误（这也是其他 AI 工具最难以解决的问题之一）。然而，在提示问题后，它能够修复查询。

![](img/c90b102636038e81d813e36be9d20fa5.png)![DeepSeek 通过提示修复了留存率查询（图片由作者提供）](img/ba0cc22e00a0d32f86bfd5910cb9b6b1.png)

DeepSeek 通过提示修复了留存率查询（图片由作者提供）

### 3. 查询优化（2.5/3）

最后，我测试了 DeepSeek 优化次优 SQL 查询的能力。我使用了从我[SQL 优化文章](https://towardsdatascience.com/mastering-sql-optimization-from-functional-to-efficient-queries-74d8692f10be)中选取的低效代码示例。它通过仅选择必要的列、将聚合步骤提前、避免冗余去重操作等方式改进了查询。

我最喜欢的是，它不仅提出了改进建议，还提供了关于所有可优化内容的详细解释，包括针对特定数据库的技巧。

![图片](img/bc278cc6e6e07f4c81aed1f72082358d.png)![DeepSeek 的 SQL 查询优化性能 1（作者提供）](img/6a97156f06d877306f51ebf0dcb24c3b.png)

DeepSeek 的 SQL 查询优化性能 1（作者提供）

![图片](img/52c423238bb9e5b7ea36c87c31370f00.png)![DeepSeek 的 SQL 查询优化性能 2（作者提供）](img/df6c8ecbfce888ef82b25eb098ddec01.png)

DeepSeek 的 SQL 查询优化性能 2（作者提供）

我只遇到了一个问题，那就是我在最后一个问题中要求它通过调整窗口函数进一步优化查询，但它生成了一个产生重复行的查询。在我指出这个问题后，它迅速进行了纠正。

![图片](img/a87f85db8554d7e1c5228a4483f50a2a.png)![图片](img/5e5c41ece88c08694468bffd694cb187.png)![DeepSeek 纠正了它造成的问题（作者提供）](img/7a6856804c7be41afa2773bae1c76366.png)

DeepSeek 纠正了它造成的问题（作者提供）

### SQL 性能摘要

总体而言，DeepSeek 在 SQL 部分表现良好，提供了对 SQL 查询的清晰解释和建议。它只犯了两处小错误，并在我的提示下迅速进行了纠正。然而，其文件上传限制可能会给用户带来不便。

![SQL 性能评估（作者提供）](img/c43b8cd91b540c80f6071e932ef1702c.png)

SQL 性能评估（作者提供）

* * *

## III. 数据探索分析（EDA）

现在让我们转换一下话题，来探讨数据探索分析（Exploratory Data Analysis）。由于 DeepSeek 的文件上传限制，我仅能上传我 Medium 文章性能的一个非常小的数据集（2KB）。如果您感兴趣，可以在我的[过往文章](https://towardsdatascience.com/my-medium-journey-as-a-data-scientist-6-months-18-articles-and-3-000-followers-c449306e45f7)中找到对 Medium 旅程的详细分析和回顾。

这里是我的提示：

> 我一直在 Medium 上撰写文章，并收集了我文章性能的此数据集。您是一位数据科学专业人士。今天的任务是帮助我对此数据集进行彻底的数据探索分析（EDA），包括必要的步骤，如数据清洗、分析、可视化、清晰的见解和可操作的建议。
> 
> 您的 EDA 将有助于更好地理解 Medium 的收益并指导未来的写作策略。

以下是用于评估 AI 工具 EDA 能力的评分标准。

![EDA 评估标准（图片由作者提供）](img/73e5a79ce2b13dc777d28d3823798ca7.png)

EDA 评估标准（图片由作者提供）

### 1. 完整性（4/5）

DeepSeek 的 EDA 响应非常有序，涵盖了 EDA 的大部分关键组件。

**数据检查**：您可以点击上传的数据集以获取预览。然而，预览是基于文本的，这使得消化起来很困难。它也没有提供任何关于数据集的文本描述。因此，我认为这一步骤是不完整的。

![检查 DeepSeek 中的数据集（图片由作者提供）](img/6a53c3b98ca803871502f2efdddc143e.png)

检查 DeepSeek 中的数据集（图片由作者提供）

**数据清洗**：DeepSeek 的报告从数据清洗开始。它检查了缺失值、数据类型，并删除了不必要的列。尽管它没有显示结果，但在每个步骤都提供了总结和说明。

![DeepSeek 中的数据清洗步骤（图片由作者提供）](img/193d8dff5333e36262f7cfc30121e734.png)

DeepSeek 中的数据清洗步骤（图片由作者提供）

**单变量分析**：DeepSeek 使用 Python 代码检查了收入和其他列的分布，以运行分析和生成可视化。它不在 UI 中绘制图表，所以我手动在我的 Jupyter 笔记本中运行它们以进行验证。

![](img/a86a6e119dcc55925a296237af7fbd1a.png)![DeepSeek 中的单变量分析（图片由作者提供）](img/e25f280470430369a4f0bcc02ba8c773.png)

DeepSeek 中的单变量分析（图片由作者提供）

**双变量和多变量分析**：DeepSeek 探讨了收入与其他许多变量之间的关系，以了解 Medium 收入的驱动因素。

![](img/9471549309e72de8fafa6b709b18d783.png)![DeepSeek 中的双变量分析（图片由作者提供）](img/e74b8a28b1f9ba039c2af8b314f0efc6.png)

DeepSeek 中的双变量分析（图片由作者提供）

**分析和推荐**：DeepSeek 还根据其分析提供了可操作的见解。

![DeepSeek 的分析和推荐（图片由作者提供）](img/f9c284b6c126068b9f42351ddfb3588f.png)

DeepSeek 的分析和推荐（图片由作者提供）

### 2. 准确性（3/4）

我审查了 DeepSeek 生成的 Python 脚本并手动运行它。虽然大部分代码运行良好，但相关矩阵部分由于包含非数值列而抛出了错误。我报告了错误信息，并通过添加`df.select_dtypes(include=[np.number])`来仅过滤数值列以纠正问题。

这个小错误导致扣了一分。

![相关矩阵代码错误（图片由作者提供）](img/1014d7ad9707189ff48a5ab2d3a3cbb7.png)

相关矩阵代码错误（图片由作者提供）

### 3. 可视化（2/4）

DeepSeek 在其 UI 中没有显示可视化，只显示 Python 代码。虽然生成的代码准确无误（除了上面提到的相关矩阵错误），但与 ChatGPT 等其他工具相比，整体用户体验不太友好。因此，我为此限制减去了两分。

### 4. 洞察力（4/4）

DeepSeek 基于其分析提供了有价值的见解和可操作的推荐。它涵盖了内容策略、出版物选择、“增强”的力量等。

### 5. 可重复性和文档（3/3）

DeepSeek 将其 EDA 报告结构化，从数据清洗到分析，再到见解。段落也格式良好，包含项目符号、代码块和高亮关键词。

### EDA 性能总结

DeepSeek 提供了一份逻辑结构化的 EDA 报告，其中包含功能代码和清晰的见解。然而，它无法在 UI 中显示可视化是一个显著的缺点——这为用户增加了额外的步骤，需要在本地运行代码并手动调整图表。

![EDA 性能评估（图片由作者提供）](img/8635b2f6907306cfd44006e1d94510ce.png)

EDA 性能评估（图片由作者提供）

## III. 机器学习（ML）

我使用相同的数据集来评估 DeepSeek 如何协助机器学习项目。以下是我的评分标准。

![机器学习评估评分标准（图片由作者提供）](img/cc49c76402107e400eb3855284943fcd.png)

机器学习评估评分标准（图片由作者提供）

### 1. 特征工程（3/3）

我首先要求它使用以下提示进行特征工程：

> 我一直在 Medium 上撰写文章，并收集了我文章性能的数据集。你是一位数据科学家专业人士。我希望你能帮助我构建一个机器学习模型来预测文章收入，并了解如何提高收入。
> 
> 让我们一步一步来完成这个任务。首先，请专注于特征工程。
> 
> 你能建议一些特征工程技术，这些技术可以帮助提高我的模型性能吗？
> 
> 请考虑转换、特征之间的交互以及可能相关的任何特定领域特征。
> 
> 对每个建议的特征或转换提供简要的解释。

DeepSeek 建议了 10 种特征工程技术。大多数方法都很合理，例如，对右偏变量应用对数转换、计算每观看一次的参与度比率、添加时间特征等。

![](img/8d42b4949e62462c4fdda045fb7ff4ac.png)![](img/aa64327b0f27391173aacf2c4e594223.png)![DeepSeek 的特征工程（图片由作者提供）](img/f7a74a2165557ddf4ec120f3d29645f4.png)

DeepSeek 的特征工程（图片由作者提供）

### 2. 模型选择（3/3）

接下来，我要求深度搜索（DeepSeek）推荐最合适的模型：“*你能推荐这个任务最合适的机器学习模型吗？对于每个推荐的模型，简要说明为什么它适合，并提及使用它时的重要考虑因素*”。

深度搜索（DeepSeek）列出了八个模型选项，从线性回归及其变体，到随机森林和其他基于树的模型，再到神经网络。它为每个模型提供了清晰的优缺点，并以总结和可执行的下一步行动结束。

![](img/daa6b6a614c0000a48daf04c71535b17.png)![](img/6c358ca05cbfe5164de53f1e0aa17974.png)![深度搜索（DeepSeek）的模型推荐（图片由作者提供）](img/c0c99211d8b7b539086080f107e7bd32.png)

深度搜索（DeepSeek）的模型推荐（图片由作者提供）

### 3. 模型训练和评估（3.5/4）

最后，让我们看看它在模型训练和评估方面的能力。我的提示是

> 你能提供训练岭回归模型的代码吗？请确保它包括将数据分为训练集和测试集以及执行交叉验证的步骤。请还建议适当的评估指标和潜在的超参数调整机会。

深度搜索（DeepSeek）的代码结构清晰，并带有注释。它运行良好，输出了回归系数。它还提供了合理的策略来选择合适的评估指标和超参数调整。我随后提出了关于如何解释岭回归模型系数的问题，它也能解释解释方法和多重共线性带来的挑战。

然而，它只提供了基本的代码，而没有在其之前的同一线程中融入任何其特征工程想法。当我要求它添加这些功能时，它不断出现错误信息“服务器忙碌。请稍后再试。”我最终在第四次尝试后得到了输出。我之前也注意到了这一点——深度搜索（DeepSeek）服务器似乎不太可靠，经常出错，尤其是在较长的对话中。因此，我因服务器可靠性问题减去了 0.5 分。

![](img/891dec7ac76bb74b894fb50fcc7d8490.png)![](img/6365176226424e00b39ada9a24ce3ae0.png)![深度搜索（DeepSeek）的模型训练和评估步骤（图片由作者提供）](img/c116f1bf019007e499d4ebbd3d781154.png)

深度搜索（DeepSeek）的模型训练和评估步骤（图片由作者提供）

### 机器学习性能摘要

对于机器学习用例，与其它 AI 工具类似，深度搜索（DeepSeek）在建议特征工程想法、头脑风暴模型和编写代码方面表现出色。然而，它需要人类专业知识来提供指导、提出后续问题并做出最终决定。

![机器学习性能（图片由作者提供）](img/58010300b87e81ed1307771b09ead0c9.png)

机器学习性能（图片由作者提供）

* * *

## 摘要和最终思考

当谈到数据科学项目时，DeepSeek v3 的性能与 ChatGPT-4o、Claude 3.5 Sonnet 和 Gemini Advanced 相当，考虑到其远低的训练成本，这一点尤其令人印象深刻。

![性能比较（图片由作者提供）](img/17d245735caa67aca6b8d9ee25955a20.png)

性能比较（图片由作者提供）

+   **DeepSeek 在编码方面表现优异，但缺少执行 Python 代码和在 UI 中直接显示可视化的关键功能**。这与我去年 8 月对 Claude 3.5 Sonnet 的观察非常相似，当时它也缺少交互式可视化功能。然而，Claude 自那时起已经增加了[分析工具功能](https://www.anthropic.com/news/analysis-tool)（尽管是通过 JavaScript 和 React，而不是 Python），克服了之前的缺点。DeepSeek 可能会跟随这一趋势并添加该功能。

+   **它的服务器似乎比其他工具不太可靠**——感觉就像 ChatGPT 初次推出时的早期版本。上传文件的限制可能会对在数据科学工作流程中有意义地使用聊天机器人构成挑战。

+   然而，其**免费聊天机器人访问和更低的 API 成本**给它带来了显著的竞争优势，尤其是在中国用户和全球小型企业中。它可能使高级 AI 工具的访问民主化，使小型公司和独立开发者能够以更低的成本利用强大的模型。

+   **DeepSeek 的崛起无疑将激励全球更多的 AI 创新**，无论是来自 OpenAI 和 Anthropic 这样的 AI 巨头，还是来自较小的 AI 创业公司。非常期待看到这个领域在未来一年中的发展变化！

* * *

**喜欢这篇文章吗？** 关注我，以获取更多关于数据科学和 AI 的文章。

> [**ChatGPT 与 Claude 与 Gemini 数据分析对比（第一部分）**](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-1-821086810318)
> 
> [**ChatGPT 与 Claude 与 Gemini 数据分析对比（第二部分）：谁在 EDA 方面表现最佳？**](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-2-whos-the-best-at-eda-6ed5a4a6f008)
> 
> [**ChatGPT 与 Claude 与 Gemini 数据分析对比（第三部分）：最佳机器学习 AI 助手**](https://towardsdatascience.com/chatgpt-vs-claude-vs-gemini-for-data-analysis-part-3-best-ai-assistant-for-machine-learning-a2078793e4fa)
> 
> [**2024 年 ChatGPT 如何成为我的最佳单人旅行伙伴**](https://ydong029.medium.com/how-chatgpt-became-my-best-solo-travel-buddy-in-2024-be0c9a92f974)

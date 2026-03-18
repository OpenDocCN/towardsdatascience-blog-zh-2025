# 如何丰富 LLM 上下文以显著提升能力

> 原文：[`towardsdatascience.com/how-to-enrich-llm-context-to-significantly-enhance-capabilities/`](https://towardsdatascience.com/how-to-enrich-llm-context-to-significantly-enhance-capabilities/)

<mdspan datatext="el1757983411005" class="mdspan-comment">LLMs 是通过一个庞大的文本数据语料库进行训练的</mdspan>，在他们的预训练阶段，实际上几乎消耗了整个互联网。当 LLMs 能够访问所有相关数据以适当地回答用户问题时，它们就会蓬勃发展。然而，在许多情况下，我们通过不提供足够的数据来限制了我们 LLMs 的能力。在这篇文章中，我将讨论为什么你应该关心为我们的 LLMs 提供更多数据，如何获取这些数据，以及具体的应用。

我也会在我的文章中开始一个新的特色：写出我的主要目标，我想通过这篇文章达到什么目标，以及你阅读后应该知道什么。如果成功，我将开始在我的每篇文章中这样做：

> **我的目标**是强调为 LLMs 提供相关数据的重要性，以及你如何将数据输入到你的 LLMs 中以提升性能

![LLMs 对数据的需求很大。学会为你的 LLMs 添加更多数据以提升性能。](img/f736f74f006c2af0aad71015cccb8f64.png)

在这篇文章中，我强调了你如何通过向你的 LLMs 添加更多数据来提升 LLMs 的性能。图片由 ChatGPT 提供。

你还可以阅读我的文章《如何在 3 步中分析和优化你的 LLMs》([How to Analyze and Optimize Your LLMs in 3 Steps](https://towardsdatascience.com/how-to-analyze-and-optimize-your-llms-in-3-steps/))和《使用多模态 LLMs 进行文档问答》([Document QA using Multimodal LLMs](https://eivindkjosbakken.com/2025/07/13/using-a-multimodal-document-ml-model-to-query-your-documents/))。

## 目录

+   为什么要把更多数据添加到 LLMs 中？

+   寻找额外的元数据

+   元数据信息检索

    +   事先检索信息

    +   按需信息检索

+   额外数据在 LLM 中的应用案例

    +   元数据过滤搜索

    +   AI 代理互联网搜索

+   结论

## 为什么要把更多数据添加到 LLMs 中？

我将从指出为什么这很重要开始我的文章。LLMs 对数据的需求极大，这意味着它们需要大量数据才能 -工作良好。这通常在 LLMs 的预训练语料库中体现出来，该语料库由用于训练 LLMs 的万亿个文本标记组成。

> 在预训练时代，重要的是互联网文本。你主要希望有一个大型的、多样化的、高质量的互联网文档集合来学习。
> 
> 在监督微调时代，重要的是对话。雇佣合同工来为问题创建答案，有点…… [`t.co/rR6yYZGgKP`](https://t.co/rR6yYZGgKP)
> 
> — Andrej Karpathy (@karpathy) [2025 年 8 月 27 日](https://twitter.com/karpathy/status/1960803117689397543?ref_src=twsrc%5Etfw)

Andrej Karpathy 在推特上关于用于 LLM 的数据

然而，大量使用数据的理念也适用于 LLM 在推理时间（在生产中使用 LLM 时）的应用。你需要向 LLM 提供所有必要的数据以回答用户请求。

在很多情况下，你由于没有提供相关信息而不自觉地降低了 LLM 的性能。

例如，如果你创建了一个问答系统，用户可以上传文件并与它们交谈。自然地，你提供了每个文件的文本内容，以便用户可以与文档聊天；然而，例如，你可能忘记将文档的**文件名**添加到用户正在聊天的上下文中。这将影响 LLM 的性能，例如，如果某些信息仅存在于文件名中或用户在聊天中引用了文件名。一些其他具体的 LLM 应用，其中额外的数据是有用的包括：

+   分类

+   信息提取

+   关键词搜索以找到相关文档以供 LLM 使用

在本文的其余部分，我将讨论你可以在哪里找到这样的数据，检索额外数据的技术，以及一些数据的具体用例。

## 寻找额外的元数据

在本节中，我将讨论你可能在应用程序中已经可用的数据。一个例子是我最后的类比，其中你有一个文件问答系统，但忘记了将文件名添加到上下文中。其他一些例子包括：

+   文件扩展名（.pdf, .docx, .xlsx）

+   文件夹路径（如果用户上传了一个文件夹）

+   时间戳（例如，如果用户询问最新的文档，这是必需的）

+   页码（用户可能要求 LLM 获取位于第 5 页的特定信息）

![元数据的替代方案](img/6631fd20640dafaeb02b65fc6e6b0c91.png)

这张图片突出了你可以获取的不同类型的元数据，包括文件类型、文件夹路径、时间戳和页码。图片由 Google Gemini 提供。

有大量的其他此类数据示例，你可能已经拥有，或者可以快速获取并添加到你的 LLM 上下文中。

你可用的数据类型将在不同的应用中差异很大。我在本文中提供的许多示例都是针对基于文本的 AI，因为这是我花费时间最多的领域。然而，如果你，例如，更多地从事视觉 AI 或基于音频的 AI，我敦促你找到你所在领域的类似示例。

对于视觉人工智能，它可能是：

+   图片/视频拍摄位置的地理数据

+   图片/视频文件的文件名

+   图片/视频文件的作者

或者对于音频 AI，它可能是

+   关于何时谁在说话的元数据

+   每句话的时间戳

+   音频录制位置的地理数据

我的观点是，外面有大量的可用数据；你所需要做的就是去寻找它，并考虑它如何对你的应用有用。

## 元数据的信息检索

有时候，您已经拥有的数据可能不足以满足需求。您希望为您的 LLM 提供更多数据以帮助它适当地回答问题。在这种情况下，您需要检索额外的数据。自然地，因为我们正处于 LLM 时代，我们将利用 LLM 来检索这些数据。

### 事先检索信息

最简单的方法是在处理任何实时请求之前检索额外的数据。对于文档 AI 来说，这意味着在处理过程中从文档中提取特定信息。您可能需要提取**文档类型**（法律文件、税务文件或销售手册）或文档中包含的特定信息（日期、姓名、地点等）。

事先获取信息的优势是：

+   速度（在生产中，您只需从您的数据库中检索值）

+   您可以利用[批量处理](https://platform.openai.com/docs/guides/batch)来降低成本

现在，检索这类信息相当简单。您只需设置一个具有特定系统提示的 LLM 来获取信息，然后将提示和文本一起输入 LLM。然后 LLM 将处理文本并为您提取相关信息。您可能希望考虑评估您信息提取的性能，在这种情况下，您可以阅读我的文章[使用自动评估评估 500 万 LLM 请求](https://eivindkjosbakken.com/2025/09/14/how-to-evaluate-5-million-documents-with-automatic-llm-evaluations/)。

您可能还希望规划所有要检索的信息点，例如：

+   位置

+   日期

+   姓名

+   …

当您创建了此列表后，您可以检索所有元数据并将其存储在数据库中。

然而，事先获取信息的最大缺点是您必须预先确定要提取的信息。在很多情况下这是困难的，在这种情况下，您可以进行实时信息检索，我将在下一节中介绍。

### 按需信息检索

当您无法事先确定要检索哪些信息时，您可以按需检索。这意味着设置一个通用函数，该函数接受要提取的数据点和提取它的文本。例如

```py
import json
def retrieve_info(data_point: str, text: str) -> str:
    prompt = f"""
        Extract the following data point from the text below and return it in a JSON object.

        Data Point: {data_point}
        Text: {text}

        Example JSON Output: {{"result": "example value"}}
    """

    return json.loads(call_llm(prompt))
```

您将此函数定义为 LLM 可以访问的工具，并且它可以在需要信息时调用它。这本质上就是[Anthropic 建立他们的深度研究系统](https://www.anthropic.com/engineering/multi-agent-research-system)的方式，他们创建了一个可以产生子代理以检索额外信息的协调代理。请注意，给予您的 LLM 使用额外提示的访问权限可能会导致大量令牌使用，因此您应该注意您 LLM 的令牌花费。

## LLM 用于元数据的用例

到目前为止，我已经讨论了为什么您应该利用额外的数据以及如何获取它。然而，为了完全理解这篇文章的内容，我还会提供一些具体的应用，这些数据可以改善 LLM 的性能。

### 元数据过滤搜索

![元数据过滤](img/9ce36c1a217d160a4ecccd5d5a58b46c.png)

此图展示了元数据过滤搜索是如何进行的，您可以使用元数据过滤来过滤掉不相关的文档。图片由 Google Gemini 提供。

我的第一个例子是，您可以使用元数据过滤进行搜索。提供如下信息：

+   文件类型（pdf, xlsx, docx，…）

+   文件大小

+   文件名

这可以在获取相关信息时帮助您的应用程序。例如，当执行 RAG 时，可以将信息提取出来以供您的 LLM 的上下文使用。您可以使用额外的元数据来过滤掉不相关的文件。

用户可能只询问与 Excel 文档相关的问题。因此，使用 RAG 从除 Excel 文档以外的文件中获取块是 LLM 上下文窗口的糟糕使用。相反，您应该过滤可用的块，只找到 Excel 文档，并利用 Excel 文档的块来最好地回答用户的查询。您可以在我的关于构建有效 AI 代理的文章中了解更多关于处理 LLM 上下文的信息（[我的文章](https://towardsdatascience.com/how-to-build-effective-ai-agents-to-process-millions-of-requests/)）。

### AI 代理互联网搜索

另一个例子是，如果您询问 AI 代理关于 LLM 预训练截止日期之后发生的近期历史问题。LLMs 通常有一个预训练数据的截止日期，因为数据需要被精心策划，并且保持其完全更新是具有挑战性的。

当用户询问关于近期历史的问题时，例如关于新闻中的近期事件，这会带来一个问题。在这种情况下，回答查询的 AI 代理需要访问互联网搜索（本质上是在互联网上执行信息提取）。这是一个按需信息提取的例子。

## 结论

在这篇文章中，我讨论了如何通过提供额外数据来显著提高您的 LLM。您可以从现有的元数据（文件名、文件大小、位置数据）中找到这些数据，或者您可以通过信息提取（文档类型、文档中提到的名称等）来检索这些数据。这些信息对于 LLM 成功回答用户查询的能力通常至关重要，在许多情况下，缺乏这些数据几乎保证了 LLM 无法正确回答问题。

**👉 我的免费电子书和网络研讨会：**

📚 [获取我的免费视觉语言模型电子书](https://eivindkjosbakken.com/ebook)

💻 [我的关于视觉语言模型网络研讨会](https://www.eivindkjosbakken.com/webinar)

**👉 在社交平台上找到我：**

📩 [订阅我的通讯](https://eivindkjosbakken.com/newsletter)

🧑‍💻 [联系我](https://eivindkjosbakken.com/)

🔗 [LinkedIn](https://www.linkedin.com/in/eivind-kjosbakken/)

🐦 [X / Twitter](https://x.com/EivindKjos)

✍️ [Medium](https://oieivind.medium.com/)

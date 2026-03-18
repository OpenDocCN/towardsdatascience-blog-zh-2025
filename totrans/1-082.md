# 介绍 Google 的文件搜索工具

> 原文：[`towardsdatascience.com/introducing-googles-file-search-tool/`](https://towardsdatascience.com/introducing-googles-file-search-tool/)

<mdspan datatext="el1763425676090" class="mdspan-comment">大型语言模型</mdspan>（LLMs）如 Gemini 已经彻底改变了软件开发中可能实现的事情。它们理解、生成和推理文本的能力令人瞩目。然而，它们有一个根本的限制：它们只知道它们被训练的内容。它们不了解您公司的内部文档、您项目的特定代码库或昨天发布的最新研究论文。

为了构建智能和实用的应用程序，我们需要弥合这个差距，并将模型的广泛推理能力与您自己的特定、私有数据相结合。这是检索增强生成（RAG）的领域。这项强大的技术从通常的外部知识库中检索相关信息。然后，它将其作为上下文提供给 LLM，以生成更准确、适当和可验证的响应。

虽然非常有效，但从头开始构建一个健壮的 RAG 管道是一个重大的工程挑战。它涉及一系列复杂的步骤：

+   数据摄取和分块。解析各种文件格式（PDF、DOCX 等）并将它们智能地分割成更小、语义上有意义的块。

+   嵌入生成。使用嵌入模型将这些文本块转换为数值向量表示。

+   向量存储。设置、管理和扩展一个专门的向量数据库来存储这些嵌入，以实现高效的搜索。

+   检索逻辑。实现一个系统，将用户的查询嵌入其中，并在向量数据库中执行相似性搜索以找到最相关的块。

+   上下文注入。以 LLM 能够有效使用信息的方式动态地将检索到的块插入到提示中。每个这些步骤都需要仔细考虑、基础设施管理和持续维护。

每个这些步骤都需要仔细考虑、基础设施管理和持续维护。

最近，Google 继续努力结束我们熟知的传统 RAG，它又收购了另一个针对这个领域的新产品。Google 的新**文件搜索**工具完全消除了您在文档上执行语义搜索之前需要分块、嵌入和向量化文档的需求。

## 什么是 Google 文件搜索工具？

在其核心，文件搜索工具是一个强大的抽象层，覆盖了完整的 RAG 管道。它处理您数据的一生，从摄取到检索，为 Gemini 的响应提供一个简单而强大的方式，使其在您的文档中得到具体体现。

让我们分解其核心组件和它们解决的问题。

1) 简单、集成的开发者体验

文件搜索不是一个单独的 API 或需要您编排的复杂外部服务。它作为 **工具** 直接集成在现有的 GeminiAPI**** 中。这种无缝集成使您只需添加几行额外的代码即可将强大的 RAG 功能添加到您的应用程序中。该工具自动...

+   安全存储您上传的文档。

+   采用复杂策略将您的文档分解成适当大小、连贯的块，以获得最佳检索结果。

+   处理您的文件，使用 Google 的最先进模型生成嵌入，并对其进行索引以实现快速检索。

+   处理检索并将相关上下文注入发送给 Gemini 的提示中。

2) 核心强大的向量搜索

检索引擎由 **gemini-embedding-001** 模型提供支持，该模型专为高性能语义搜索而设计。与传统仅查找精确匹配的关键词搜索不同，向量搜索理解查询的 **意义和上下文**。这使得它能够从您的文档中提取相关信息，即使用户的查询使用了完全不同的措辞。

3) 内置引用以验证

信任和透明度对于企业级 AI 应用至关重要。文件搜索工具自动将基础元数据包含在模型的响应中。这些元数据包含引用，指定用于生成答案的确切来源文档的哪些部分。

这是一个重要的功能，允许您：-

+   验证准确性。轻松检查模型的来源以确认其响应的正确性。

+   建立用户信任。向用户展示信息来源，增加他们对系统的信心。

+   启用更深入的探索。提供到源文档的链接，使用户能够更深入地探索感兴趣的主题。

4. 支持广泛的格式。

知识库很少由简单的文本文件组成。文件搜索工具开箱即支持广泛的标准化文件格式，包括 PDF、DOCX、TXT、JSON 以及各种编程语言和应用文件格式。这种灵活性意味着您可以从现有的文档中构建一个全面的知识库，而无需执行繁琐的预处理或数据转换步骤。

5. 可负担性

Google 使其文件搜索工具的使用极具成本效益。存储和查询嵌入免费。您只需为初始文档内容的任何嵌入付费，费用低至每 100 万个令牌 0.15 美元（例如，基于 gemini-embedding-001 嵌入模型）。

## 使用文件搜索

现在我们对文件搜索工具有了更好的了解，是时候看看我们如何在我们的工作流程中使用它了。为此，我将展示一些示例 Python 代码，展示如何调用和使用文件搜索。

然而，在那之前，最佳实践是设置一个独立的开发环境，以保持我们的各种项目彼此隔离。

我将使用 UV 工具来做这件事，并在 WSL2 Ubuntu for Windows 下的 Jupyter 笔记本中运行我的代码。但是，请随意使用最适合您的包管理器。

```py
$ cd projects
$ uv init gfs
$ cd gfs
$ uv venv
$ source gfs/bin/activate
(gfs) $ uv pip install google-genai jupyter
```

您还需要一个 Gemini API 密钥，您可以通过以下链接从 Google 的 AI Studio 主页获取。

[**Google AI Studio**](https://aistudio.google.com/)

登录后，在屏幕左下角附近寻找“获取 API 密钥”链接。

## 示例代码—在 PDF 文档上进行简单搜索

为了测试目的，我从三星 S25 手机的网站上下载了用户手册到我的本地台式电脑。它有超过 180 页。您可以使用[此链接](https://downloadcenter.samsung.com/content/UM/202505/20250522202232085/SM-S93X_UG_EU_15_Eng_Rev.2.0_250514.pdf)获取。

启动 Jupyter 笔记本，并在一个单元中输入以下代码。

```py
import time
from google import genai
from google.genai import types

client = genai.Client(api_key='YOUR_API_KEY')
store = client.file_search_stores.create()

upload_op = client.file_search_stores.upload_to_file_search_store(
    file_search_store_name=store.name,
    file='SM-S93X_UG_EU_15_Eng_Rev.2.0_250514.pdf'
)

while not upload_op.done:
  time.sleep(5)
  upload_op = client.operations.get(upload_op)

# Use the file search store as a tool in your generation call
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What models of phone does this document apply to ...',
    config=types.GenerateContentConfig(
        tools=[types.Tool(
            file_search=types.FileSearch(
                file_search_store_names=[store.name]
            )
        )]
    )
)
print(response.text)
```

在导入所需的库之后，我们创建了一个“文件搜索存储”，这是一个用于存储您上传的文件数据和索引的容器。接下来，我们将输入文件上传到存储中，并等待上传完成。

接下来，我们调用**generate_content**函数，该函数将回答我们向所选模型（例如我们的示例中的 Gemini 2.5 flash）提出的问题，关于我们的输入文件，然后在打印出模型的响应之前。

这里是运行上述代码时得到的响应。

```py
This document applies to the following phone models: 
SM-S931B, 
SM-S931B/DS, 
SM-S936B, 
SM-S936B/DS, 
SM-S937B, 
SM-S937B/DS, 
SM-S938B, and SM-S938B/DS. It also specifically mentions 
Galaxy S25 Ultra, Galaxy S25 Edge, Galaxy S25+, and Galaxy S25.
```

我们可以通过检查 PDF 的第一页来验证该信息的准确性，其中提供了相关模型的列表。

![图片](img/889a19fa07eacdf35a89a5d22f3d4acd.png)

图片由作者提供

让我们深入挖掘，找到一个更复杂的问题。比如说，你想知道如何在不使用时自动关闭屏幕。在 PDF 的第 156 页上，它说：

> 您可以设置屏幕在您不使用时自动关闭。打开设置，点击显示→屏幕超时，然后选择设备关闭屏幕前需要等待的时间长度。

文件搜索工具能识别这个吗？

```py
...
...
...

# Use the file search store as a tool in your generation call
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='How do I set the screen to turn off automatically when not in use',
    config=types.GenerateContentConfig(
        tools=[types.Tool(
            file_search=types.FileSearch(
                file_search_store_names=[store.name]
            )
        )]
    )
)
print(response.text)
```

那么响应呢？

```py
To set your device's screen to turn off automatically when not in use, 
you can adjust the "Screen timeout" setting. This setting determines how 
long the device waits before turning off the display's backlight.

For a Samsung device (as indicated by the provided user guide), you can 
typically find this option by navigating to:
Settings → Display → Screen timeout.

There, you can select the desired length of time before the screen turns off.
```

注意，文件搜索功能在回答问题时也可以利用其模型自身的内部知识库，而无需一定咨询文档存储来找到答案。

## 处理多个输入文件

如果您的文档语料库由多个文件组成，使用简单的 for 循环很容易将它们全部包含进来，但您应该注意文件搜索的一些限制。根据谷歌自己的文档，这些限制是，

> 文件搜索 API 有以下限制，以确保服务稳定性：
> 
> 最大文件大小/每文档限制：100 MB
> 
> 项目文件搜索存储的总大小（基于用户级别）：
> 
> 免费：1 GB
> 
> 第一级：10 GB
> 
> 第二级：100 GB
> 
> 第三级：1 TB

## 控制分块

当文件添加到文件搜索存储时，系统会自动将其分割成更小的块，嵌入和索引内容，然后上传。如果你想调整这种分割方式，可以使用**chunking_config**选项来设置块大小的限制，并指定块之间应该有多少个 token 重叠。以下是一个代码片段，展示了如何做到这一点。

```py
...
...

operation = client.file_search_stores.upload_to_file_search_store(
    file_search_store_name=file_search_store.name,
    file='SM-S93X_UG_EU_15_Eng_Rev.2.0_250514.pdf'
    config={
        'chunking_config': {
          'white_space_config': {
            'max_tokens_per_chunk': 200,
            'max_overlap_tokens': 20
          }
        }
    }
)
...
...
```

## 文件搜索与谷歌的其他 RAG 相关工具（如上下文定位和 LangExtract）有何不同？

我最近撰写了两篇关于谷歌在这个领域类似产品的文章：上下文定位和 LangExtract。表面上，它们做的是类似的事情。这是正确的——直到某个程度。

主要区别在于，文件搜索**实际上**是一个 RAG 产品，因为它永久存储你的文档嵌入，而其他两个工具则不是。这意味着一旦你的嵌入存储在文件搜索存储中，它们将永远保留，或者直到你选择删除它们。你不必每次想要回答关于文件的问题时都重新上传文件。

这里有一个方便的表格，用于参考不同之处。

```py
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Feature            | Google File Search                   | Google Context Grounding              | LangExtract                          |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Primary Goal       | To answer questions and generate     | Connects model responses to verified  | Extract specific, structured data    |
|                    | content from private documents.      | sources to improve accuracy and       | (like JSON) from unstructured text.  |
|                    |                                      | reduce hallucinations.                |                                      |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Input              | User prompt and uploaded files       | User prompt and configured data       | Unstructured text plus schema or     |
|                    | (PDFs, DOCX, etc.).                  | source (e.g., Google Search, URL).    | prompt describing what to extract.   |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Output             | Conversational answer grounded in    | Fact-checked natural language answer  | Structured data (e.g., JSON) mapping |
|                    | provided files with citations.       | with links or references.             | info to original text.               |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Underlying Process | Managed RAG system that chunks,      | Connects model to info source; uses   | LLM-based library for targeted info  |
|                    | embeds, and indexes files.           | File Search, Google Search, etc.      | extraction via examples.             |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
| Typical Use Case   | Chatbot for company knowledge base   | Answering recent events using live    | Extracting names, meds, dosages from |
|                    | or manuals.                          | Google Search results.                | clinical notes for a database.       |
+--------------------+--------------------------------------+---------------------------------------+--------------------------------------+
```

## 删除文件搜索存储

谷歌在 48 小时后自动从其文件存储中删除你的原始文件内容，但保留文档嵌入，允许你继续查询文档内容。如果你决定它们不再需要，你可以删除它们。这可以通过下面的代码片段以编程方式完成。

```py
...
...
...
# deleting the stores
# List all your file search stores
for file_search_store in client.file_search_stores.list():
    name = file_search_store.name
    print(name)

# Get a specific file search store by name
my_file_search_store = client.file_search_stores.get(name='your_file_search_store_name')

# Delete a file search store
client.file_search_stores.delete(name=my_file_search_store.name, config={'force': True})
```

## 摘要

传统上，构建 RAG 管道需要复杂的步骤——摄入数据，将其分割成块，生成嵌入，设置向量数据库，并将检索到的上下文注入到提示中。谷歌的新文件搜索工具抽象出所有这些任务，提供了一种完全管理的、端到端的 RAG 解决方案，通过**generateContent**调用直接集成到 Gemini API 中。

在本文中，我在提供其使用的完整 Python 代码示例之前，概述了文件搜索的一些关键功能和优势。我的示例演示了将大型 PDF 文件（三星手机手册）上传到文件搜索存储，并通过 Gemini 模型和 API 查询它以准确提取特定信息。我还展示了你可以使用的代码，以微管理你的文档的分割策略，如果文件搜索默认的分割策略不符合你的需求。最后，为了将成本降至最低，我还提供了一个代码片段，展示了如何在你完成使用后删除不需要的存储。

在我撰写本文时，我突然想到，从表面上看，这个工具与我之前撰写过的许多谷歌产品在这个领域有许多相似之处，即 LangExtract 和上下文定位。然而，我继续解释说，每个工具都有其关键的区别，其中文件搜索是三个工具中唯一的真正 RAG 系统，并以易于阅读的表格格式突出了这些区别。

在这篇文章中，我无法涵盖谷歌文件搜索工具的更多内容，包括文件元数据和引用的使用。我鼓励您使用以下链接在线探索谷歌的 API 文档，以获得所有文件搜索功能的全面描述。

[`ai.google.dev/gemini-api/docs/file-search#file-search-stores`](https://ai.google.dev/gemini-api/docs/file-search#file-search-stores)

注意：我与谷歌或本文中提到的任何工具都没有关联。

# 评估您的 RAG 解决方案

> 原文：[`towardsdatascience.com/evaluating-your-rag-solution/`](https://towardsdatascience.com/evaluating-your-rag-solution/)

## 简介

<mdspan datatext="el1758064165191" class="mdspan-comment">检索增强生成</mdspan> (RAG) 解决方案无处不在。在过去的几年里，我们见证了它们在客户服务、医疗保健、情报等领域以及更多地方迅速增长，因为组织使用 RAG 或混合-RAG 解决方案。**但是，我们如何评估这些解决方案？我们可以使用哪些方法来确定我们 RAG 模型的优势和劣势？**

本文将通过使用 [LangChain](https://python.langchain.com/docs/introduction/) 和更多工具，结合开源研究数据构建自己的聊天机器人来介绍 RAG。我们还将利用 [DeepEval](https://deepeval.com/) 来评估我们的 RAG 管道，包括检索器和生成器。最后，我们将讨论测试 RAG 解决方案的方法。

## 检索增强生成

随着 LLM 的出现，当这些“基础”预训练模型在大量数据集上训练后给出错误答案时，引发了大量的批评。**随之而来的是检索增强生成（RAG），这是一种结合搜索和生成能力，在生成响应之前参考上下文特定信息的解决方案。**

![](img/867f072dd90d5681343d1de764ffb97e.png)

图片由作者提供

由于其减少幻觉和增强事实性的能力，RAG 在过去几年中变得非常流行。它们灵活、易于更新，并且比微调大型语言模型便宜得多。现在我们每天都会遇到 RAG 解决方案。例如，许多组织利用 RAG 为员工构建内部聊天机器人，以导航他们的知识库，以及构建外部聊天机器人以支持客户服务和其他业务功能。

## 构建 RAG 管道

对于我们的 RAG 解决方案，我们将使用与人工智能相关的开源研究摘要。我们可以使用这些数据来生成更多与人工智能、机器学习等相关问题的“技术性”答案。

*所使用的数据来自 OpenAlex API ([`openalex.org/`](https://openalex.org/))。这是一个包含世界各地开源研究的数据库/目录。数据在无权利保留许可（CC0 许可）下免费访问。*

### 数据摄取

首先，我们需要使用 OpenAlex API 加载数据。下面是按出版年份和关键词进行搜索的代码。我们使用“深度学习”、“自然语言处理”、“计算机视觉”等关键词进行 AI/ML 的搜索。

```py
import pandas as pd
import requests

def import_data(pages, start_year, end_year, search_terms):

    """
    This function is used to use the OpenAlex API, conduct a search on works, a return a dataframe with associated works.

    Inputs: 
        - pages: int, number of pages to loop through
        - search_terms: str, keywords to search for (must be formatted according to OpenAlex standards)
        - start_year and end_year: int, years to set as a range for filtering works
    """

    #create an empty dataframe
    search_results = pd.DataFrame()

    for page in range(1, pages):

        #use paramters to conduct request and format to a dataframe
        response = requests.get(f'https://api.openalex.org/works?page={page}&per-page=200&filter=publication_year:{start_year}-{end_year},type:article&search={search_terms}')
        data = pd.DataFrame(response.json()['results'])

        #append to empty dataframe
        search_results = pd.concat([search_results, data])

    #subset to relevant features
    search_results = search_results[["id", "title", "display_name", "publication_year", "publication_date",
                                        "type", "countries_distinct_count","institutions_distinct_count",
                                        "has_fulltext", "cited_by_count", "keywords", "referenced_works_count", "abstract_inverted_index"]]

    return(search_results)

#search for AI-related research
ai_search = import_data(30, 2018, 2025, "'artificial intelligence' OR 'deep learn' OR 'neural net' OR 'natural language processing' OR 'machine learn' OR 'large language models' OR 'small language models'")
```

当查询 OpenAlex 数据库时，摘要以倒排索引的形式返回。下面是一个取消倒排索引并返回摘要原始文本的函数。

```py
def undo_inverted_index(inverted_index):

    """
    The purpose of the function is to 'undo' and inverted index. It inputs an inverted index and
    returns the original string.
    """

    #create empty lists to store uninverted index
    word_index = []
    words_unindexed = []

    #loop through index and return key-value pairs
    for k,v in inverted_index.items(): 
        for index in v: word_index.append([k,index])

    #sort by the index
    word_index = sorted(word_index, key = lambda x : x[1])

    #join only the values and flatten
    for pair in word_index:
        words_unindexed.append(pair[0])
    words_unindexed = ' '.join(words_unindexed)

    return(words_unindexed)

#create 'original_abstract' feature
ai_search['original_abstract'] = list(map(undo_inverted_index, ai_search['abstract_inverted_index']))
```

### 创建向量数据库

接下来，我们需要生成表示摘要的嵌入，并将它们存储在向量数据库中。**利用向量数据库是一个最佳实践，因为它们是为低延迟查询设计的，并且可以扩展以处理数十亿的数据点。**它们还使用专门的索引和最近邻算法，根据上下文和/或语义相似性快速检索数据，对于 LLM 应用至关重要。

![图片](img/c7ef6be75b9358e409fa286dba2db02a.png)

图片由作者提供

首先，我们从 LangChain 导入必要的库，并从 HuggingFace 加载我们的嵌入模型。虽然我们可能使用更大的嵌入模型可以得到更好的结果，但我决定使用较小的模型来强调此管道的速度。

*您可以通过使用 Hugging Face 的 MTEB 排行榜（[*https://huggingface.co/spaces/mteb/leaderboard*](https://huggingface.co/spaces/mteb/leaderboard)）根据大小、性能、预期用途等比较和查找嵌入模型。*

```py
from langchain_community.docstore.in_memory import InMemoryDocstore
from langchain_community.vectorstores import FAISS
from langchain_huggingface.embeddings import HuggingFaceEmbeddings
from langchain_core.documents import Document

#load embedding model
embeddings = HuggingFaceEmbeddings(model_name="thenlper/gte-small")
```

接下来，我们使用 FAISS（或 LangChain 对 FAISS 的包装）创建我们的向量数据库。我们首先创建索引，格式化我们的数据和文档，同时存储它们的元数据（标题和年份）。然后我们创建一个 ID 列表，将文档和 ID 添加到数据库中，并将数据库本地保存。

```py
#save index with faiss
index = faiss.IndexFlatL2(len(embeddings.embed_query("hello world")))

#format abstracts as documents
documents = [Document(page_content=ai_search['original_abstract'][i], metadata={"title": ai_search['title'][i], "year": ai_search['publication_year'][i]}) for i in range(len(ai_search))]

#create list of ids as strings
n = len(ai_search)
ids = list(range(1, n + 1))
ids = [str(x) for x in my_list]

#add documents to vector store
vector_store.add_documents(documents=documents, ids=ids)

#save the vector store
vector_store.save_local("Data/faiss_index")
```

使用 LangChain 的向量存储，我们可以直接查询我们的文档。让我们通过搜索“计算机视觉”来快速测试这一点。我们可以在下面看到，返回的第一个文档，“使用 OPENCV 进行人脸检测和识别”，与计算机视觉高度相关。

```py
#test that vector database is working
vector_store.similarity_search("computer vision", k=3)
```

> `[文档(id='783', 元数据={'标题': '使用 OPENCV 进行人脸检测和识别', '年份': 2020}, 页面内容='计算机视觉是人工智能领域最迷人且最具挑战性的任务之一。计算机视觉充当计算机软件和我们所看到的视觉之间的桥梁。它使我们能够...')`

### 创建 RAG 管道

现在让我们开发我们的 RAG 管道。RAG 解决方案的一个主要组件是用于生成响应的生成模型。为此，我们将使用来自 LangChain 的 OpenAI 模型。

因此，我们可以比较在实现 RAG 管道前后响应的变化，让我们询问“基础”模型：“计算机视觉领域最近有哪些最新进展？”。

```py
from langchain_openai import OpenAI
from langchain.chains import RetrievalQA
from langchain import PromptTemplate

#set API key
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY","API KEY") 

#load llm
llm = OpenAI(openai_api_key=OPENAI_API_KEY)

#test llm response
llm.invoke("What are the most recent advancements in computer vision?")
```

> ‘\n\n1. 深度学习：深度学习是机器学习的一个子集，在计算机视觉任务（如目标检测、识别和图像分类）中取得了显著的进展。它使用具有多个隐藏层的神经网络从图像中学习和提取特征，从而实现更准确和高效的结果。\n\n2. 生成对抗网络（GANs）：GANs 是一种深度学习算法，通过学习大量数据集来生成新的图像。它们已在图像合成、超分辨率和图像到图像翻译等任务中使用，并在创建逼真图像方面显示出令人印象深刻的结果。\n\n3. 卷积神经网络（CNNs）：CNNs 是一种深度学习算法，彻底改变了计算机视觉领域。它们在从图像中提取特征方面非常有效，已在图像分类、目标检测和分割等任务中使用。\n\n4. 迁移学习：迁移学习允许在没有从头开始的情况下，将预训练模型用于不同的任务或数据集。它在计算机视觉任务中显示出有希望的结果，尤其是在训练数据有限的任务中。\n\n5. 图像分割：随着深度学习的发展，图像分割变得更加准确和高效。它涉及将图像分割成不同的区域或段以识别对象

从上面的回答中，我们可以看到计算机视觉的一般概述，对其工作原理的高级描述，以及不同类型的模型和应用。**虽然这是一个很好的总结，但它并没有直接回答我们的问题。这是一个 RAG 的大好机会！**

接下来，我们将构建我们的 RAG 管道组件。首先，我们需要一个**检索器**来获取与我们的查询相关的最相关的 k 个文档。然后，我们构建一个**提示**来指导我们的模型如何回答问题。最后，我们将它们与**基础生成模型**结合起来，以创建我们的管道。

让我们快速重测我们的查询：“计算机视觉中最新的进展是什么？”

```py
#test that vector database is working
retriever = db.as_retriever(search_kwargs={"k": 3})

#create a prompt template
template = """<|user|>
Relevant information:
{context}

Provide a concise answer to the following question using relevant information provided above:
{question}
If the information above does not answer the question, say that you do not know. Keep answers to 3 sentences or shorter.<|end|>
<|assistant|>"""

#define prompt template
prompt = PromptTemplate(
    template=template,
    input_variables=["context", "question"])

#create RAG pipeline
rag = RetrievalQA.from_chain_type(llm=llm, chain_type="stuff", retriever=retriever, return_source_documents=True,
                                  chain_type_kwargs={"prompt": prompt}, verbose = True)

#test rag response
rag.invoke("What are the most recent advancements in computer vision?")
```

> 计算机视觉最新的进展包括具有视觉能力的大型语言模型的出现，例如 OpenAI 的 GPT-4V、谷歌的 Bard AI 和微软的 Bing AI。这些模型能够分析图像，并具有访问实时信息的能力，使它们直接嵌入到许多应用中。随着 AI 的快速发展，预计将进一步取得进展。

这个回答在回答我们的问题方面做得更好。它直接指出了最新的进展，提到了具体的特性、模型以及它们如何推动计算机视觉的发展。这对于我们的技术聊天机器人来说是一个有希望的结果。

**但这不是一个恰当的评价。接下来，我们将进一步在我们的 RAG 解决方案上测试多个指标，以帮助我们确定它是否准备好投入生产。**

* * *

## LLM-as-a-Judge

为了开始评估我们的解决方案，我们将使用另一个生成模型来确定我们的 RAG 解决方案是否符合某些标准。虽然 LLM 作为裁判的方法有一些注意事项并且需要谨慎使用，但它们提供了很多灵活性和效率。它们还可以在评估过程中提供详细的见解，如下所示。

我们的 RAG 由 2 个主要组件组成，即检索器和生成器。**我们将分别评估这些组件**。我们的发现可能会促使我们调整超参数，替换嵌入模型，或使用不同的生成模型。

### 检索器评估

首先，我们将评估我们的检索器，即获取相关内容的组件。我们将基于 3 个指标进行判断：

1.  **上下文精确度**：代表检索系统正确排名相关节点的能力更强。它首先使用一个 LLM 来确定每个节点是否与输入相关，然后计算加权累积精确度。

1.  **上下文召回率**：代表检索系统从知识库中可用的总相关集中捕获所有相关信息的更强能力。

1.  **上下文相关性**：评估给定输出所呈现信息的整体相关性。

首先，我们导入库并初始化指标。

```py
from deepeval import evaluate
from deepeval.test_case import LLMTestCase, LLMTestCaseParams
from deepeval.metrics import (
    ContextualPrecisionMetric,
    ContextualRecallMetric,
    ContextualRelevancyMetric)

#set your OpenAI API key
os.environ["OPENAI_API_KEY"] = "API KEY"

# Initialize metrics
contextual_precision = ContextualPrecisionMetric()
contextual_recall = ContextualRecallMetric()
contextual_relevancy = ContextualRelevancyMetric()
```

**接下来，我们需要构建一个测试用例，即给定查询的预期输出。这些测试数据集可能很难构建，并且应该有来自理解可能被提出的问题及其答案的领域专家的输入。**

对于这个例子，我们将只创建一个带有模拟预期输出的测试用例。这不会给我们真正的结果，但会给我们一个示例结果。

```py
#define user query
input = 'What are the most recent advancements in computer vision?'

#RAG output
actual_output = rag.invoke(input)['result']

#contexts used from the retriver
retrieved_contexts = []
for el in range(0,3):
  retrieved_contexts.append(rag.invoke(input)['source_documents'][el].page_content)

#expected output (example)
expected_output = 'Recent advancements in computer vision include Vision-Language Models (VLMs) that merge vision and language, Neural Radiance Fields (NeRFs) for 3D scene generation, and powerful Diffusion Models and Generative AI for creating realistic visuals. Other key areas are Edge AI for real-time processing, enhanced 3D vision techniques like NeRFs and Visual SLAM, advanced self-supervised learning methods, deepfake detection systems, and increased focus on Ethical AI and Explainable AI (XAI) to ensure fairness and transparency.'
```

使用上述组件，我们现在可以构建我们的测试用例并计算我们的 3 个指标。

```py
#create test case
test_case = LLMTestCase(
    input=input,
    actual_output=actual_output,
    retrieval_context=retrieved_contexts,
    expected_output=expected_output)

#compute contextual precision and print results
contextual_precision.measure(test_case)
print("Score: ", contextual_precision.score)
print("Reason: ", contextual_precision.reason)

#compute contextual recall and print results
contextual_recall.measure(test_case)
print("Score: ", contextual_recall.score)
print("Reason: ", contextual_recall.reason)

#compute relevancy precision and print results
contextual_relevancy.measure(test_case)
print("Score: ", contextual_relevancy.score)
print("Reason: ", contextual_relevancy.reason)
```

> 分数：1.0 原因：分数为 1.00 是因为相关节点被排在最前面：第一个节点讨论了‘计算机视觉算法的近期进展’和‘显著成就’，第二个节点涵盖了‘计算机视觉的演变’和基础进步。那个不相关的节点，仅描述了 OpenCV 工具包且缺乏对近期进展的讨论，被正确地排在最后。这种完美的排序确保了最高的上下文精确度。
> 
> 分数：0.0 原因：分数为 0.00 是因为预期输出中的句子都无法追溯到检索上下文中的任何节点；没有重叠或相关信息存在。
> 
> 分数：0.5555555555555556 原因：得分为 0.56，因为虽然有几条陈述讨论了计算机视觉（例如，‘深度学习技术在图像分类、目标检测和图像分割等计算机视觉任务中的显著成就被突出显示’）方面的最新进展和深度学习进步，但大部分上下文是通用背景或无关细节（例如，‘关于“卷积”作为数学运算的解释与计算机视觉的进步没有直接相关性。’）。

如上所示，使用 LLM 作为评委的一个好处是我们能够得到关于为什么我们的分数是这样的详细反馈。例如，因为 LLM 认为一些信息是不必要的（稍微陷入关于 CNN 的兔子洞），所以得到了 55%的上下文相关性。

我们还可以使用 DeepEval 中的“evaluate”函数来更好地自动化这个过程。当在多个测试案例上测试你的 RAG 时，这很有用。

```py
#run all metrics with 'evaluate' function
evaluate(test_cases=[test_case],
         metrics=[contextual_precision, contextual_recall, contextual_relevancy])
```

### 生成评估

接下来，我们将评估我们的生成器，它根据检索器提供的上下文生成响应。在这里，我们将计算 2 个指标：

1.  **答案相关性**：类似于上下文相关性，评估你的生成器中的提示模板是否能够指导你的 LLM 根据上下文给出相关的输出。

1.  **忠实度**：评估你生成器中使用的 LLM 能否输出不虚构或与检索上下文中呈现的任何事实信息相矛盾的信息。

和之前一样，让我们初始化这些指标。

```py
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric

answer_relevancy = AnswerRelevancyMetric()
faithfulness = FaithfulnessMetric()

#compute answer relevancy and print results
answer_relevancy.measure(test_case)
print("Score: ", answer_relevancy.score)
print("Reason: ", answer_relevancy.reason)

#compute faithfulness and print results
faithfulness.measure(test_case)
print("Score: ", faithfulness.score)
print("Reason: ", faithfulness.reason)
```

> 分数：1.0 原因：得分为 1.00，因为答案完全相关，并且直接回答了问题，没有包含任何无关信息。继续保持专注和信息丰富！
> 
> 分数：1.0 原因：干得好！没有矛盾，所以实际输出完全忠实于检索上下文。

如上所示，我们的生成器在这个测试案例中运行得非常好。答案保持相关，模型没有自相矛盾。再次强调，我们可以使用“evaluate”函数来评估多个测试案例。

```py
#run all metrics with 'evaluate' function
evaluate(test_cases=[test_case],
         metrics=[answer_relevancy, faithfulness])
```

使用这些指标的一个缺点是它们是通用的，并且只针对我们生成输出的一些方面，如相关性。**但我们可以创建定制的指标来决定我们的 RAG 解决方案在我们特别关心的领域表现如何。**

例如，我们可以提出像“我的 RAG 如何处理黑色幽默？”或“输出是否以适合儿童的水平编写？”这样的问题。以我们的例子为例，让我们确定我们的 RAG 在提供技术性回答方面做得如何。

```py
from deepeval.metrics import GEval

#create evaluation for technical language
tech_eval = GEval(
    name="Technical Language",
    criteria="Determine how technically written the actual output is",
    evaluation_params=[LLMTestCaseParams.ACTUAL_OUTPUT])

#run evaluation
tech_eval.measure(test_case)
print("Score: ", tech_eval.score)
print("Reason: ", tech_eval.reason)
```

> 得分：0.6437823499114202 原因：该响应使用了适当的技术术语，如“深度学习”、“图像分类”、“目标检测”、“图像分割”、“GPU”和“FPGA”。解释清晰，但有些泛泛而谈，缺乏具体的例子或最新的突破。技术细节适中，提到了算法和硬件方面，但没有深入探讨特定的模型或方法。写作主要是正式的，并遵循技术规范，但深度和具体性可以改进。

在上面的输出中，我们可以看到我们的 RAG 产生了某些技术性的答案。它使用了适当的技术术语，并有清晰的解释，但缺乏例子。这可能是由于我们使用的是摘要作为数据源，这些摘要的写作水平相对较高。

## 人工评估

**尽管 LLM 作为裁判的方法为我们提供了大量有价值的信息，但它们是通用的，应该谨慎使用，因为它们并没有完全评估现实世界的适用性。**然而，人类可以更好地探索这一点，因为没有人比组织内的领域专家更了解数据。

人工评估通常审查正确性、论证质量和流畅度。评估者必须确定输出是否准确，逻辑上是否将检索到的证据与结论相连接，是否自然且有用。记住你的 RAG 解决方案的数据、用户和目的，这对于正确处理这些特定领域的要求非常重要。

* * *

## 结论

在这篇文章中，我们能够利用 FAISS、LangChain 等开源研究构建一个 RAG 管道。我们还深入探讨了如何评估 RAG 解决方案，评估了我们的检索器和生成器。像 DeepEval 这样的库利用 LLM 作为裁判的指标来构建测试案例，并确定相关性、忠实度等。最后，我们讨论了在确定 RAG 解决方案的现实世界适用性时，人工评估的重要性。

* * *

*希望您喜欢我的文章！请随时评论、提问或要求其他主题。*

*在 LinkedIn 上与我联系：*[*https://www.linkedin.com/in/alexdavis2020/*](https://www.linkedin.com/in/alexdavis2020/)

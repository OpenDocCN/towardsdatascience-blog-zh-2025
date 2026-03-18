# 增强 RAG：超越传统方法

> 原文：[`towardsdatascience.com/enhancing-rag-beyond-vanilla-approaches/`](https://towardsdatascience.com/enhancing-rag-beyond-vanilla-approaches/)

检索增强生成（RAG）是一种强大的技术，通过结合外部信息检索机制来增强语言模型。虽然标准的 RAG 实现提高了响应的相关性，但它们在复杂的检索场景中往往难以应对。本文探讨了传统 RAG 设置的局限性，并介绍了提高其准确性和效率的高级技术。

### **传统 RAG 的挑战**

为了说明 RAG 的限制，考虑一个简单的实验，我们尝试从一组文档中检索相关信息。我们的数据集包括：

+   一篇主要讨论保持健康、高效和良好体态的最佳实践的文档。

+   两篇关于无关主题的额外文档，但包含在不同语境中使用的某些相似词汇。

```py
main_document_text = """
Morning Routine (5:30 AM - 9:00 AM)
✅ Wake Up Early - Aim for 6-8 hours of sleep to feel well-rested.
✅ Hydrate First - Drink a glass of water to rehydrate your body.
✅ Morning Stretch or Light Exercise - Do 5-10 minutes of stretching or a short workout to activate your body.
✅ Mindfulness or Meditation - Spend 5-10 minutes practicing mindfulness or deep breathing.
✅ Healthy Breakfast - Eat a balanced meal with protein, healthy fats, and fiber.
✅ Plan Your Day - Set goals, review your schedule, and prioritize tasks.
...
"""
```

使用标准的 RAG 设置，我们用以下方式查询系统：

1.  *为了保持健康和高效，我应该做什么？*

1.  *保持健康和高效的最佳实践是什么？*

### **辅助函数**

为了提高检索准确性和简化查询处理，我们实现了一系列基本的辅助函数。这些函数服务于各种目的，从查询 ChatGPT API 到计算文档嵌入和相似度分数。通过利用这些函数，我们创建了一个更高效的 RAG 管道，能够有效地检索用户查询的最相关信息。

为了支持我们的 RAG 改进，我们定义了以下辅助函数：

```py
# **Imports**
import os
import json
import openai
import numpy as np
from scipy.spatial.distance import cosine
from google.colab import userdata

# Set up OpenAI API key
os.environ["OPENAI_API_KEY"] = userdata.get('AiTeam')
```

```py
def query_chatgpt(prompt, model="gpt-4o", response_format=openai.NOT_GIVEN):
    try:
        response = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            temperature=0.0 , # Adjust for more or less creativity
            response_format=response_format
        )
        return response.choices[0].message.content.strip()
    except Exception as e:
        return f"Error: {e}"
```

```py
def get_embedding(text, model="text-embedding-3-large"): #"text-embedding-ada-002"
    """Fetches the embedding for a given text using OpenAI's API."""
    response = client.embeddings.create(
        input=[text],
        model=model
    )
    return response.data[0].embedding
```

```py
def compute_similarity_metrics(embed1, embed2):
    """Computes different similarity/distance metrics between two embeddings."""
    cosine_sim = 1- cosine(embed1, embed2)  # Cosine similarity

    return cosine_sim
```

```py
def fetch_similar_docs(query, docs, threshold = .55, top=1):
  query_em = get_embedding(query)
  data = []
  for d in docs:
    # Compute and print similarity metrics
    similarity_results = compute_similarity_metrics(d["embedding"], query_em)
    if(similarity_results >= threshold):
      data.append({"id":d["id"], "ref_doc":d.get("ref_doc", ""), "score":similarity_results})

  # Sorting by value (second element in each tuple)
  sorted_data = sorted(data, key=lambda x: x["score"], reverse=True)  # Ascending order
  sorted_data = sorted_data[:min(top, len(sorted_data))]
  return sorted_data
```

## **评估传统 RAG**

为了评估传统 RAG 设置的有效性，我们使用预定义的查询进行简单测试。我们的目标是确定系统是否基于语义相似性检索到最相关的文档。然后，我们分析其局限性并探索可能的改进。

```py
"""# **Testing Vanilla RAG**"""

query = "what should I do to stay healthy and productive?"
r = fetch_similar_docs(query, docs)
print("query = ", query)
print("documents = ", r)

query = "what are the best practices to stay healthy and productive ?"
r = fetch_similar_docs(query, docs)
print("query = ", query)
print("documents = ", r)
```

## **改进 RAG 的高级技术**

为了进一步细化检索过程，我们引入了增强 RAG 系统功能的高级函数。这些函数生成结构化信息，有助于文档检索和查询处理，使我们的系统更加健壮和具有上下文意识。

为了解决这些挑战，我们实施了三个关键增强：

#### **1. 生成 FAQs**

通过自动创建与文档相关的常见问题列表，我们扩展了模型可以匹配的潜在查询范围。这些 FAQs 一旦生成，就会与文档一起存储，提供更丰富的搜索空间，而无需承担持续的成本。

```py
def generate_faq(text):
  prompt = f'''
  given the following text: """{text}"""
  Ask relevant simple atomic questions ONLY (don't answer them) to cover all subjects covered by the text. Return the result as a json list example [q1, q2, q3...]
  '''
  return query_chatgpt(prompt, response_format={ "type": "json_object" })
```

#### **2. 创建概述**

文档的高级总结有助于捕捉其核心思想，使检索更加有效。通过将概述嵌入到文档中，我们为相关查询提供了额外的入口点，提高了匹配率。

```py
def generate_overview(text):
  prompt = f'''
  given the following text: """{text}"""
  Generate an abstract for it that tells in maximum 3 lines what is it about and use high level terms that will capture the main points,
  Use terms and words that will be most likely used by average person.
  '''
  return query_chatgpt(prompt)
```

#### **3. 查询分解**

我们不是用宽泛的用户查询进行搜索，而是将它们分解成更小、更精确的子查询。然后，每个子查询都与我们的增强文档集合进行比较，该集合现在包括：

+   原始文档

+   生成的 FAQ

+   生成的概述

通过合并这些多个来源的检索结果，我们显著提高了找到相关信息的可能性。

```py
def decompose_query(query):
  prompt = f'''
  Given the user query: """{query}"""
break it down into smaller, relevant subqueries
that can retrieve the best information for answering the original query.
Return them as a ranked json list example [q1, q2, q3...].
'''
  return query_chatgpt(prompt, response_format={ "type": "json_object" })
```

## **评估改进后的 RAG**

实施这些技术后，我们重新运行了初始查询。这次，查询分解生成了几个子查询，每个子查询都专注于原始问题的不同方面。因此，我们的系统成功从常见问题解答（FAQ）和原始文档中检索到相关信息，这比传统的 RAG 方法有显著的改进。

```py
"""# **Testing Advanced Functions**"""

## Generate overview of the document
overview_text = generate_overview(main_document_text)
print(overview_text)
# generate embedding
docs.append({"id":"overview_text", "ref_doc": "main_document_text", "embedding":get_embedding(overview_text)})

## Generate FAQ for the document
main_doc_faq_arr = generate_faq(main_document_text)
print(main_doc_faq_arr)
faq =json.loads(main_doc_faq_arr)["questions"]

for f, i in zip(faq, range(len(faq))):
  docs.append({"id": f"main_doc_faq_{i}", "ref_doc": "main_document_text", "embedding":  get_embedding(f)})

## Decompose the 1st query
query = "what should I do to stay healty and productive?"
subqueries = decompose_query(query)
print(subqueries)

subqueries_list = json.loads(subqueries)['subqueries']

## compute the similarities between the subqueries and documents, including FAQ
for subq in subqueries_list:
  print("query = ", subq)
  r = fetch_similar_docs(subq, docs, threshold=.55, top=2)
  print(r)
  print('=================================\n')

## Decompose the 2nd query
query = "what the best practices to stay healty and productive?"
subqueries = decompose_query(query)
print(subqueries)

subqueries_list = json.loads(subqueries)['subqueries']

## compute the similarities between the subqueries and documents, including FAQ
for subq in subqueries_list:
  print("query = ", subq)
  r = fetch_similar_docs(subq, docs, threshold=.55, top=2)
  print(r)
  print('=================================\n')
```

这里是一些生成的 FAQ：

```py
{
  "questions": [
    "How many hours of sleep are recommended to feel well-rested?",
    "How long should you spend on morning stretching or light exercise?",
    "What is the recommended duration for mindfulness or meditation in the morning?",
    "What should a healthy breakfast include?",
    "What should you do to plan your day effectively?",
    "How can you minimize distractions during work?",
    "How often should you take breaks during work/study productivity time?",
    "What should a healthy lunch consist of?",
    "What activities are recommended for afternoon productivity?",
    "Why is it important to move around every hour in the afternoon?",
    "What types of physical activities are suggested for the evening routine?",
    "What should a nutritious dinner include?",
    "What activities can help you reflect and unwind in the evening?",
    "What should you do to prepare for sleep?",
    …
  ]
}
```

## **成本效益分析**

虽然这些增强引入了前期处理成本——生成 FAQ、概述和嵌入，但这是对每份文档的一次性成本。相比之下，一个优化不良的 RAG 系统会导致两个主要低效性：

1.  由于检索质量低而感到沮丧的用户。

1.  从检索过多、松散相关的文档中产生的查询成本增加。

对于处理高查询量的系统，这些低效性会迅速累积，使得预处理成为一个值得的投资。

## **结论**

通过将文档预处理（FAQ 和概述）与查询分解相结合，我们创建了一个更智能的 RAG 系统，该系统在准确性和成本效益之间取得平衡。这种方法提高了检索质量，减少了无关结果，并确保了更好的用户体验。

随着 RAG 的不断发展，这些技术将在改进 AI 驱动检索系统中发挥关键作用。未来的研究可能会探索进一步的优化，包括动态阈值和强化学习用于查询优化。

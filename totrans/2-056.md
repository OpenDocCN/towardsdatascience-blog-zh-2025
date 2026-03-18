# 揭秘余弦相似度

> 原文：[`towardsdatascience.com/demystifying-cosine-similarity/`](https://towardsdatascience.com/demystifying-cosine-similarity/)

*<mdspan datatext="el1754656867774" class="mdspan-comment">余弦相似度</mdspan>* 是自然语言处理（NLP）领域中用于操作化诸如语义搜索和文档比较等任务的常用指标。入门级的 NLP 课程通常只提供使用余弦相似度进行此类任务（而不是，比如说，欧几里得距离）的高层次理由，而不解释其背后的数学原理，这使得许多数据科学家对这一主题的理解相当模糊。为了填补这一空白，以下文章阐述了余弦相似度指标的数学直觉，并展示了如何通过 Python 中的实际示例来帮助我们解释实践中的结果。

**注意：** 下文中所有图表和公式均由本文作者创建。

## 数学直觉

余弦相似度指标基于读者可能从高中数学中回忆起的余弦函数。余弦函数表现出重复的波形模式，如图 1 所示，其完整周期在 0 <= *x* <= 2**pi* 的范围内描绘。用于生成该图的 Python 代码也包含在内以供参考。

```py
import numpy as np
import matplotlib.pyplot as plt

# Define the x range from 0 to 2*pi
x = np.linspace(0, 2 * np.pi, 500)
y = np.cos(x)

# Create the plot
plt.figure(figsize=(8, 4))
plt.plot(x, y, label='cos(x)', color='blue')

# Add notches on the x-axis at pi/2 and 3*pi/2
notch_positions = [0, np.pi/2, np.pi, 3*np.pi/2, 2*np.pi]
notch_labels = ['0', 'pi/2', 'pi', '3*pi/2', '2*pi']
plt.xticks(ticks=notch_positions, labels=notch_labels)

# Add custom horizontal gridlines only at y = -1, 0, 1
for y_val in [-1, 0, 1]:
    plt.axhline(y=y_val, color='gray', linestyle='--', linewidth=0.5)

# Add vertical gridlines at specified x-values
for x_val in notch_positions:
    plt.axvline(x=x_val, color='gray', linestyle='--', linewidth=0.5)

# Customize the plot
plt.xlabel("x")
plt.ylabel("cos(x)")

# Final layout and display
plt.tight_layout()
plt.show()
```

![](img/dfe22fd119c2f68a85f9c1c75a240470.png)

图 1：余弦函数

函数参数 *x* 表示弧度角（例如，嵌入空间中两个向量之间的角度），其中 *pi*/2、*pi*、3**pi*/2 和 2**pi* 分别对应 90、180、270 和 360 度。

要理解为什么余弦函数可以作为设计向量相似度指标的有用基础，请注意，基本余弦函数（如图 1 所示，没有进行任何功能变换）在 x = 2**a***pi* 处有极大值，在 x = (2**b* + 1)**pi* 处有极小值，在 x = (*c* + 1/2)**pi* 处有根，其中 *a*、*b* 和 *c* 是某些整数。换句话说，如果 *x* 表示两个向量之间的角度，则 *cos(x)* 在向量指向同一方向时返回最大值，在向量指向相反方向时返回最小值，在向量相互垂直时返回零。

余弦函数的这一行为巧妙地捕捉了 NLP 中的两个关键概念之间的相互作用：*语义重叠*（传达两个文本之间共享多少意义）和*语义极性*（捕捉文本中意义的对立性）。例如，文本“我喜欢这部电影”和“I enjoyed this film”具有高语义重叠（尽管使用了不同的词语，它们表达了基本相同的意义）和低语义极性（它们没有表达相反的意义）。现在，如果两个单词的嵌入向量恰好编码了语义重叠和极性，那么我们预计同义词的余弦相似度将接近 1，反义词的余弦相似度将接近-1，而无关词语的余弦相似度将接近 0。

在实践中，我们通常不会直接知道角度 *x*。相反，我们必须从向量本身推导出余弦值。给定两个向量 *U* 和 *V*，每个向量都有 *n* 个元素，这两个向量之间角度的余弦值——相当于余弦相似度度量——是通过向量的点积除以向量模的乘积来计算的：

![图片](img/ed612a52afd770c0c6d12e0d7f0555fa.png)

上述两个向量之间余弦角的公式可以从所谓的余弦定理推导出来，如本视频第 12 至 18 分钟所演示：

本视频展示了余弦定理本身的巧妙证明：

以下 Python 实现的余弦相似度明确地操作化上述公式，而不依赖于任何黑盒、第三方包：

```py
import math

def cosine_similarity(U, V):
    if len(U) != len(V):
        raise ValueError("Vectors must be of the same length.")

    # Compute dot product and magnitudes
    dot_product = sum(u * v for u, v in zip(U, V))
    magnitude_U = math.sqrt(sum(u ** 2 for u in U))
    magnitude_V = math.sqrt(sum(v ** 2 for v in V))

    # Zero vector handling to avoid division by zero
    if magnitude_U == 0 or magnitude_V == 0:
        raise ValueError("Cannot compute cosine similarity for zero-magnitude vectors.")

    return dot_product / (magnitude_U * magnitude_V)
```

感兴趣的读者可以参考[这篇文章](https://towardsdatascience.com/declarative-and-imperative-prompt-engineering-for-generative-ai/)，了解使用 NumPy 和 SciPy 包的更高效的 Python 实现，该实现用于计算余弦距离度量（定义为 1 减去余弦相似度）。

最后，值得比较余弦相似度（或距离）的数学直觉与欧几里得距离的数学直觉，欧几里得距离衡量两个向量之间的线性距离，也可以作为向量相似度度量。特别是，两个向量 *U* 和 *V*（长度均为 *n*）之间的欧几里得距离可以通过以下公式计算：

![图片](img/ab1385ec69d2de341c1b20e5af2d6146.png)

下面是相应的 Python 实现：

```py
import math

def euclidean_distance(U, V):
    if len(U) != len(V):
        raise ValueError("Vectors must be of the same length.")

    # Compute sum of squared differences
    sum_squared_diff = sum((u - v) ** 2 for u, v in zip(U, V))

    # Take the square root of the sum
    return math.sqrt(sum_squared_diff)
```

注意，由于欧几里得距离公式中的元素差异是平方的，因此得到的度量将始终是非负数——如果向量相同，则为零，否则为正。在 NLP 的背景下，这意味着欧几里得距离不会像余弦距离那样反映语义极性。此外，只要两个向量指向同一方向，它们之间角度的余弦值将保持不变，无论向量大小如何。相比之下，欧几里得距离度量受向量大小差异的影响，这可能导致实践中产生误导性的解释（例如，两个在语义上相似但长度不同的文本可能会产生较高的欧几里得距离）。因此，余弦相似度在许多 NLP 场景中是首选的度量，在这些场景中，确定向量——或语义——方向性是主要关注点。

## 理论与实践对比

在实际的 NLP 场景中，余弦相似度的解释取决于向量嵌入编码极性和语义重叠的程度。在下面的实际操作示例中，我们将使用一个未编码极性的预训练嵌入模型（[all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)）和一个编码极性的模型（[distilbert-base-uncased-finetuned-sst-2-english](https://huggingface.co/distilbert/distilbert-base-uncased-finetuned-sst-2-english)）来调查两个给定单词之间的相似度。我们还将利用 SciPy 包提供的函数来实现更高效的余弦相似度和欧几里得距离的计算。

```py
from scipy.spatial.distance import cosine as cosine_distance
from sentence_transformers import SentenceTransformer
from transformers import AutoTokenizer, AutoModel
import torch

# Words to embed
words = ["movie", "film", "good", "bad", "spoon", "car"]

# Load a pre-trained embedding model from Hugging Face
model_1 = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
model_2_name = "distilbert-base-uncased-finetuned-sst-2-english"
model_2_tokenizer = AutoTokenizer.from_pretrained(model_2_name)
model_2 = AutoModel.from_pretrained(model_2_name)

# Generate embeddings for model 1
embeddings_1 =  dict(zip(words, model_1.encode(words)))

# Generate embeddings for model 2
inputs = model_2_tokenizer(words, padding=True, truncation=True, return_tensors="pt")
with torch.no_grad():
    outputs = model_2(**inputs)
    embedding_vectors_model_2 = outputs.last_hidden_state.mean(dim=1)
embeddings_2 = {word: vector for word, vector in zip(words, embedding_vectors_model_2)}

# Compute and print cosine similarity (1 - cosine distance) for both embedding models
print("Cosine similarity for embedding model 1:")
print("movie", "\t", "film", "\t", 1 - cosine_distance(embeddings_1["movie"], embeddings_1["film"]))
print("good", "\t", "bad", "\t", 1 - cosine_distance(embeddings_1["good"], embeddings_1["bad"]))
print("spoon", "\t", "car", "\t", 1 - cosine_distance(embeddings_1["spoon"], embeddings_1["car"]))
print()

print("Cosine similarity for embedding model 2:")
print("movie", "\t", "film", "\t", 1 - cosine_distance(embeddings_2["movie"], embeddings_2["film"]))
print("good", "\t", "bad", "\t", 1 - cosine_distance(embeddings_2["good"], embeddings_2["bad"]))
print("spoon", "\t", "car", "\t", 1 - cosine_distance(embeddings_2["spoon"], embeddings_2["car"]))
print()
```

**输出：**

```py
Cosine similarity for embedding model 1:
movie 	 film 	 0.8426464702276286
good 	 bad 	 0.5871497042685934
spoon 	 car 	 0.22919675707817078

Cosine similarity for embedding model 2:
movie 	 film 	 0.9638281550070811
good 	 bad 	 -0.3416433451550165
spoon 	 car 	 0.5418748837234599
```

“电影”和“影片”这两个词通常用作同义词，它们的余弦相似度接近 1，正如预期的那样，表明高度语义重叠。而“好”和“坏”是反义词，我们可以在使用已知编码语义极性的第二个嵌入模型时看到这一点，负余弦相似度结果反映了这一点。最后，“勺子”和“汽车”这两个词在语义上不相关，它们向量嵌入的正交性可以通过它们的余弦相似度结果比“电影”和“影片”更接近零来表示。

## 总结

两个向量之间的余弦相似度基于它们形成的角度的余弦值，并且——与欧几里得距离等度量不同——对向量大小差异不敏感。从理论上讲，如果向量指向同一方向（表示高度相似），余弦相似度应接近 1；如果向量指向相反方向（表示高度不相似），余弦相似度应接近-1；如果向量正交（表示无关），余弦相似度应接近 0。然而，在给定的 NLP 场景中，余弦相似度的确切解释取决于用于将文本数据向量化所使用的嵌入模型的性质（例如，嵌入模型是否除了语义重叠之外还编码极性）。

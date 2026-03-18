# NLP 图解，第三部分：Word2Vec

> 原文：[`towardsdatascience.com/nlp-illustrated-part-3-word2vec-5b2e12b6a63b/`](https://towardsdatascience.com/nlp-illustrated-part-3-word2vec-5b2e12b6a63b/)

欢迎来到我们探索自然语言处理激动人心的世界的第三部分！如果你已经阅读了[第二部分](https://medium.com/r?url=https%3A%2F%2Ftowardsdatascience.com%2Fnlp-illustrated-part-2-word-embeddings-6d718ac40b7d)，你会记得我们讨论了词嵌入以及为什么它们如此酷。

> [**NLP 图解，第二部分：词嵌入**](https://medium.com/towards-data-science/nlp-illustrated-part-2-word-embeddings-6d718ac40b7d)

词嵌入使我们能够创建捕捉其细微差别和复杂关系的单词地图。

![图片](img/dbcd3f8b6df13307b9d8c09a450d1d9e.png)

本文将解析使用称为**Word2Vec**的技术构建词嵌入背后的数学，Word2Vec 是一种专门设计用来生成有意义的词嵌入的机器学习模型。

> Word2Vec 提供了两种方法——Skip-gram 和 CBOW——但我们将重点介绍 Skip-gram 方法的工作原理，因为它是最广泛使用的。

这些单词和概念现在可能听起来很复杂，但不用担心——其核心只是一些直观的数学（和一些机器学习魔法）。

*简短地说——在深入这篇文章之前，我强烈建议你阅读我的[机器学习基础知识系列](https://medium.com/@shreya.rao/list/machine-learning-starter-pack-b89c3a7f97ad)。一些概念（如梯度下降和损失函数**）建立在那些基础知识之上，理解它们会让这篇文章更容易理解。*

> [**机器学习入门包**](https://medium.com/@shreya.rao/list/b89c3a7f97ad)

*话虽如此，如果你对这些概念不熟悉，请不要担心——这篇文章将简要介绍它们，以确保你仍然可以跟上！*

* * *

由于 Word2Vec 是一个机器学习模型，就像任何 ML 模型一样，它需要两样东西：

+   **训练数据**：用于学习文本数据

+   **问题陈述**：模型试图回答的问题

### 训练数据

我们试图创建一个单词地图，因此我们的训练数据将是文本。让我们从这句话开始：

![图片](img/f4fcfe04366e2dcef8c49f8b2fa899b3.png)

这将是我们的玩具训练数据。当然，在现实世界中，Word2Vec 是在大量的文本语料库上训练的——想想整本书、维基百科或大量的网站集合。但现在，我们保持简单，只使用这句话，因此模型将只为这 18 个单词学习嵌入。

### 问题陈述

对于 Word2Vec 来说，核心问题很简单：***给定两个单词，确定它们是否是邻居***

![图片](img/3e41e7e1e2626db0debdf0db2a2fd6ff.png)

为了定义“邻居”，我们使用一个称为上下文窗口的东西，它指定了要考虑的两侧相邻单词的数量。

例如，如果我们想找到单词 *"happiness"* 的邻居…*

![图片](img/5112fb85d15b00de56ce89a325243af2.png)

…并将上下文窗口大小设置为 2，*"happiness"* 的邻居将是 *"can"* 和 **"***be"*。

![图片](img/31ee897e433e269b32857fe6bf8c5eb5.png)

在这里，如果我们把 *"happiness"* 和 *"can"* 输入到模型中，理想情况下我们希望它预测它们是邻居。

![图片](img/8fa6cb6022daf48332054531cdd6333b.png)

类似地，对于单词 *"darkness"*，上下文窗口为 2 时，邻居将是 *"in"* 和 *"the"*（之前），以及 *"of"* 和 *"times"*（之后）。

![图片](img/26bc40017305d309fd13f2e793c952a7.png)

如果我们将上下文窗口设置为 3，*"happiness"* 的邻居将是两侧各三个词。

![图片](img/cb73e00cac1fd163c80c0a1813e3d71c.png)

> **术语过渡**：在这里，"happiness"被称为**目标词**，而邻近的词被称为**上下文词**。

**默认情况下，Word2Vec 中的上下文窗口大小设置为 5。然而，为了简化我们的示例，我们将使用上下文窗口大小为 2。**

现在，我们需要将这个句子转换成一个整洁的小表格，就像我们处理其他机器学习问题一样，有明确定义的**输入**和**输出值**。

![图片](img/ebfba92032b6a12b4f463b1bbc8b10fd.png)

我们可以通过将**目标词**与每个**上下文词**配对作为输入来构建这个数据集...

![图片](img/50a22df1785f2d9563dc666cddc32ac2.png)

…输出将是一个标签，表示目标词和上下文词是否是邻居：

![图片](img/9c3da661e1418eb1204b865b4681b87a.png)

> 1 表示它们是邻居

但这里有一个明显的问题。我们所有的训练对都是正例（邻居），这并没有教会模型什么是**非邻居**。

进入**负采样**。

**负采样**引入了不是邻居的词对。例如，我们知道 *"happiness"* 和 *"light"* 不是邻居，因此我们将这些数据添加到我们的训练数据中，并使用标签 0 表示它们不是邻居。

![图片](img/01aa44b41a833a66cc5292af987d08ca.png)

通过添加负样本，最终数据集包含正负样本对，以便模型可以学习预测给定对是否为真实邻居。

> 通常，对于大型数据集，我们使用每对正样本 2 到 5 个负样本，对于小型数据集，最多 10 个。

我们将使用每对正样本 2 个负样本。我们的训练数据集现在看起来是这样的：

![图片](img/f895d466884b413d60b5b50f963406c3.png)

现在是时候进入有趣的部分了——机器学习的魔法。这是我们正在解决的问题：**给定一个目标词和一个上下文词，预测它们是邻居的概率。**

让我们一步一步来分析。

## 步骤 0：决定嵌入维度

我们首先决定词嵌入的大小。正如我们所学的，**更大的嵌入可以捕捉到更多的细微差别**和更丰富的关系，但代价是计算成本的增加。

**Word2Vec 的默认嵌入大小是 100 维，但为了使解释简单，我们只需使用 2 维。**

![图片](img/72f031c50dc01be0fbe02dbf9a0e2f4f.png)

这意味着每个词都将被表示为 2D 图上的一个点，如下所示**：**

![图片](img/db2f5f07f96b5151fbe17127bb96b64d.png)

## 第 1 步：初始化嵌入矩阵

接下来，我们初始化两组不同的嵌入 - **目标嵌入**和**上下文嵌入**。

![图片](img/164087596d5621d4666c855a0ed36f79.png)

并且，在训练开始时，这些嵌入是**随机初始化的**：

![图片](img/c26a735faf4b37351f77182de5028ef7.png)

**目标嵌入和上下文嵌入以不同的值随机初始化，因为它们具有不同的用途。**

+   **目标嵌入**：表示在训练中作为目标词的每个词

+   **上下文嵌入**：表示当它是一个上下文（邻近）词时的每个词

## 第 2 步：计算目标词和上下文词的相似度

在训练过程中，我们处理一个正对及其对应的负样本块。

因此，在第一次遍历中，我们只关注第一个正对及其对应的 2 个负样本。

![图片](img/a5cbd7a1f5bc71a9c9e43cce525c42da.png)

**现在我们可以通过计算两个嵌入的[点积](https://tivadardanka.com/blog/how-the-dot-product-measures-similarity)来确定两个词的相似度：目标嵌入（如果是目标词）和上下文嵌入（如果是上下文词）。**

+   较大的点积表明词更“相似”（可能是邻居）

+   较小的点积表明它们更不相似（不太可能是邻居）

*记住，在第一次遍历中，我们只计算第一个块中 3 对之间的相似度。*

让我们先计算目标词嵌入 *"happiness"* 与上下文词嵌入 *"can"* 的点积：

![图片](img/1fc075bd78325552e1c3029b8b097db5.png)

我们得到：

![图片](img/4c6d2cbcdd569ba46a7de950faee0759.png)

现在我们需要找到一种方法将这些分数转换为概率，因为我们想知道这两个词成为邻居的可能性有多大。我们可以通过将这个点积通过 sigmoid 函数来实现。

作为快速复习，sigmoid 函数将任何输入值压缩到 0 到 1 之间的范围，这使得它非常适合解释概率。如果点积很大（表示高度相似），sigmoid 输出将接近 1；如果点积很小（表示低相似度），sigmoid 输出将更接近 0。

![图片](img/aef048c301c97dcc593aeeaccdc88072.png)

因此，将点积 -0.36 通过 sigmoid 函数，我们得到：

![图片](img/bc8bd7a27d333add42f45f5247b7dc20.png)

同样，我们可以计算其他两个对的点积和相应的概率...

![图片](img/c8937171696a4a04c350b7ac97e0ed74.png)

…以得到预测 *"幸福"* 和 *"光"* 是邻居的概率…

![图片](img/ef6d2e10c1a683670ececefc437201d9.png)

…以及预测 *"幸福"* 和 *"甚至"* 是邻居的概率：

![图片](img/11384d409217cb91f20af9ae0cd7d0cb.png)

这就是计算模型预测这些 3 对邻居概率的方法。

![图片](img/5d45b617658f28961cb2a401854e6e80.png)

如我们所见，预测值相当随机且不准确，这是有道理的，因为嵌入层是用随机值初始化的。

接下来，我们进入关键步骤：**更新这些嵌入以改进预测**。

## 第 4 步：计算误差

> 注意：如果你还没有阅读关于逻辑回归的文章，阅读它可能会有所帮助，因为那里计算误差的过程与这里非常相似。但不用担心，我们也会在这里介绍基础知识。

现在我们有了我们的预测，我们需要计算“误差”值来衡量模型的预测与真实标签之间的偏差程度。为此，我们使用**对数损失函数**。

对于每一个预测，误差的计算如下：

![图片](img/9c05a56f432c95c75273d0ea7955643b.png)

块中所有预测的整体对数损失是各个预测误差的平均值：

![图片](img/d5eda421325aba48989431a9cb2bb8fd.png)

对于我们的例子，如果我们计算上面 3 对的损失，它将看起来像这样：

![图片](img/b953fc20c5b57d679918824176f6c14d.png)

评估这一点…

![图片](img/ae7d9e4cc0e52466fb20be2c7f3b8caa.png)

…我们得到 0.3。我们的目标是减少这个损失到**0**或尽可能接近**0**。损失为**0**意味着模型的预测完美匹配真实标签。

## 第 4 步：使用梯度下降更新嵌入

由于我们在之前的逻辑回归文章中已经介绍过这一点，所以这里不再深入细节。然而，**我们知道最小化损失函数的最佳方式是使用梯度下降**。

简单来说，对数损失是一个凸函数…

![图片](img/9a2110421655c99869a241421f8ab91e.png)

…并且梯度下降帮助我们找到这条曲线上的最低点——损失最小化的点。

![图片](img/0563f0dac415e8a84eb44b5f2aaf6889.png)

它通过以下方式做到这一点：

+   计算损失函数相对于嵌入的梯度（斜率），

+   通过在梯度的相反方向上稍微调整嵌入来减少损失

因此，一旦梯度下降发挥了作用，我们就会得到新的嵌入，如下所示：

![图片](img/8802105ea89eb30fbc8a9aac6b6a5c22.png)

让我们可视化这个变化。我们以我们的目标嵌入（*"幸福"*)和上下文嵌入（*"可以"，"光"*和*"甚至"*）在我们的块中开始。

![图片](img/f855453d5cd9ef9c5ef074ba8e284fa8.png)

梯度下降后，它们稍微移动了一下，如下所示：

![图片](img/627dafdd371308a0a8d4367edded6657.png)

这是这一步骤的真正魔力。我们看到自动地：

+   对于正对，目标嵌入“*幸福*”被推向与上下文嵌入“*可以*”更近的位置，其邻居

+   对于负对，目标嵌入（*“幸福”*）被调整以远离非邻近上下文嵌入“*光*”和“*甚至*”

## 步骤 5：重复步骤 2–4

现在我们只需要使用下一块的正负对来冲洗并重复步骤 2-4。

让我们看看第二个块看起来是什么样子。

![图片](img/496cecdec15a76fe8df1441ab12e9eb2.png)

对于这些值，我们通过以下方式确定模型对词语是否为邻居的预测：

（1）计算点积并通过 sigmoid 函数传递...

![图片](img/d2a49720148b178111d0430b5327cf8d.png)

（2）然后使用对数损失和梯度下降来更新这个块中词语的目标和上下文嵌入值：

![图片](img/f67c16d8b96436cad437eed542e9aa72.png)

再次，这样做会将邻近的词嵌入拉近，而不同的词嵌入会被推开更远。

大概就是这样。我们只需重复这些步骤，针对训练数据中的每个块。

> **旁注：在训练数据集的所有块中通过一次被称为一个周期。我们通常重复 5-20 个周期以获得超级稳健的训练过程。**

到我们完整训练过程的结束时，我们将得到最终的目标和嵌入，看起来可能像这样：

![图片](img/63338122dc9b65c61acd1a84ec807993.png)

如果我们去掉上下文嵌入，我们就只剩下最终的目标嵌入。

![图片](img/0cbbcb9776afbb11e9a104bfcd5a8e14.png)

**而这些最终的目标嵌入正是我们最初追求的词嵌入！！**

> 旁注：如果需要，上下文嵌入可以平均或与目标嵌入结合以创建混合表示。然而，这种情况很少见，并不是标准做法。

这是因为训练过程根据词关系来细化嵌入。相似词语（邻居）被拉近，而不同词语（非邻居）被推开。在这个过程中，它还捕捉到词语之间的更深层次关系，包括同义词、类比和细微的上下文相似性。

在这里，我们的训练数据只是一个包含 18 个单词的单句，所以嵌入可能看起来没有意义。但想象一下在庞大的语料库上训练——一整本书、一系列文章或来自网络的数十亿个句子。

就这样！这就是我们如何使用 Word2Vec 创建词嵌入，特别是 skip-gram 方法。

## Word2Vec 真实应用

现在我们已经揭开了 Word2Vec 背后的数学魔法，让我们将其变为现实，并创建我们自己的词嵌入。

### 使用预训练的词嵌入

开始的最简单和最有效的方法是使用**预训练的词嵌入**。这些嵌入已经在像 Google 新闻和维基百科这样的大规模数据集上进行了训练，因此它们非常稳健。这意味着我们不必从头开始，这样可以节省时间和计算资源。

我们利用一些预训练的 Word2Vec 嵌入，使用**Gensim**，这是一个流行的 Python 库，用于 NLP，它针对处理大规模文本处理任务进行了优化。

```py
# install gensim 
# !pip install --upgrade gensim

import gensim.downloader as api
```

让我们来看看 Gensim 中所有可用的预训练 Word2Vec 模型：

```py
available_models = api.info()['models']

print("Available pre-trained Word2Vec models in Gensim:n")
for model_name, details in available_models.items():
    if 'word2vec' in model_name.lower():  # find models with 'word2vec' in their name
        print(f"Model: {model_name}")
        print(f"  - Description: {details.get('description')}")
```

```py
Available pre-trained Word2Vec models in Gensim:

Model: word2vec-ruscorpora-300
  - Description: Word2vec Continuous Skipgram vectors trained on full Russian National Corpus (about 250M words). The model contains 185K words.
Model: word2vec-google-news-300
  - Description: Pre-trained vectors trained on a part of the Google News dataset (about 100 billion words). The model contains 300-dimensional vectors for 3 million words and phrases. The phrases were obtained using a simple data-driven approach described in 'Distributed Representations of Words and Phrases and their Compositionality' (https://code.google.com/archive/p/word2vec/).
Model: __testing_word2vec-matrix-synopsis
  - Description: [THIS IS ONLY FOR TESTING] Word vecrors of the movie matrix.
```

我们看到有两个可用的预训练模型（因为其中一个模型被标记为*测试*）。让我们来测试一下**word2vec-google-news-300**模型！

这是如何找到单词*"beautiful"*的同义词的方法：

```py
w2v_google_news.most_similar("king")
```

```py
[('gorgeous', 0.8353005051612854),
 ('lovely', 0.8106936812400818),
 ('stunningly_beautiful', 0.7329413294792175),
 ('breathtakingly_beautiful', 0.7231340408325195),
 ('wonderful', 0.6854086518287659),
 ('fabulous', 0.6700063943862915),
 ('loveliest', 0.6612576246261597),
 ('prettiest', 0.6595001816749573),
 ('beatiful', 0.6593326330184937),
 ('magnificent', 0.6591402888298035)]
```

这些都很有意义。

如果您还记得[上一篇文章](https://medium.com/towards-data-science/nlp-illustrated-part-2-word-embeddings-6d718ac40b7d)，我们看到了如何对词嵌入执行**数学运算**以获得直观的结果。最流行的例子之一是…

![](img/75c81a8d27c6f1a824ee132bd6d269cc.png)

…我们可以这样测试：

```py
# king + woman - man
w2v_google_news.most_similar_cosmul(positive=['king', 'woman'], negative=['man'])
```

结果令人印象深刻地准确！

让我们尝试另一种组合：

![](img/bcacf24521b357c571a9b2713040826c.png)

```py
# better + bad - good
w2v_google_news.most_similar_cosmul(positive=['better', 'bad'], negative=['good'])
```

```py
[('worse', 0.9141383767127991),
 ('uglier', 0.8268526792526245),
 ('sooner', 0.7980951070785522),
 ('dumber', 0.7923389077186584),
 ('harsher', 0.791556715965271),
 ('stupider', 0.7884790301322937),
 ('scarier', 0.7865160703659058),
 ('angrier', 0.7857241034507751),
 ('differently', 0.7801468372344971),
 ('sorrier', 0.7758733034133911)]
```

而*"worse"*是最佳匹配！非常酷。

如我们所见，这些预训练模型非常稳健，可以用于大多数用例。然而，它们并不适合每种情况。例如，如果我们处理的是像法律或医学文本这样的利基领域，通用嵌入可能无法捕捉语言的具体含义和细微差别。

假设我们有一个法律文本：

> "上诉人根据第 57 条规则寻求宣告性救济，声称被告违反了 1934 年证券交易法第 10(b)条，未披露重要事实的受托人义务。"

法律文件通常以正式、高度结构化的风格编写，使用诸如**"Rule 57"**或**"Section 10(b)"**之类的术语引用特定的法律和法规。像**"material facts"**这样的词有精确的法律含义——可以影响案件结果的事实——这与日常语言中对"material"的理解非常不同。

在通用语料库上训练的预训练嵌入，如**Google 新闻**，无法捕捉这些细微的、特定领域的含义。相反，对于这类任务，我们需要在**特定领域语料库**上训练的嵌入，如法律判决、法规或合同。

### 从零开始编写我们自己的 Word2Vec

这就是构建我们自己的 Word2Vec 模型有所帮助的地方。通过在法律语料库上训练，我们可以创建针对我们用例的嵌入，捕捉法律领域的特定关系和含义。

* * *

就这样，我们就完成了！你现在已经知道了关于 Word2Vec 你需要知道的一切。

> *就像往常一样，您随时可以通过[LinkedIn](https://www.linkedin.com/in/shreyarao24/)与我联系，或者通过以下邮箱地址发送邮件给我* [[email protected]](/cdn-cgi/l/email-protection)*!*
> 
> 除非特别说明，所有图片均为作者所有。

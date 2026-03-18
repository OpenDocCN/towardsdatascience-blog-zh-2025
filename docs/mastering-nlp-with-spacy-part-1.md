# 掌握 spaCY 自然语言处理（Part 1）

> 原文：[`towardsdatascience.com/mastering-nlp-with-spacy-part-1/`](https://towardsdatascience.com/mastering-nlp-with-spacy-part-1/)

## <mdspan datatext="el1753726700659" class="mdspan-comment">介绍</mdspan>

自然语言处理（NLP）是人工智能的一部分，专注于理解文本。它是帮助机器阅读、处理并从文本中找到有用的模式或信息，以供我们的应用程序使用。[SpaCy](https://spacy.io/)是一个库，使这项工作变得更容易、更快。

许多开发者今天使用像 ChatGPT 或 Llama 这样的大型模型来完成大多数 NLP 任务。这些模型功能强大，可以做很多事情，但它们通常成本高昂且速度较慢。在现实世界的项目中，我们需要更专注且快速的东西。这正是 spaCy 大显身手的地方。

现在，spaCy 甚至通过`spacy-llm`模块让您能够结合其与大型模型（如 ChatGPT）的优势。这是一种获得速度和力量的好方法。

### 安装 Spacy

将以下命令复制并粘贴以使用 pip 安装 spaCy。

在以下单元格中，将“**&ndash**”替换为“-”。

```py
python &ndashm venv. env
source .env/bin/activate
pip install &ndashU pip setuptools wheel
pip install &ndashU spacy
```

SpaCy 没有附带统计语言模型，这是在特定语言上执行操作所需的。对于每种语言，都有许多基于构建模型所使用的资源大小而构建的模型。

支持的所有语言都列在这里：[`spacy.io/usage/models`](https://spacy.io/usage/models)

您可以通过命令行下载语言模型。在这个例子中，我正在下载英语语言模型。

```py
python &ndashm spacy download en_core_web_sm
```

到目前为止，您已经准备好使用 load()功能使用该模型。

```py
import spacy

nlp = spacy.load("en_core_web_sm")
doc = nlp("This is a text example I want to analyze")
```

### SpaCy 管道

当你在 spaCy 中加载一个语言模型时，它会通过一个您可以定制的管道处理您的文本。这个管道由各种**组件**组成，每个组件处理一个特定的任务。其核心是分词器，它将文本分解成单个标记（单词、标点等）。

这个管道的结果是一个`Doc`对象，它是进一步分析的基础。根据您想要实现的目标，可以包含其他组件，如 Tagger（用于词性标注）、Parser（用于依存句法分析）和 NER（用于命名实体识别）。我们将在接下来的文章中看到 Tagger、Parser 和 NER 的含义。

![图片](img/5612f1db461c74f4e770fc646a16b3b5.png)

管道（图片由作者提供）

为了创建一个 doc 对象，您可以简单地执行以下操作。

```py
import spacy
```

```py
nlp = spacy.load("en_core_web_md")
doc = nlp("My name is Marcello")
```

我们将熟悉 spaCy 提供的更多容器对象。

> spaCy 的核心数据结构是 Language 类、Vocab 和 Doc 对象。

通过检查文档，您将找到容器对象的完整列表。

![图片](img/b5c820a4a34f916cb0c1249525ccde8d.png)

来自 spaCy 文档

### 使用 spaCy 进行分词

在 NLP 中，处理文本的第一步是分词。这是至关重要的，因为所有后续的 NLP 任务都依赖于对分词的处理。分词是句子可以分解成的最小有意义的文本单元。直观地，您可能会认为分词是由空格分隔的单独单词，但这并不简单。

分词通常依赖于统计模式，其中经常一起出现的字符组被视为单个分词以进行更好的分析。

您可以在 hugging face space 上尝试不同的分词器：[`huggingface.co/spaces/Xenova/the-tokenizer-playground`](https://huggingface.co/spaces/Xenova/the-tokenizer-playground)

当我们在 spaCy 中对某些文本应用 nlp()时，文本会自动分词。让我们看看一个例子。

```py
doc = nlp("My name is Marcello Politi")
for token in doc:
  print(token.text)
```

![图片](img/615c8fb293db717489764afceaf00288.png)

图片由作者提供

从例子来看，这似乎是一个简单的使用 text.split(“”)进行的分割。那么让我们尝试对更复杂的句子进行分词。

```py
doc = nlp("I don't like cooking, I prefer eating!!!")
for i, token in enumerate(doc):
  print(f"Token {i}:",token.text)
```

![图片](img/fa3f01a30d56ad744716810b2fa089a2.png)

图片由作者提供

SpaCy 的分词器是基于规则的，这意味着它使用语言规则和模式来确定如何分割文本。它不基于像现代 LLMs 那样的统计方法。

有趣的是，规则是可定制的；这给了您对分词过程的完全控制。

此外，spaCy 分词器是非破坏性的，这意味着您可以从分词中恢复原始文本。

让我们看看如何**自定义分词器**。为了实现这一点，我们只需要为我们的分词器定义一条新规则，我们可以通过使用特殊的 ORTH 符号来完成。

```py
import spacy
from spacy.symbols import ORTH

nlp = spacy.load("en_core_web_sm")
doc = nlp("Marcello Politi")

for i, token in enumerate(doc):
  print(f"Token {i}:",token.text)
```

![图片](img/010eb05ffaf4d7016aa9b41bd9060683.png)

图片由作者提供

我想以不同的方式分词单词“Marcello”。

```py
special_case = [{ORTH:"Marce"},{ORTH:"llo"}]
nlp.tokenizer.add_special_case("Marcello", special_case)
doc = nlp("Marcello Politi")

for i, token in enumerate(doc):
  print(f"Token {i}:",token.text)
```

![图片](img/c0388c9e62d6bea5a86378e4db3f8457.png)

图片由作者提供

在大多数情况下，默认的分词器表现良好，很少有人需要修改它，通常只有研究人员会这样做。

将文本分割成分词比将段落分割成句子更容易。SpaCy 能够通过使用依赖分析器来完成这项任务；您可以在文档中了解更多关于它的信息。但让我们看看它在实际中是如何工作的。

```py
import spacy
nlp = spacy.load("en_core_web_sm")

text = "My name is Marcello Politi. I like playing basketball a lot!"
doc = nlp(text)

for i, sent in enumerate(doc.sents):
  print(f"sentence {i}:", sent.text)
```

### spaCy 的词元化

单词/分词可以有不同的形式。词元是单词的基本形式。例如，“dance”是“dancing”、“danced”、“dancer”、“dances”等单词的词元。

当我们将单词还原到其基本形式时，我们正在应用词元化。

![图片](img/4ba8b1a685436d40e1a4b64ddbe41328.png)

词元化（图片由作者提供）

在 SpaCy 中，我们可以轻松访问单词的词元。查看以下代码。

```py
import spacy

nlp = spacy.load("en_core_web_sm")
doc = nlp("I like dancing a lot, and then I love eating pasta!")
for token in doc:
    print("Text :", token.text, "--> Lemma :", token.lemma_)
```

![图片](img/419fffe5d2772885b1c4b31585cf0b15.png)

图片由作者提供

### 最后的想法

总结本篇 spaCy 系列的第一个部分，我分享了让我对这个 NLP 工具着迷的基础知识。

我们介绍了 spaCy 的设置、加载语言模型以及深入挖掘分词和词元化，这些是使文本处理感觉不像黑盒的主要步骤。

与那些如 ChatGPT 这样的大型模型相比，spaCy 的轻量级和快速方法非常适合许多项目，尤其是在需要额外功能时，还可以通过 spacy-llm 选项使用那些大型模型！

在下一部分，我将向您展示我是如何使用 spaCy 的命名实体识别和依存句法分析来处理现实世界的文本任务的。请继续关注第二部分，它将更加注重实践操作！

[领英](https://www.linkedin.com/in/marcello-politi/) ️|  [X (推特)](https://x.com/Marcello_AI) |  [网站](https://marcello-politi.super.site/)

### 资源

+   [`spacy.io/api`](https://spacy.io/api)

+   [`github.com/PacktPublishing/Mastering-spaCy-Second-Edition?tab=readme-ov-file`](https://github.com/PacktPublishing/Mastering-spaCy-Second-Edition?tab=readme-ov-file)

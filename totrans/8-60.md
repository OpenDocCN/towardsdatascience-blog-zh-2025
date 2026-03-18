# 认识 GPT，仅解码器的 Transformer

> 原文：[`towardsdatascience.com/meet-gpt-the-decoder-only-transformer-12f4a7918b36/`](https://towardsdatascience.com/meet-gpt-the-decoder-only-transformer-12f4a7918b36/)

![Emiliano Vittoriosi 在 Unsplash 上的照片](img/60394d79467104a406bb237bb9650a27.png)

由[Emiliano Vittoriosi](https://unsplash.com/@emilianovittoriosi?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)拍摄的照片

## 简介

大型语言模型（LLMs），如 ChatGPT、Gemini、Claude 等，已经存在一段时间了，我相信我们中的每个人都至少使用过其中之一。在撰写本文时，ChatGPT 已经实现了基于 GPT 的第四代模型，名为 GPT-4。但你是否知道 GPT 实际上是什么，其背后的神经网络架构是什么样的？在本文中，我们将讨论 GPT 模型，特别是 GPT-1、GPT-2 和 GPT-3。我还会演示如何使用 PyTorch 从头开始编写它们，以便你能更好地理解这些模型的结构。

### GPT 的简要历史

在我们深入探讨 GPT 之前，我们需要事先了解原始的 Transformer 架构。一般来说，一个 Transformer 由两个主要组件组成：*编码器*和*解码器*。前者负责理解输入序列，而后者用于根据输入生成另一个序列。例如，在问答任务中，解码器将生成对输入序列的答案，而在机器翻译任务中，它用于生成输入的翻译。

![图 1. Transformer 模型。左侧的块是编码器，右侧的块是解码器[1]。](../Images/4eca89483cba810090b788ea911bb2e3.png)

图 1. Transformer 模型。左侧的块是编码器，右侧的块是解码器[1]。

上文中提到的 Transformer 的两个主要组件也由几个子组件组成，例如*注意力块*、*前向掩码*和*层归一化*。这里我假设你已经对这些有基本了解。如果你还没有，我强烈推荐你阅读我之前关于这个主题的文章，你可以通过文章末尾提供的链接访问它[2]。

已经证明，Transformer 在语言建模方面表现出令人印象深刻的能力。有趣的是，未来的研究人员发现，其编码器和解码器部分可以单独工作来完成这项任务。这实际上就是 BERT（*双向 Transformer 编码器表示*）和 GPT（*生成预训练 Transformer*）被发明的时候，BERT 基本上只是一个编码器的堆叠，而 GPT 是一个解码器的堆叠。

更具体地谈谈 GPT，其第一个版本（GPT-1）是由 OpenAI 在 2018 年发布的。随后在 2019 年和 2020 年分别发布了 GPT-2 和 GPT-3。然而，当时知道 GPT 的人并不多，因为它只能通过 API 使用。直到 2022 年，OpenAI 发布了带有 GPT-3.5 后端的 ChatGPT，这使得公众可以轻松地与这个 LLM 交互。下面是一个展示 GPT 模型演变的图。

![图 2. GPT 模型随时间演变的历程[3].](../Images/2aa937d52bdf5c87114826bca84867bd.png)

图 2. GPT 模型随时间演变的历程[3]。

* * *

## GPT-1

第一个 GPT 版本是在 2018 年由 Radford 等人发表在题为"*通过生成预训练改进语言理解*"的研究论文[4]中。之前我提到过，GPT 基本上只是一个解码器的堆叠，在 GPT-1 的情况下，解码器块重复了 12 次。重要的是要记住，GPT-1 中实现的解码器架构与原始 Transformer 中的架构并不完全相同。在下面的图中，左侧的模型是 GPT-1 论文中提出的解码器，而右侧的是原始 Transformer 的解码器部分。我们可以看到，在原始解码器中用红色突出显示的部分在 GPT-1 中不存在。这基本上是因为这个组件被用来结合来自编码器和来自解码器输入本身的信息。在 GPT-1 的情况下，由于我们没有编码器部分，所以我们只需省略它。

![图 3. GPT-1 架构（左侧）[4]和原始 Transformer 架构的解码器部分[5].](../Images/be95068022c2ec5e6e30101b175864de.png)

图 3. GPT-1 架构（左侧）[4]和原始 Transformer 架构的解码器部分[5]。

### GPT-1 预训练

GPT-1 模型的训练过程分为两个步骤：*预训练*和*微调*。预训练的目标是教会模型根据前面的标记预测序列中的下一个标记——这个过程通常被称为*语言模型*。这个预训练步骤使用的是自监督机制，即标签来自数据集本身的训练过程。使用这种方法，我们不需要进行人工标记。相反，我们只需从长文本中随机位置抽取 513 个标记，将前 512 个作为特征，最后一个作为标签。这个标记数量是基于 GPT-1 的*上下文窗口*参数选择的，默认设置为 512。除了标记化机制外，GPT-1 还使用 BPE（*字节对编码*）。这本质上意味着每个标记不一定对应一个单词。相反，它也可以是一个子词，甚至是一个单独的字母。

GPT-2 的预训练使用的是下面图 4 中所示的目标函数，其中 *uᵢ* 是被预测的标记，*uᵢ₋ₖ, …, uᵢ₋₁* 是 *k* 个之前的标记（*上下文窗口*），而 *Θ* 是模型参数。这个方程本质上所做的就是计算在序列中给定之前的标记发生某个标记的可能性。概率最高的标记将被作为预测输出返回。通过迭代这个过程，模型将继续在提示中提供的文本。如果我们回到图 3，我们会看到 GPT-1 模型有两个头部：*文本预测*和*任务分类器*。稍后，这个文本生成过程将使用*文本预测*头部来完成。

![图 4. 预训练的目标函数[4]。](../Images/fd0d465c3ade8257893fafdc8d22204a.png)

图 4. 预训练的目标函数[4]。

### GPT-1 微调

尽管默认情况下 GPT 是一个*生成模型*，但在微调阶段我们将其视为*判别模型*。这主要是因为在这个阶段的目标仅仅是执行一个典型的分类任务。在下面的目标函数中，*y*代表要预测的类别，而 *x¹, …, xᵐ* 表示序列 *x* 中的 *m* 个输入标记。我们可以简单地将这个方程理解为将文本分类到特定的类别。这种分类机制将后来用于执行各种*下游任务*，我很快就会解释。

![图 5. 下游分类任务的目标函数[4]。](../Images/03529a65cff048c71b5e649d0943a6f1.png)

图 5. 下游分类任务的目标函数[4]。

论文中实验了四种不同的下游任务：*分类*、*自然语言推理（蕴涵）*、*句子相似度*和*多项选择题回答*。下面的图示展示了这些任务的工作流程。

![图 6. GPT-1 模型的下游任务工作流程[4]。](../Images/0c0551b2345bc700e46c5b6c29cc3c62.png)

图 6. GPT-1 模型的下游任务工作流程[4]。

用绿色着色的 Transformer 块是 GPT-1 模型，每个模型都具有完全相同的架构。为了使模型能够执行不同的任务，我们需要相应地安排输入文本。对于标准的**文本分类任务**，例如情感分析或文档分类，我们可以在将其输入 GPT-1 模型之前，将标记序列放在*开始*和*提取*标记之间来标记文本的开始和结束。然后，得到的张量将被转发到一个线性层，该层的每个神经元对应一个类别。

![图 7. 情感分析（分类）任务的输入文本及其对应标签的示例[3]。](../Images/3eecd53dff750dcd88bc62b5e8de9358.png)

图 7. 情感分析（分类）任务输入文本及其对应标签的示例 [3]。

对于**文本蕴涵**，模型接受*前提*和*假设*作为一个单一序列，由*分隔符*标记分隔。在这种情况下，*任务分类器*头负责分类*假设*是否蕴涵*前提*。

![图 8. 文本蕴涵任务输入文本及其对应标签的示例 [3]。](../Images/2aca08fed31515123029efd1cb7e8924.png)

图 8. 文本蕴涵任务输入文本及其对应标签的示例 [3]。

在**文本相似度任务**中，模型通过接受两种不同顺序的两个待比较文本来工作：首先是*文本 1*，然后是*文本 2*，以及首先是*文本 2*，然后是*文本 1*。这两个序列并行输入到 GPT 模型中，然后对生成的输出进行求和，最终预测它们是否相似。或者，我们也可以配置输出层执行回归任务，返回一个连续的相似度分数。

![图 9. 文本相似度测量数据集的一个示例 [3]。](../Images/42ba605a0d2b59664c9d8d5d617d1372.png)

图 9. 文本相似度测量数据集的一个示例 [3]。

最后，对于**多选题回答**，我们将包含事实的文本和相应的问句都包裹在*上下文*块中。接下来，我们在添加其中一个答案之前在其前面放置一个*分隔符*标记。对于每个问题的所有可能的答案，我们都要这样做。使用这种数据集结构，我们通过将它们传递给模型来进行推理，让模型计算每个问题-答案对的相似度分数。这个分数表示每个答案根据给定的事实如何很好地回答问题。我们基本上可以将其视为一个标准的分类任务，其中选定的答案是具有最高相似度分数的那个答案。

![图 10. 多选题回答任务数据集的一个示例 [3]。](../Images/df87cd9091d9c82087d50fdba76a0948.png)

图 10. 多选题回答任务数据集的一个示例 [3]。

在微调阶段，我们并不完全忽略语言建模过程，因为它仍然提供了一些关于下一个应该出现什么标记的想法。换句话说，我们可以将其视为一个*辅助目标*，这对于加速收敛同时提高分类器模型的可推广性是有用的。因此，下游任务目标函数（L2）需要与语言建模目标函数（L1）相结合。下面的图 11 展示了它在形式数学定义中的表达，其中权重 *λ* 通常设置小于 1，允许模型更多地关注下游任务。

![图 11. 用于微调的目标函数 [4]。](../Images/9a1b2bd32068ed8e663fd01308771e76.png)

图 11.用于微调的[4]目标函数。

因此，总结一下，GPT-1 的基本工作原理是继续前面的序列。如果我们不进一步微调模型，它将根据自我监督训练阶段提供的数据理解来继续序列。同时，如果我们进行微调，模型也将继续序列，但仅使用监督学习阶段提供的特定真实值。

### GPT-1 实现：前瞻掩码与位置编码

既然我们已经了解了 GPT-1 背后的理论，那么现在让我们从头开始实现其架构设计！我们将首先导入所需的模块。

```py
# Codeblock 1
import torch
import torch.nn as nn
```

之后，我们将继续进行参数配置，您可以在下面的代码块 2 中看到。我们在这里设置的变量与 GPT-1 论文中指定的变量完全相同，除了`BATCH_SIZE`和`N_CLASS`（分别在第`#(1)`和`#(2)`行标记）。`BATCH_SIZE`变量是必要的，因为 PyTorch 默认情况下会以批处理方式处理张量，无论其中包含的样本数量是多少。在这种情况下，我假设每个批次中只有一个样本。同时，`N_CLASS`将用于在执行下游任务时运行的*任务分类器*头部。例如，这里我将参数设置为 3。使用这种配置，我们可以使用头部进行 3 类分类任务，如我之前在图 7 和 8 中向您展示的情感分析或文本蕴涵案例。

```py
# Codeblock 2
BATCH_SIZE = 1          #(1)
N_CLASS    = 3          #(2)
SEQ_LENGTH = 512        #(3)
VOCAB_SIZE = 40000      #(4)

D_MODEL    = 768        #(5)
N_LAYERS   = 12         #(6)
NUM_HEADS  = 12         #(7)
HIDDEN_DIM = D_MODEL*4  #(8)
DROP_PROB  = 0.1        #(9)
```

`SEQ_LENGTH`参数（`#(3)`），这是表示*上下文窗口*的另一个术语，设置为 512。在训练数据集上执行的 BPE 标记化机制产生了 40,000 个独特的标记，因此我们需要使用这个数字来设置`VOCAB_SIZE`（`#(4)`）。接下来，`D_MODEL`参数表示用于表示标记的特征向量长度，在 GPT-1 的情况下，这个值设置为 768（`#(5)`）。之前我提到解码器层重复了 12 次。在上面的代码中，这个数字被分配给`N_LAYERS`变量（`#(6)`）。解码器层本身还包含一些其他组件，这些参数需要手动配置。这些参数包括*注意力头*的数量（`#(7)`）、*前馈*块中的隐藏神经元数量（`#(8)`）以及*dropout*层的比率（`#(9)`）。

在配置了所需的参数之后，接下来要做的事情是初始化一个创建所谓的 *前瞻* 掩码的函数和一个创建 *位置嵌入* 的类。*前瞻* 掩码可以被认为是一种工具，它防止模型在训练阶段查看后续的标记，考虑到在推理阶段后续标记是不可用的。同时，*位置嵌入* 用于给每个标记分配特定的数字，这对于保留有关标记顺序的信息很有用。实际上，尽管 *前瞻* 掩码已经包含了这些信息，但 *位置嵌入* 进一步强调了这一点。

请查看下面的代码块 3 和 4，以了解我如何实现刚才解释的两个概念。我不会深入探讨它们，因为我已经在关于 Transformer 的文章中提供了完整的解释，该文章的链接在参考文献列表 [2] 中提供——您可以点击它并滚动到底部的 *位置编码* 和 *前瞻掩码* 部分。甚至下面的代码也与我那里写的完全一样！

```py
# Codeblock 3
def create_mask():
    mask = torch.tril(torch.ones((SEQ_LENGTH, SEQ_LENGTH)))
    mask[mask == 0] = -float('inf')
    mask[mask == 1] = 0
    return mask
```

```py
# Codeblock 4
class PositionalEncoding(nn.Module):
    def forward(self):
        pos = torch.arange(SEQ_LENGTH).reshape(SEQ_LENGTH, 1)
        i = torch.arange(0, D_MODEL, 2)
        denominator = torch.pow(10000, i/D_MODEL)

        even_pos_embed = torch.sin(pos/denominator)
        odd_pos_embed  = torch.cos(pos/denominator)

        stacked = torch.stack([even_pos_embed, odd_pos_embed], dim=2)
        pos_embed = torch.flatten(stacked, start_dim=1, end_dim=2)

        return pos_embed
```

### GPT-1 实现：解码器

现在我们来谈谈我在 `DecoderGPT1()` 类中实现的解码器部分。我之所以这样命名，是因为我们将专门使用它来处理 GPT-1。请参阅代码块 5a 和 5b 中的详细实现。

```py
# Codeblock 5a
class DecoderGPT1(nn.Module):
    def __init__(self):
        super().__init__()

        self.multihead_attention = nn.MultiheadAttention(embed_dim=D_MODEL,  #(1)
                                                         num_heads=NUM_HEADS, 
                                                         batch_first=True)  #(2)
        self.dropout_0 = nn.Dropout(DROP_PROB)
        self.norm_0 = nn.LayerNorm(D_MODEL)  #(3)

        self.feed_forward = nn.Sequential(nn.Linear(D_MODEL, HIDDEN_DIM),  #(4) 
                                          nn.GELU(), 
                                          nn.Linear(HIDDEN_DIM, D_MODEL))
        self.dropout_1 = nn.Dropout(DROP_PROB)
        self.norm_1 = nn.LayerNorm(D_MODEL)  #(5)

        nn.init.normal_(self.feed_forward[0].weight, 0, 0.02)  #(6)
        nn.init.normal_(self.feed_forward[2].weight, 0, 0.02)  #(7)
```

在上面提到的 `__init__()` 方法中，我初始化了几个神经网络层，其中每一个都对应于图 3 中显示的解码器内部的每个子组件。第一个是 *多头注意力* 层 (`#(1)`), 其中用于 `embed_dim` 和 `num_heads` 的值取自我们之前初始化的变量。此外，在这里我将 `batch_first` 参数设置为 `True` (`#(2)`)，因为我们的批处理维度位于 0 轴上，这在使用 PyTorch 张量时是一种常见做法。接下来，我们在 `#(3)` 和 `#(5)` 行初始化了两个带有 `D_MODEL` 作为每个输入参数的 *层归一化* 层。这实际上意味着这两个层将对每个标记的 768 个值执行归一化。

对于 *前馈* 块，我使用 `nn.Sequential()` (`#(4)`) 创建它，其中我初始化了两个线性层和中间的 GELU 激活函数。第一个线性层负责将 768 (`D_MODEL`) 维度的标记表示扩展到 3072 (`HIDDEN_DIM`) 维度。然后我们通过 GELU 传递它，然后再将其缩小回 768 维度。本文的作者提到，这些层的权重初始化遵循均值为 0、标准差为 0.02 的正态分布。我们可以使用 `#(6)` 和 `#(7)` 行的代码手动配置它们。

现在让我们继续到代码块 5b，在那里我定义了 `DecoderGPT1()` 类的 `forward()` 方法。您可以看到，它通过接受两个输入来工作：`x` 和 `attn_mask` (`#(1)`)。第一个输入是嵌入的标记序列，而第二个输入是由我们之前定义的 `create_mask()` 函数生成的 *look-ahead* 掩码。

```py
# Codeblock 5b
    def forward(self, x, attn_mask):  #(1)
        residual = x  #(2)
        print(f"original &amp; residualt: {x.shape}")

        x = self.multihead_attention(x, x, x, attn_mask=attn_mask)[0]  #(3)
        print(f"after attentiontt: {x.shape}")

        x = self.dropout_0(x)  #(4)
        print(f"after dropouttt: {x.shape}")

        x = x + residual  #(5)
        print(f"after additiontt: {x.shape}")

        x = self.norm_0(x)  #(6)
        print(f"after normalizationt: {x.shape}")

        residual = x
        print(f"nx &amp; residualtt: {x.shape}")

        x = self.feed_forward(x)  #(7)
        print(f"after feed forwardt: {x.shape}")

        x = self.dropout_1(x)
        print(f"after dropouttt: {x.shape}")

        x = x + residual
        print(f"after additiontt: {x.shape}")

        x = self.norm_1(x)
        print(f"after normalizationt: {x.shape}")

        return x
```

在进行任何操作之前，我们在上面的 `forward()` 方法中做的第一件事是将原始输入张量 `x` 存储到 `residual` 变量中 (`#(2)`)。然后，`x` 张量本身通过 *multihead attention* 层进行处理 (`#(3)`)。由于我们即将执行 *self* *attention*（而不是 *cross* *attention*），因此该层的 *query*、*key* 和 *value* 输入都来自 `x`。不仅如此，这里我们还需要将 *look-ahead* 掩码作为 `attn_mask` 参数的参数传递。在通过 *attention* 层处理完成后，我们将 `x` 张量通过一个 *dropout* 层 (`#(4)`)，然后再将其与 `residual` 结合 (`#(5)`)，并通过 *layer norm* 进行归一化 (`#(6)`)。剩余的过程几乎相同，只是我们将 `self.multihead_attention` 层替换为 `self.feed_forward` 层 (`#(7)`)。

为了检查我们的解码器是否正常工作，我们可以传递一个大小为 1×512×768 的张量，如下面的代码块 6 所示。这模拟了一个由 512 个标记组成的序列，每个标记表示为一个 768 维的向量。

```py
# Codeblock 6
decoder_gpt_1 = DecoderGPT1()
x = torch.randn(BATCH_SIZE, SEQ_LENGTH, D_MODEL)
look_ahead_mask = create_mask()

x = decoder_gpt_1(x, look_ahead_mask)
```

我们可以在生成的输出中看到，这个张量成功通过了解码器中的所有组件。值得注意的是，在每个处理过程中，包括最终输出，张量的维度都保持不变。这一特性允许我们堆叠多个解码器，而不用担心张量维度会破坏。-- 嗯，实际上，在 *attention* 和 *feed forward* 层内部确实有一些维度变化，但在被馈送到后续层之前，它会立即回到原始维度。

```py
# Codeblock 6 output
original &amp; residual   : torch.Size([1, 512, 768])
after attention       : torch.Size([1, 512, 768])
after dropout         : torch.Size([1, 512, 768])
after addition        : torch.Size([1, 512, 768])
after normalization   : torch.Size([1, 512, 768])

x &amp; residual          : torch.Size([1, 512, 768])
after feed forward    : torch.Size([1, 512, 768])
after dropout         : torch.Size([1, 512, 768])
after addition        : torch.Size([1, 512, 768])
after normalization   : torch.Size([1, 512, 768])
```

### GPT-1 实现：带有输入和文本预测的解码器

我们已经完成了解码器块，现在我们将连接输入层，并将 *text prediction* 头附加到输出上。您可以在下面的 `GPT1()` 类中看到我是如何实现它们的。

```py
# Codeblock 7a
class GPT1(nn.Module):
    def __init__(self):
        super().__init__()

        self.token_embedding = nn.Embedding(num_embeddings=VOCAB_SIZE, 
                                            embedding_dim=D_MODEL)  #(1)

        self.positional_encoding = PositionalEncoding()  #(2)

        self.decoders = nn.ModuleList([DecoderGPT1() for _ in range(N_LAYERS)])  #(3)

        self.linear = nn.Linear(in_features=D_MODEL, out_features=VOCAB_SIZE)  #(4)

        nn.init.normal_(self.token_embedding.weight, mean=0, std=0.02)  #(5)
        nn.init.normal_(self.linear.weight, mean=0, std=0.02)  #(6)
```

在 `__init__()` 方法内部，我们首先初始化一个 `nn.Embedding()` 层。这个层用于将每个标记映射到 768 维（`D_MODEL`）向量（`#(1)`）。其次，我们使用我们之前创建的 `PositionalEncoding()` 类初始化一个 *位置编码* 张量（`#(2)`）。12 个解码器层需要逐个初始化，在这种情况下，我使用一个简单的 `for` 循环来完成。所有这些解码器随后被存储在 `self.decoders`（`#(3)`）中。接下来，我们初始化一个线性层，这基本上对应于 *文本预测* 头（`#(4)`）。这个层负责将每个向量映射到 `VOCAB_SIZE`（40,000）个神经元，其中每个神经元都表示选择特定标记的概率。同样，在这里我也手动配置了权重初始化分布，使用第 `#(5)` 行和 `#(6)` 的代码。

接下来，我们来看代码块 7b 中的 `forward()` 方法，我们首先使用 `self.token_embedding` 层（`#(1)`）处理输入张量。然后，我们通过逐元素相加（`#(2)`）将位置编码张量注入到 `x` 中。得到的张量随后被转发到由 12 个解码器组成的堆栈中，这可以通过另一个循环实现，如第 `#(3)` 行所示。记住，GPT-1 模型有两个头。在这种情况下，*文本预测* 头将包含在 `forward()` 方法中，而 *任务分类器* 头将稍后在单独的类中实现。为了完成这个任务，我将返回原始解码器输出（`decoder_output`）以及下一词预测输出（`text_output`），如第 `#(5)` 行所示。稍后，我将使用 `decoder_output` 作为 *任务分类器* 头的输入。

```py
# Codeblock 7b
    def forward(self, x):
        print(f"original inputtt: {x.shape}")

        x = self.token_embedding(x.long())  #(1)
        print(f"embedded tokenstt: {x.shape}")

        x = x + self.positional_encoding()  #(2)
        print(f"after additiontt: {x.shape}")

        for i, decoder in enumerate(self.decoders):
            x = decoder(x, attn_mask=look_ahead_mask)  #(3)
            print(f"after decoder #{i}t: {x.shape}")

        decoder_output = x  #(4)
        print(f"decoder_outputtt: {decoder_output.shape}")

        text_output = self.linear(x)
        print(f"text_outputtt: {text_output.shape}")

        return decoder_output, text_output  #(5)
```

我们可以通过下面的代码块 8 检查我们的 `GPT1()` 类是否正常工作。这里的 `x` 张量被假定为长度为 `SEQ_LENGTH`（512）的标记序列，其中每个元素都是介于 0 到 `VOCAB_SIZE`（40,000）之间的随机整数，代表编码的标记。

```py
# Codeblock 8
gpt1 = GPT1()

x = torch.randint(0, VOCAB_SIZE, (BATCH_SIZE, SEQ_LENGTH))
x = gpt1(x)
```

```py
# Codeblock 8 output
original input     : torch.Size([1, 512])  #(1)
embedded tokens    : torch.Size([1, 512, 768])  #(2)
after addition     : torch.Size([1, 512, 768])
after decoder #0   : torch.Size([1, 512, 768])
after decoder #1   : torch.Size([1, 512, 768])
after decoder #2   : torch.Size([1, 512, 768])
after decoder #3   : torch.Size([1, 512, 768])
after decoder #4   : torch.Size([1, 512, 768])
after decoder #5   : torch.Size([1, 512, 768])
after decoder #6   : torch.Size([1, 512, 768])
after decoder #7   : torch.Size([1, 512, 768])
after decoder #8   : torch.Size([1, 512, 768])
after decoder #9   : torch.Size([1, 512, 768])
after decoder #10  : torch.Size([1, 512, 768])
after decoder #11  : torch.Size([1, 512, 768])
decoder_output     : torch.Size([1, 512, 768])  #(3)
text_output        : torch.Size([1, 512, 40000])  #(4)
```

根据上述输出，我们可以看到我们的 `self.token_embedding` 层成功地将 512 个标记的序列（`#(1)`）转换成了 768 维标记向量序列（`#(2)`）。这个张量维度一直保持到最后一个解码器层，其输出被存储在 `decoder_output` 变量中（`#(3)`）。最后，在经过 *任务分类器* 头处理后，张量维度变为 1×512×40000（`#(4)`），包含有关下一标记预测的信息。在原始 Transformer 中，这通常被称为 *右移* 输出。这基本上意味着存储在 0 行的信息是第 1 个标记的预测，第 1 行包含第 2 个标记的预测，依此类推。因此，由于我们想要预测第 513 个标记，我们可以简单地取最后一行（第 512 行）并选择与概率最高的标记对应的元素。

要计算模型参数的数量，我们可以使用下面的`count_parameters()`函数。

```py
# Codeblock 9
def count_parameters(model):
    return sum([params.numel() for params in model.parameters()])

count_parameters(gpt1)
```

```py
# Codeblock 9 output
146534464
```

我们可以看到，我们的 GPT-1 实现大约有 1.46 亿个参数。— 我必须承认，这个数字与原始论文中披露的数字不同，即 1.17 亿。这种差异可能是因为我遗漏了一些复杂的细节。如果你知道我应该更改代码的哪个部分来实现这个数字，请随时评论！

### GPT-1 实现：任务分类器头部

记住，我们的`GPT1()`类只包括*文本预测*头部。对于语言模型来说，这已经足够了，但对于微调，我们需要手动创建*任务分类器*头部。查看下面的代码块 10，看看我是如何实现的。

```py
# Codeblock 10
class TaskClassifier(nn.Module):
    def __init__(self):
        super().__init__()

        self.linear = nn.Linear(in_features=D_MODEL, out_features=N_CLASS)  #(1)
        nn.init.normal_(self.linear.weight, mean=0, std=0.02)

    def forward(self, x):  #(2)
        print(f"decoder_outputt: {x.shape}")

        class_output = self.linear(x)
        print(f"class_outputt: {class_output.shape}")

        return class_output
```

与*文本预测*类似，*任务分类器*头部基本上也是一个线性层。然而，在这种情况下，它将每个 768 维的 token 嵌入映射到 3（`N_CLASS`）个输出值，这些输出值对应于我们想要训练的分类任务的类别数量（`#(1)`）。稍后，解码器的输出将被用作`forward()`方法的输入（`#(2)`）。因此，为了测试这个`TaskClassifier()`类，我将通过一个与解码器输出维度完全匹配的虚拟张量，即 1×512×768。我们可以在下面的代码块 11 中看到，这个张量成功通过了*任务分类器*头部。

```py
# Codeblock 11
task_classifier = TaskClassifier()

x = torch.randn(BATCH_SIZE, SEQ_LENGTH, D_MODEL)
x = task_classifier(x)
```

```py
# Codeblock 11 output
decoder_output : torch.Size([1, 512, 768])
class_output   : torch.Size([1, 512, 3])  #(1)
```

如果我们仔细观察上面的输出，我们可以看到生成的张量现在具有 1×512×3（`#(1)`）的形状。这本质上意味着每个单独的标记现在都由 3 个数字表示。如前所述，在这个例子中，我们将模拟一个具有 3 个类别：*积极*、*消极*和*中立*的情感分析任务。为了确定整个序列的情感，我们可以聚合所有标记的 logits，或者只使用最后一个标记的 logits（考虑到它已经包含了整个序列的信息）。此外，使用相同的输出张量形状，我们可以使用类似的想法来执行 token 级别的分类任务，例如*命名实体识别（NER）*或*词性标注（POS）*。

在推理阶段稍后，每次我们想要执行特定的下游任务时，我们都会使用`TaskClassifier()`头部。下面代码块 12 是一个执行前向传递的示例代码。它本质上做的是将分词后的句子传递到`gpt1`模型中，该模型返回原始解码输出和下一个单词的预测（`#(1)`）。然后，我们使用解码器的输出作为*任务分类器*头部的输入，该头部将返回可用类别的 logits（`#(2)`）。

```py
# Codeblock 12
def gpt1_fine_tune(x, gpt1, task_classifier):
    print(f"original inputtt: {x.shape}")

    decoder_output, text_output = gpt1(x)  #(1)
    print(f"decoder_outputtt: {decoder_output.shape}")
    print(f"text_outputtt: {text_output.shape}")

    class_output = task_classifier(decoder_output)  #(2)
    print(f"class_outputtt: {class_output.shape}")

    return text_output, class_output
```

根据以下代码块产生的输出，我们可以看到我们上面的`gpt1_fine_tune()`函数工作正常。

```py
# Codeblock 13
gpt1 = GPT1()
task_classifier = TaskClassifier()

x = torch.randint(0, VOCAB_SIZE, (BATCH_SIZE, SEQ_LENGTH))
text_output, class_output = gpt1_fine_tune(x, gpt1, task_classifier)
```

```py
# Codeblock 13 output
original input  : torch.Size([1, 512])
decoder_output  : torch.Size([1, 512, 768])
text_output     : torch.Size([1, 512, 40000])
class_output    : torch.Size([1, 512, 3])
```

### GPT-1 局限性

尽管在处理我在图 6 中展示的四个下游任务方面取得了显著的结果，但重要的是要知道这种方法存在一些缺点。首先，训练过程复杂，因为我们需要在不同的过程中执行 *预训练* 和 *微调*。其次，由于微调是一个判别过程，我们仍然需要进行手动标注（与预训练的生成过程不同，预训练使用的是自监督标注方法）。第三，模型不够灵活，因为它只能在其微调的任务上工作。例如，一个专门用于情感分析的模型不能用于问答任务。-- 幸运的是，GPT-2 很快就被引入来处理这些问题。

* * *

## GPT-2

GPT-2 在 GPT-1 几个月后发表的论文 "*Language Models are Unsupervised Multitask Learners*" 中被介绍 [6]。这篇论文的作者发现，普通的 GPT 语言模型实际上可以在不进行微调的情况下执行各种下游任务。这是通过修改目标函数来实现的。如果 GPT-1 仅根据前一个标记序列进行预测，即 *P(output | input)*，那么 GPT-2 不仅基于序列，还基于给定的任务进行预测，即 *P(output | input, task)*。有了这个特性，相同的提示词会在给定的任务不同时导致模型产生不同的输出。而且，我们可以简单地将在提示词中包含任务作为自然语言。

例如，如果你用一个提示词 "*lorem ipsum dolor sit amet*" 来提示模型，它很可能会继续输出 "*consectetur adipiscing elit.*"。但是，如果你在提示词中加入一个任务，比如 "它是什么意思？"，模型将会给出关于它实际含义的解释。我尝试了在 ChatGPT 上这样做，得到的答案正是我预期的。

![图 12. 如果未指定任务，ChatGPT 只会继续输入句子 [3]。](../Images/077b01df98b1defbc6d4a9d76de48a1b.png)

图 12. 如果未指定任务，ChatGPT 只会继续输入句子 [3]。

![图 13. 分配特定任务导致模型响应不同的示例 [3]。](../Images/61bbd50cadbc5094f390a3c89f5c6631.png)

图 13. 分配特定任务导致模型响应不同的示例 [3]。

通过以自然语言的形式提供任务的想法可以通过用大量文本进行自监督训练模型来实现。为了比较，GPT-1 用于执行语言模型的语料库数据集是 BooksCorpus 数据集，其中包含超过 7000 本未出版的书籍，相当于大约 5GB 的文本。同时，用于 GPT-2 的数据集是 WebText，其大小约为 40GB。不仅数据集更大，模型本身也更大。GPT-2 论文的作者创建了四个模型变体，每个变体都有不同的配置，如图 14 所示。第一行的是与我们刚刚实现的 GPT-1 论文等效的，而被称为 GPT-2 的模型是最后一行。我们可以看到，GPT-2 在参数数量方面比 GPT-1 大约大 13 倍。基于关于数据集和模型大小的这些信息，我们可以肯定地期待 GPT-2 的表现会比其前辈好得多。

![图 14. GPT-2 论文中提出的四种模型变体[6]。](../Images/88de20c407bef7368cf9124bb5ccbf5f.png)

图 14. GPT-2 论文中提出的四种模型变体[6]。

需要知道的是，如果我们实际上要创建模型，`N_LAYERS`和`D_MODEL`并不是我们需要更改的唯一参数。下面的代码块显示了 GPT-2 的完整参数配置。

```py
# Codeblock 14
BATCH_SIZE = 1
SEQ_LENGTH = 1024  #(1)
VOCAB_SIZE = 50257  #(2)

D_MODEL    = 1600
NUM_HEADS  = 25  #(3)
HIDDEN_DIM = D_MODEL*4  #(4)
N_LAYERS   = 48
DROP_PROB  = 0.1
```

在这个 GPT 版本中，作者将考虑用于预测下一个标记的标记数从 512 扩展到 1024（`#(1)`），这样现在它可以关注和处理更长的标记序列，允许模型接受更长的提示。词汇量也更大。在 GPT-1 中，之前独特的标记数仅为 40,000，但在 GPT-2 中，这个数字增加到 50,257（`#(2)`）。我们需要更改的最后一件事是注意力头的数量，现在设置为 25，如第`#(3)`行所示。`HIDDEN_DIM`参数实际上也发生了变化，但我们不需要手动指定这个值，因为它仍然配置为是嵌入维度的 4 倍大（`#(4)`）。

### GPT-2 实现：解码器

谈到架构实现，重要的是要知道 GPT-2 中使用的解码器与 GPT-1 中使用的解码器有所不同。在 GPT-2 的情况下，我们使用所谓的*预归一化*，而 GPT-1 使用*后归一化*。*预归一化*的想法是在执行主要操作之前放置*层归一化*，即*多头注意力*和*前馈*块。您可以在以下图中看到说明。

![图 15. 没有任务分类器头的 GPT-1 架构（左）和 GPT-2 架构（右）[3]。](../Images/f77fea48d22c1492219a5bd307aa3b61.png)

图 15. 没有任务分类器头的 GPT-1 架构（左）和 GPT-2 架构（右）[3]。

我在下面的`DecoderGPT23()`类中实现了 GPT-2 的解码器。剧透一下：我这样命名是因为 GPT-2 和 GPT-3 的架构结构完全相同。

```py
# Codeblock 15
class DecoderGPT23(nn.Module):
    def __init__(self):
        super().__init__()

        self.norm_0 = nn.LayerNorm(D_MODEL)
        self.multihead_attention = nn.MultiheadAttention(embed_dim=D_MODEL, 
                                                         num_heads=NUM_HEADS, 
                                                         batch_first=True)
        self.dropout_0 = nn.Dropout(DROP_PROB)

        self.norm_1 = nn.LayerNorm(D_MODEL)
        self.feed_forward = nn.Sequential(nn.Linear(D_MODEL, HIDDEN_DIM), 
                                          nn.GELU(), 
                                          nn.Linear(HIDDEN_DIM, D_MODEL))
        self.dropout_1 = nn.Dropout(DROP_PROB)

        nn.init.normal_(self.feed_forward[0].weight, 0, 0.02)
        nn.init.normal_(self.feed_forward[2].weight, 0, 0.02)

    def forward(self, x, attn_mask):
        residual = x
        print(f"original &amp; residualt: {x.shape}")

        x = self.norm_0(x)
        print(f"after normalizationt: {x.shape}")

        x = self.multihead_attention(x, x, x, attn_mask=attn_mask)[0]
        print(f"after attentiontt: {x.shape}")

        x = self.dropout_0(x)
        print(f"after dropouttt: {x.shape}")

        x = x + residual
        print(f"after additiontt: {x.shape}")

        residual = x
        print(f"nx &amp; residualtt: {x.shape}")

        x = self.norm_1(x)
        print(f"after normalizationt: {x.shape}")

        x = self.feed_forward(x)
        print(f"after feed forwardt: {x.shape}")

        x = self.dropout_1(x)
        print(f"after dropouttt: {x.shape}")

        x = x + residual
        print(f"after additiontt: {x.shape}")

        return x
```

好吧，我认为没有必要进一步解释上面的代码，因为它基本上与 GPT-1 的解码器相同，只是在这里我们将*层归一化*块放置在不同的位置。所以，现在我们将直接跳到测试代码。请看下面的代码块 16。

```py
# Codeblock 16
decoder_gpt_2 = DecoderGPT23()
x = torch.randn(BATCH_SIZE, SEQ_LENGTH, D_MODEL)
look_ahead_mask = create_mask()

x = decoder_gpt_2(x, look_ahead_mask)
```

我们可以在生成的输出中看到，我们的`x`张量成功通过了解码器层内部的所有子组件。

```py
# Codeblock 16 output
original &amp; residual : torch.Size([1, 1024, 1600])
after normalization : torch.Size([1, 1024, 1600])
after attention     : torch.Size([1, 1024, 1600])
after dropout       : torch.Size([1, 1024, 1600])
after addition      : torch.Size([1, 1024, 1600])

x &amp; residual        : torch.Size([1, 1024, 1600])
after normalization : torch.Size([1, 1024, 1600])
after feed forward  : torch.Size([1, 1024, 1600])
after dropout       : torch.Size([1, 1024, 1600])
after addition      : torch.Size([1, 1024, 1600])
```

### GPT-2 实现：带有输入和文本预测的解码器

虽然 GPT-2 中使用的解码器与 GPT-1 中使用的不同，但其他组件，即*位置编码*和*前瞻掩码*保持不变。因此，我们可以直接重用它们。用于附加这两个组件的代码基本上相同，但在下面的代码块 17 中仍有一些需要注意的复杂细节。首先，在这里我们在将层归一化层放置在流程中之前，在行`#(1)`处初始化了另一个*层归一化*层。这基本上是因为在 GPT-2 中，我们还有一个放置在解码器外部的*层归一化*块，这在 GPT-1 中之前是不存在的（见图 15）。其次，没有必要像在`GPT1()`类中那样（在代码块 7b 的行`#(4)`）存储原始解码器输出。这基本上是因为 GPT-2 不需要进行微调来执行任何下游任务。相反，它将完全依赖于*任务预测*头来执行。

```py
# Codeblock 17
class GPT23(nn.Module):
    def __init__(self):
        super().__init__()

        self.token_embedding = nn.Embedding(num_embeddings=VOCAB_SIZE, 
                                            embedding_dim=D_MODEL)

        self.positional_encoding = PositionalEncoding()

        self.decoders = nn.ModuleList([DecoderGPT23() for _ in range(N_LAYERS)])

        self.norm_final = nn.LayerNorm(D_MODEL)  #(1)

        self.linear = nn.Linear(in_features=D_MODEL, out_features=VOCAB_SIZE)

        nn.init.normal_(self.token_embedding.weight, mean=0, std=0.02)
        nn.init.normal_(self.linear.weight, mean=0, std=0.02)

    def forward(self, x):
        print(f"original inputtt: {x.shape}")

        x = self.token_embedding(x.long())
        print(f"embedded tokenstt: {x.shape}")

        x = x + self.positional_encoding()
        print(f"after additiontt: {x.shape}")

        for i, decoder in enumerate(self.decoders):
            x = decoder(x, attn_mask=look_ahead_mask)
            print(f"after decoder #{i}t: {x.shape}")

        x = self.norm_final(x)  #(2)
        print(f"after final normt: {x.shape}")

        text_output = self.linear(x)
        print(f"text_outputtt: {text_output.shape}")

        return text_output
```

现在我们可以使用以下代码块测试上面的`GPT23()`类。这里我用长度为 1024 的标记序列进行测试。生成的输出非常长，因为我们重复了 48 次解码器层。

```py
# Codeblock 18
gpt2 = GPT23()

x = torch.randint(0, VOCAB_SIZE, (BATCH_SIZE, SEQ_LENGTH))
x = gpt2(x)
```

```py
# Codeblock 18 output
original input    : torch.Size([1, 1024])
embedded tokens   : torch.Size([1, 1024, 1600])
after addition    : torch.Size([1, 1024, 1600])
after decoder #0  : torch.Size([1, 1024, 1600])
after decoder #1  : torch.Size([1, 1024, 1600])
after decoder #2  : torch.Size([1, 1024, 1600])
after decoder #3  : torch.Size([1, 1024, 1600])
.
.
.
.
after decoder #44 : torch.Size([1, 1024, 1600])
after decoder #45 : torch.Size([1, 1024, 1600])
after decoder #46 : torch.Size([1, 1024, 1600])
after decoder #47 : torch.Size([1, 1024, 1600])
after final norm  : torch.Size([1, 1024, 1600])
text_output       : torch.Size([1, 1024, 50257])
```

如果我们尝试打印出参数数量，我们可以看到 GPT-2 大约有 16 亿个参数。就像我们之前做的 GPT-1 实现一样，这个参数数量也与论文中披露的略有不同，论文中显示的约为 15 亿，如图 14 所示。

```py
# Codeblock 19
count_parameters(gpt2)
```

```py
# Codeblock 19 output
1636434257
```

* * *

## GPT-3

GPT-3 是在 2020 年发布的论文 "*Language Models are Few-Shot Learners*" 中提出的，这篇论文的标题表明所提出的模型能够在仅提供几个示例的情况下执行广泛的任务，也就是所谓的 "*shots.*" 尽管强调了 *few-shot* 学习，但在实践中，这个模型也能够执行 *one-shot* 或甚至 *zero-shot* 学习。如果你还不熟悉 *few-shot* 学习，它基本上是一种使用少量示例来适应特定任务的方法。尽管目标与 *微调* 相似，但 *few-shot* 学习允许它在不更新模型权重的情况下做到这一点。在 GPT 模型的案例中，这要归功于 *注意力* 机制的存在，它允许模型动态地关注提示和示例中提供的最相关部分。与从 GPT-1 到 GPT-2 的改进类似，GPT-3 在 *few-shot* 学习方面比其前辈表现更好，这也要归功于使用了更多的训练数据和更大的模型大小。

### **GPT-3 实现方法：模型配置与架构设计**

你已经读到了剧透，对吧？GPT-3 的架构设计与 GPT-2 完全相同。它们之间的区别仅在于模型大小，我们可以通过使用更大的参数值来调整。下面的代码块 20 展示了 GPT-3 的参数配置。

```py
# Codeblock 20
BATCH_SIZE = 1
SEQ_LENGTH = 2048
VOCAB_SIZE = 50257

D_MODEL    = 12288
NUM_HEADS  = 96
HIDDEN_DIM = D_MODEL*4
N_LAYERS   = 96
DROP_PROB  = 0.1
```

由于上述变量已经更新，我们可以简单地运行以下代码块来初始化 GPT-3 模型 (`#(1)`) 并通过它传递一个表示标记序列的张量 (`#(2)`).

```py
# Codeblock 21
gpt3 = GPT23()  #(1)

x = torch.randint(0, VOCAB_SIZE, (BATCH_SIZE, SEQ_LENGTH))
x = gpt3(x)  #(2)
```

不幸的是，由于内存有限，我无法运行上述代码。我甚至尝试在拥有 30 GB 内存 的 Kaggle Notebook 上运行它，但仍然出现了内存不足的错误。所以，对于这个，我无法向你展示模型初始化时创建的参数数量。然而，论文中提到 GPT-3 由大约 1750 亿个参数组成，这基本上意味着它比 GPT-2 大 100 多倍，所以现在也就明白了为什么它只能在极其庞大且强大的机器上运行。看看下面的图，看看 GPT 版本之间是如何不同的。

![图 16\. 不同 GPT 版本的比较 [3].](../Images/cc9ae545b306f93ce89baa4a71c5626e.png)

图 16\. 不同 GPT 版本的比较 [3]。

* * *

## 结束

关于不同 GPT 版本的理论和实现，尤其是 GPT-1、GPT-2 和 GPT-3 的内容，基本上就这些了。正如本文所写，OpenAI 尚未官方公布 GPT-4 的架构细节，所以我们目前还不能复现它。希望 OpenAI 能很快发布论文！

感谢你阅读我的文章到这里。我非常感激你的时间，并希望你在这里学到一些新东西。祝你有美好的一天！

_ 注意：您也可以在此处访问本文中使用的代码[这里](https://github.com/MuhammadArdiPutra/medium_articles/blob/main/Meet%20GPT%2C%20The%20Decoder-Only%20Transformer.ipynb)._

## 参考文献

[1] Ashish Vaswani 等人. 注意力即是所需。Arxiv. [`arxiv.org/pdf/1706.03762`](https://arxiv.org/pdf/1706.03762) [访问日期：2024 年 10 月 31 日].

[2] Muhammad Ardi. 论文解读：注意力即是所需。数据科学之路。 [`medium.com/towards-data-science/paper-walkthrough-attention-is-all-you-need-80399cdc59e1`](https://medium.com/towards-data-science/paper-walkthrough-attention-is-all-you-need-80399cdc59e1) [访问日期：2024 年 11 月 4 日].

[3] 原始图像由作者创建。

[4] Alec Radford 等人. 通过生成预训练改进语言理解。OpenAI. [`cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf`](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf) [访问日期：2024 年 10 月 31 日].

[5] 基于文献[1]由作者创建的图像。

[6] Alec Radford 等人. 语言模型是无监督多任务学习者。OpenAI. [`cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf`](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) [访问日期：2024 年 10 月 31 日].

[7] Top B. Brown 等人. 语言模型是少样本学习者。Arxiv. [`arxiv.org/pdf/2005.14165`](https://arxiv.org/pdf/2005.14165) [访问日期：2024 年 10 月 31 日].

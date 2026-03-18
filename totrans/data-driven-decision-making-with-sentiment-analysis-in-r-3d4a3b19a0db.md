# 使用 R 进行情感分析的数据驱动决策

> 原文：[`towardsdatascience.com/data-driven-decision-making-with-sentiment-analysis-in-r-3d4a3b19a0db/`](https://towardsdatascience.com/data-driven-decision-making-with-sentiment-analysis-in-r-3d4a3b19a0db/)

![图片由 Ralf Ruppert 提供，来自 Pixabay](img/3862914d816d81ad74ca6413bdaaa30e.png)

图片由 [Ralf Ruppert](https://pixabay.com/users/ralf1403-21380246/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7602768) 提供，来自 [Pixabay](https://pixabay.com//?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=7602768)

## 商业真的应该倾听客户的意见吗？

在一个快速发展的世界中，每时每刻都在变得更加以 AI 驱动，企业现在需要不断寻求竞争优势以保持可持续性。公司可以通过定期观察和分析客户对其产品和服务的意见来实现这一点。他们通过评估来自许多来源的评论来实现这一点，包括线上和线下。识别客户反馈中的正面和负面趋势，使他们能够微调产品功能和设计满足客户需求的市场营销策略。

因此，需要恰当地辨别客户意见，以找到有价值的见解，这有助于做出明智的商业决策。

## 熟悉情感分析

情感分析，作为自然语言处理（NLP）的一部分，是今天的一种流行技术，因为它研究任何给定文本中人们的意见、情感和情绪。企业可以通过对其收集到的包含有价值信息的反馈应用情感分析来理解公众意见，监控品牌声誉，并改善客户体验。然而，由于其非结构化性质，这可能使其难以分析。通过定期分析客户情感，公司可以识别其优势和劣势，决定如何提升产品开发，并构建更好的营销策略。

在 Python 和 R 中用于情感分析的强大包使企业能够揭示有价值模式，跟踪情感趋势，并做出数据驱动的决策。在本文中，我们将探讨如何使用不同的包（**[Quanteda](https://cran.r-project.org/web/packages/quanteda/readme/README.html), [Sentimentr](https://cran.r-project.org/web/packages/sentimentr/index.html)** 和 **[Textstem](https://cran.r-project.org/web/packages/textstem/index.html)**）通过处理、分析和可视化文本数据来对客户反馈进行情感分析。

## 添加现实世界背景

对于这个教程，让我们考虑一家虚构的科技公司，PhoneTech，它最近为其年轻受众推出了一款新的智能手机，属于预算细分市场。现在，他们想了解其新推出的产品的公众认知，因此希望分析来自社交媒体、在线评论和客户调查的客户反馈。

为了实现这一点，PhoneTech 需要使用情感分析来找出产品的优缺点，指导产品开发，并调整营销策略。例如，PhoneTech 已从各种平台收集反馈，如**社交媒体**（例如，非正式评论如“相机很棒 🔥 但电池续航 😒 . #失望”）、**在线评论**（例如，半结构化评论如“出色的构建质量！ ⭐⭐⭐⭐ 电池续航可以更长一些”），以及**客户调查**（例如，对“您喜欢/不喜欢产品的哪些方面？”等问题的结构化回答）。

需要注意的是，客户反馈通常包括非正式语言、表情符号和特定术语。我们可以使用 R 包来清理、分词和分析这些数据，以便将原始文本转化为可操作的商业洞察。

## 实施情感分析

接下来，我们将使用选择的`quanteda`包在 R 中构建情感分析模型。

## 1. 导入必要的包和数据集

为了评估给定数据集中的情感，我们需要几个包，包括`dplyr`来操作客户反馈条目的数据，`quanteda`（许可证：[GPL-3.0 许可证](https://github.com/quanteda/quanteda#GPL-3.0-1-ov-file)）用于文本分析，以及`quanteda.textplots`来创建词云。此外，`tidytext`（许可证：[MIT 许可证](https://cran.r-project.org/web/licenses/MIT) + 文件 [[LICENSE](https://cran.r-project.org/web/packages/sentimentr/LICENSE)](https://cran.r-project.org/web/packages/tidytext/LICENSE)）用于使用情感词典进行评分，而`ggplot2`将用于数据可视化，`textstem`（许可证：[GPL-2](https://cran.r-project.org/web/licenses/GPL-2)）将帮助进行文本词干提取和词元还原，`sentimentr`（许可证：MIT + 文件 LICENSE）将用于情感分析，而`RColorBrewer`将为我们的可视化提供调色板。

这些包可以通过以下命令轻松安装-

```py
install.packages(c("dplyr", "ggplot2", "quanteda", "quanteda.textplots", 
                   "tidytext", "textstem", "sentimentr", "colorbrewer"))
```

安装完成后，我们可以按以下方式加载包：

```py
# Load necessary R packages
library(dplyr)
library(ggplot2)
library(quanteda)
library(quanteda.textplots)
library(tidytext)
library(textstem)
library(sentimentr)
library(RColorBrewer)
```

**客户评论数据集**

在现实世界数据集的情况下，这些数据实际上会使用来自各种社交媒体平台的多工具进行抓取。收集到的数据将代表包含非正式语言、表情符号和特定领域术语的反馈。这样的综合数据集可以允许对来自不同来源的客户情感和意见进行详细分析。

然而，对于这个教程，让我们使用使用 R 生成的合成数据集，这些数据集通过包涵盖了上述点。包含 200 行的数据集代表了来自不同来源的客户反馈（每行约 2-3 句话），包括带有表情符号和符号的原始文本、缩写等，模仿现实世界场景。这些句子只是电子商务或产品网站上常见评论的通用表示（谈论 UI、设计、手机功能、价格、电池寿命、客户服务支持等关键词），并且与表情符号随机组合以创建评论文本。

您可以在 GitHub 上找到使用 R 生成的合成数据集[这里](https://github.com/Devashree21/Decision-Making-with-Sentiment-Analysis-in-R/blob/main/sentiment_data.csv)。

```py
#load the dataset
data <- read.csv("sentiment_data.csv")
# Print the dimensions (number of rows and columns) of the dataset
dataset_size <- dim(data)
print(dataset_size)
```

![](img/e82f080ba9253d47a8ae3265753c72ef.png)

由于数据集中有很多文本，让我们每行打印几个单词来概述数据集。

为了实现这一点，我们首先定义一个函数来从我们的数据集中每个反馈条目中提取前几个单词。然后，我们将从数据集中随机抽取 5 行，并应用该函数来截断反馈文本。最后，我们将打印出结果数据帧，以了解反馈文本。

```py
# Function to extract the first few words
extract_first_words <- function(text, num_words = 10) {
 if (is.na(text) || !is.character(text)) {
 return(NA)
}
words <- unlist(strsplit(text, "s+"))
 return(paste(words[1:min(num_words, length(words))], collapse = " "))
}
# Randomly sample 5 rows from the dataset
set.seed(123)
random_feedback <- data[sample(nrow(data), size = 5, replace = FALSE), ]
# Extract the first 5 words
random_feedback$text <- sapply(random_feedback$text, function(text) {
truncated <- extract_first_words(text)
paste0(truncated, "...")
})
# Print the data frame
print(random_feedback)
```

![](img/43a5f9b0107dcb920a5f045ae9094cbd.png)

## 2. 预处理文本数据

在进行文本分析之前，我们需要预处理文本以确保格式干净且一致。预处理将涉及几个关键步骤：

1.  **文本清理**，这包括移除标点符号、数字和特殊字符；

1.  **文本归一化**，这包括将字母转换为小写；

1.  **对文本进行分词**，这包括将文本分割成单个单词或标记；

1.  **移除停用词**，这包括有意移除对情感分析不贡献的单词（例如，“the”、“and”）；最后，

1.  **对文本进行词干提取或词形还原**，其中单词被还原到其基本形式。这些步骤有助于减少噪声并提高分析的准确性。

现在，我们将对数据集实施上述预处理步骤。

```py
# Cleaning the dataset
corpus <- quanteda::corpus(data$text)
tokens_clean <- quanteda::tokens(corpus, remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE) %>%
tokens_tolower() %>%
tokens_remove(stopwords("en"))
# Convert tokens to character vectors for lemmatization
tokens_char <- sapply(tokens_clean, function(tokens) paste(tokens, collapse = " "))
# Lemmatize tokens using WordNet
lemmatized_texts <- lemmatize_strings(tokens_char)
```

在此代码中，我们将数据集的文本列转换为 quanteda 语料库对象。我们通过分词来清理文本，这包括去除标点符号、数字和符号，将所有单词转换为小写，并过滤掉常见的停用词。最后，应用词干提取将单词还原到其基本形式，例如将"running"改为"run"，以确保文本分析的一致性。在此需要指出的是，我们没有使用词干提取，因为它可能会由于将单词简化为基本形式而导致部分或不完全提取单词。从另一方面来说，它通过简单的规则来截断单词的末尾。例如，算法可能会从"amazing"中移除后缀"-ing"，得到"amaz"，或者"terrible"可能变成"terribl"。为了避免这种情况并获得更准确的根形式，我们将使用词元化，这是一个更复杂的过程，它依赖于词典将单词映射到其基础或词典形式，并考虑单词的上下文和词性来返回其基础或词典形式。

现在我们已经清理和分词了文本数据，我们可以继续下一步。我们的目标是分析反馈条目中的情感。我们将使用 sentimentr 包来评估结构化数据中的情感，从而深入了解反馈条目的情感基调。

## 3. 使用 Sentimentr 包进行情感分析

现在，我们可以使用`sentimentr`包中的 sentiment 函数对这些句子进行情感分析。此函数计算每段文本的情感得分，并识别正面和负面单词。

接下来，我们总结每个文档的情感得分。我们按文档分组得分，计算正负单词的总数。我们还计算一个复合得分，并将整体情感分类为正面或负面。

```py
# Perform sentiment analysis using sentimentr
sentiment_scores <- sentiment(lemmatized_texts)
# Summarize sentiment scores for each document
sentiment_summary <- sentiment_scores %>%
group_by(element_id) %>%
summarize(
positive_words = sum(sentiment > 0),
negative_words = sum(sentiment < 0),
compound = sum(sentiment)
) %>%
mutate(
sentiment = ifelse(compound > 0, "Positive", "Negative")
)
```

最后，我们将这个情感摘要与原始文本数据合并，并打印结果。这为我们提供了对数据集中情感的一个清晰、简洁的评价。

```py
# Merge with original text for context using row number as a common column
sentiment_summary <- sentiment_summary %>%
mutate(doc_id = as.character(element_id)) %>%
left_join(data %>% mutate(doc_id = as.character(1:nrow(data))), by = "doc_id") %>%
select(text, positive_words, negative_words, compound, sentiment)
# Print the sentiment evaluation table
print(sentiment_summary)
```

![图片](img/f403e1849e5b7b84b99597290b4201a7.png)

输出表格清晰地显示了每行评论中的正面和负面单词计数，以及复合得分以及预测情感。一眼望去，模型在排序正面和负面评论方面做得相当不错。尽管由于评论内容在表格中的不完全显示，一些评论明显看起来是负面的（例如，“不会推荐……。”），但很可能在该特定评论中包含更多正面关键词（如 satisfies、best、good 等），根据模型评估导致正面情感。因此，这样的评论在将其纳入结果解释以供决策之前需要仔细单独审查。

接下来，我们需要打印一个文档特征矩阵（DFM），它是对文本的结构化表示，其中行代表文档，列代表特征（单词）。每个单元格包含文档中特定单词的频率。在这里，清洗后的语料库被转换成 DFM，使其准备好进行统计分析与可视化。

```py
# Create a document-feature matrix (DFM)
dfm <- dfm(corpus_clean)
```

本节计算每个文本条目的情感指标。将正面和负面词汇计数相加，并计算一个复合分数，作为这些计数的差值。正的复合分数表示正面情感，负的分数表示负面情感。这些信息与原始文本结合，进行全面的情感评估。

## 4. 分析情感占比

```py
# Evaluate sentiment proportions as percentages
sentiment_proportion <- sentiment_summary %>%
group_by(sentiment) %>%
summarise(count = n()) %>%
mutate(proportion = count / sum(count) * 100)
print(sentiment_proportion)
```

![图片由作者提供](img/b5c583af560bd0d9c80dd4ab78ebab3e.png)

为了理解整体情感分布，我们计算数据集中正面和负面情感的占比。按情感类型分组，计算每个类别的条目计数，并将其归一化以得出它们的占比。

## 5. 可视化情感分布

我们将在`ggplot2`中创建一个条形图来可视化正面和负面情感的占比，以便直观地展示情感分布，使观察哪种类型的情感似乎占主导地位变得容易。

```py
# Plot sentiment distribution as percentages
ggplot(sentiment_proportion, aes(x = sentiment, y = proportion, fill = sentiment)) +
geom_bar(stat = "identity", width = 0.7) +
scale_fill_manual(values = c("Positive" = "blue", "Negative" = "red")) +
labs(title = "Distribution of Sentiments",
x = "Sentiment Type",
y = "Percentage",
fill = "Sentiment") +
theme_minimal() +
theme(panel.grid = element_blank())
```

![图片由作者提供](img/1bce41f44dad9d500587f335565b1d9e.png)

图片由作者提供

在我们的数据集中，正面情感似乎占主导地位。因此，更多的客户对 PhoneTech 的产品表示满意。

## 6. 可视化高频词

```py
# Plotting top 10 terms
top_terms <- topfeatures(dfm, 10)
bar_colors <- colorRampPalette(c("lightblue", "blue"))(length(top_terms))
# Barplot
barplot(top_terms, main = "Top 10 Terms", las = 2, col = bar_colors, horiz = TRUE, cex.names = 0.7)
```

![图片由作者提供](img/317a409a8c5796638b57666f8b8761d0.png)

图片由作者提供

数据集中出现频率最高的 5 个词似乎是“推荐”、“设计”、“智能手机”、“显示”和“糟糕”。尽管这些词单独使用并不很有助于理解情感，但 PhoneTech 的人员可以深入挖掘这些词在评论中与产品的关联，并构建一些其他图表，以增加更多上下文，从而明确这些词是否在特定的评论中使用。

那么，让我们过滤掉正面反馈，创建一个 DFM，然后再次绘制，看看客户真正对产品说了些什么。

```py
# Filter positive feedback 
positive_feedback <- sentiment_summary %>% 
 filter(sentiment == "Positive") 
# Create a DFM for positive feedback 
positive_tokens <- quanteda::tokens(positive_feedback$text, remove_punct = TRUE, remove_numbers = TRUE, remove_symbols = TRUE) %>% 
 tokens_tolower() %>% 
 tokens_remove(stopwords("en")) 
positive_dfm <- dfm(positive_tokens)
# Plot top 5 terms with a gradient
top_positive <- topfeatures(positive_dfm, 5)
bar_colors <- colorRampPalette(c("lightblue", "blue"))(length(top_positive))
# Plot with gradient
barplot(top_positive, main = "Top 5 Positive Terms", las = 2, col = bar_colors, horiz = TRUE, cex.names = 0.7)
```

![图片由作者提供](img/70f0dc34d12b8f539c16f1d0dec9a683.png)

图片由作者提供

在数据集中，产品性能、智能手机（可能表明品牌）、显示和设计似乎是最常讨论的。

在我们的数据集中，可视化这些情感的一种方法是通过生成词云，并根据需要使用`max_words`参数微调单词频率。

## 7. 生成词云

```py
# Word cloud
textplot_wordcloud(dfm, max_words = 200, color = RColorBrewer::brewer.pal(8, "Reds"), min_size = 0.5, max_size = 8)
```

在进行情感分析任务时，我们还可以使用词云以引人入胜和直观的格式显示最频繁出现的词。需要注意的是，较大的单词表示更高的频率，这个图表特别有助于快速识别给定数据集中的关键主题。

![作者图片](img/a2c8389f670dbf35aef1f41c18754906.png)

作者图片

对于 PhoneTech 团队来说，考虑两个分别针对正面和负面的词云可能值得考虑，以更好地了解产品最被喜爱的功能和痛点。

## 8. 情感抽样与回顾

最后，我们将从数据集中打印五条随机句子以检查其情感评估结果。这将帮助我们验证情感分析输出并深入了解单个条目。

```py
# Sample 5 sentences from the dataset
sample_indices <- sample(1:nrow(sentiment_summary), 5)
sample_sentiment_summary <- sentiment_summary[sample_indices, ]
# Print the sample sentences
print(sample_sentiment_summary)
```

![图片](img/ff04ccbda0016e093a1db25b128ecf80.png)

因此，上述所有步骤形成了一个分析文本数据以及提取有价值见解的全面流程。这些步骤共同帮助将原始文本转化为可操作的见解，支持任何公司的数据驱动决策。

## 解释情感分析结果

正确评估和评估情感分析的结果至关重要。****为此，我们生成了一个文档-特征矩阵（DFM），以找到顶级词汇和整体情感分布，帮助我们了解整体客户情绪并识别反馈中的模式。此外，我们的模型生成了情感分数，以提供关于评论语气的概念。

例如，PhoneTech 发现**68%**的反馈是正面的，其中最常用的词汇是“设计”和“性能”，这突出了营销的关键销售点。相反，剩余的**32%**的评论，即负面评论，讨论了客户服务和糟糕的图片，指出了需要改进的潜在领域。

比较随时间或跨来源（如社交媒体与在线评论）的情感趋势，有助于识别客户感知的变化。准确的解释对于做出明智的决策和制定有针对性的策略至关重要。

虽然该模型似乎能够有效地识别正面和负面评论，但进一步的步骤可以包括微调模型以对中性评论进行排序（如果有的话），以进行更全面的分析。

## 将情感洞察应用于策略微调

情感分析揭示了产品改进的关键领域及其对 PhoneTech 的优势，这些优势可以用来增强其业务。通过解决正面和负面的客户反馈，PhoneTech 可以推动整体满意度并吸引更多买家。

基于情感分析结果，PhoneTech 可以识别以下可操作的见解和策略，以改善其业务：

### 正面策略：

**(1) 精炼营销策略**：

+   客户似乎对简洁快速的 UI 感到满意。

+   对 UI 设计的正面反馈表明，这是关键的销售点，PhoneTech 应在他们的营销活动中继续推广，以吸引更多买家。

### 负面策略：

**(1) 加强产品特性**：

+   关于图像质量的频繁投诉表明硬件或软件存在问题。

+   快速改进这些领域可以增强用户体验并减少负面评论。

**(2) 解决客户服务问题**:

+   及时处理客户服务问题并将它们解决，将提升产品满意度。

+   这些行动可以预防或减少负面评论，同时确保更好的用户体验并提高整体可靠性。

## 情感分析最佳实践

+   **文本上下文**: 由于基于词汇的情感分析往往无法捕捉讽刺和上下文，使用像机器学习这样的高级技术有助于更好地捕捉细微差别。

+   **领域特定语言**: 由于通用词汇表可能不理解行业特定术语和俚语，针对行业技术术语定制词汇表可以提高准确性。

+   **使用非正式语言和表情符号**: 由于非正式语言和表情符号可能难以分析，使用像`quanteda`这样的工具进行数据清理和系统分析是有益的。

+   **结合技术**: 由于依赖单一方法会限制分析深度，将文本处理与机器学习相结合可以提供全面的洞察。

## 主要收获

+   情感分析有助于企业了解客户意见，以改进产品和服务的质量。

+   R 包`quanteda`、`sentimentr`和`textstem`在客户评论文本分析方面协同工作得很好。

+   所概述的情感分析方法可以轻松应用于金融、医疗保健和零售等行业，以获得可操作的见解。

## 结论

情感分析使企业对其客户需求和痛点有清晰的了解。公司可以利用这些见解来改进产品并制定数据驱动的策略。

在本文中，我们探讨了如何使用 R 包帮助对科技产品的客户反馈进行情感分析。我们讨论了挑战的背景，包括可能包括数据收集和准备、语料库创建、分词、特征提取、构建情感模型和可视化结果等步骤，以在 R 中实现情感分析。我们还考虑了分析结果，这些结果似乎有影响，并且公司需要考虑以进一步改进产品。

其他寻求获得可操作见解、增强产品功能、完善营销策略和有效监控品牌声誉的领域公司可以采取与情感分析显著相似的方法。

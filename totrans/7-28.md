# RAG 并非对 LLM 的幻觉免疫

> 原文：[`towardsdatascience.com/detecting-hallucination-in-rag-ecaf251a6633/`](https://towardsdatascience.com/detecting-hallucination-in-rag-ecaf251a6633/)

![由 Johannes Plenio 在 Unsplash 上拍摄的照片](img/98dbf8a75f9cde4b5150acd3a17c5bd0.png)

由[Johannes Plenio](https://unsplash.com/@jplenio?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)拍摄的照片

我最近开始更倾向于 Graph RAGs，而不是基于向量存储的 RAGs。

向向量数据库道歉；在大多数情况下，它们工作得非常出色。但要注意的是，你需要文本中的明确提及来检索正确的上下文。

我们有解决这个问题的方法，我在之前的帖子中介绍了一些。

> [**没有检索模型构建 RAG 是一个严重的错误**](https://towardsdatascience.com/multi-rep-colbert-retrieval-models-for-rags-fe05381b8819)

例如，ColBERT 和 Multi-representation 是我们构建 RAG 应用时应考虑的有用的检索模型。

GraphRAGs 在检索问题上受影响较小（我并没有说它们不受影响）。每当检索需要一些推理时，GraphRAG 的表现都非常出色。

提供相关上下文解决了基于 LLM 的应用中的一个关键问题：幻觉。然而，这并不能完全消除幻觉。

当你不能修复某事时，你就去衡量它。这就是本文的重点。换句话说，**我们如何评估 RAG 应用**？

但在那之前，为什么 LLM 最初会撒谎呢？

## 为什么 LLM 会幻觉（即使是 RAG）？

语言模型有时会撒谎——好吧——有时是不准确的。这主要是由于两个原因。

第一个原因是 LLM 没有足够的上下文来回答。这就是检索增强生成（RAG）出现的原因。RAGs 为 LLM 提供它在训练中未曾见过的上下文。

一些模型在提供上下文的情况下可以很好地回答问题，而另一些则不行。例如，[LLama 3.1 8B](https://ai.meta.com/blog/meta-llama-3-1/)在提供上下文以生成答案时表现良好，而[DistilBERT](https://huggingface.co/docs/transformers/en/model_doc/distilbert)则不行。

> [**我微调了 Tiny Llama 3.2 1B 以替代 GPT-4o**](https://towardsdatascience.com/i-fine-tuned-the-tiny-llama-3-2-1b-to-replace-gpt-4o-7ce1e5619f3d)

第二个原因是当答案需要一些推理时。并不是每个 LLM 都擅长推理；每个 LLM 的推理方式都不同。例如，[Llama 2 13B 在推理任务上的表现不如 GPT-4o](https://www.llm-reasoners.net/leaderboard)。

当然，这些模型来自不同的时代，把它们并排放在一起比较并不合适。但这也是我想说明的重点——如果你没有明智地选择你的模型，你可能会期待它产生幻觉的答案。

* * *

## 如何评估 RAG 中的幻觉

现在我们已经确信每个基于 LLM 的应用都可能产生幻觉，并且测量是唯一控制它的方法，我们该如何去做呢？

最近，一些 LLM 评估框架已经发展起来。其中两个，[RAGAS](https://docs.ragas.io/en/v0.1.21/index.html#)和[Deepeval](https://docs.confident-ai.com/)，特别受欢迎。这两个工具都是开源的，并且免费使用，尽管存在付费版本。

在这篇文章中，我将使用 Deepeval。一个快速说明：我不是 Deepeval 的关联者；我只是喜欢它。

让我们从安装这个工具开始。你可以从 PyPI 仓库获取它。

```py
pip install -qU deepeval ragas
```

因为 LLM 评估是模糊的，所以我们不能像测试软件那样测试 LLM。LLM 生成的响应是不可预测的，因此需要另一个 LLM 来评估它们。

LLM 评估者需要是一个有能力的模型。我会选择 GPT-4o-mini 来进行这个评估。它既经济又准确，但你可以尝试其他选项。

我们可以用两种不同的方式评估 RAGs。第一种是基于提示的技术，称为**[G-Eval](https://arxiv.org/abs/2303.16634)**。第二种是**RAGAS，它允许我们系统地评估 RAGs**。

RAGAS 和 G-Eval 都是评估 RAGs 的非常有帮助的框架。当在这两个框架之间选择时，我会使用 RAGAS 作为默认方法。由于计算是特定的，结果很容易与不同的评估进行比较。然而，评估标准有时可能比 RAGAS 框架更复杂。在这种情况下，你可以使用 G-Eval 来指定评估步骤。

### G-Eval：更通用的评估框架

如前所述，G-Eval 是一个基于提示的评估框架，这意味着我们可以告诉 LLM 如何评估我们的 RAG。我们可以通过设置输入参数`criteria`或`evaluation_steps`来实现这一点。

这里有一个使用`criteria`参数的例子。

```py
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase
from deepeval.test_case import LLMTestCaseParams

test_case = LLMTestCase(
    input="When did XYZ, Inc complete the acquisition of ABC, Inc?",
    actual_output="XYZ, Inc completed the acquisition of ABC, Inc on January 15, 2025, solidifying its market leadership.",
    expected_output="XYZ, Inc completed the acquisition of ABC, Inc on January 10, 2025.",
)

correctness_metric_criteria = GEval(
    name="Correctness with Criteria",
    criteria="Verify if the actual output provides a factually accurate and complete response to the expected output without contradictions or omissions.",
    evaluation_params=[LLMTestCaseParams.INPUT, LLMTestCaseParams.ACTUAL_OUTPUT, LLMTestCaseParams.EXPECTED_OUTPUT],
)

correctness_metric_criteria.measure(test_case)

print("Score:", correctness_metric_criteria.score)
print("Reason:", correctness_metric_criteria.reason)

>> Score: 0.6683615588714628
>> Reason: The Actual Output accurately states the acquisition and maintains a similar context, but the date is incorrect compared to the Expected Output.
```

上述例子验证了实际输出与预期输出的一致性。在这种情况下，日期不同。但其他所有内容都是正确的。

G-Eval 的好处在于我们可以定义如何评估它。如果我们对日期差异小于 10 天的情况表示接受，我们可以在标准中指定。以下是新的评估方法。

```py
...

correctness_metric_criteria = GEval(
    ...
    criteria="Verify if the actual output is correct and the date in the expected output is not more than 10 days apart. ",
    ...
)

...

>> Score: 0.8543395625001086
>> Reason: The Actual Output provides an accurate acquisition date of XYZ, Inc completing the acquisition of ABC, Inc and matches the Expected Output except for a 5-day difference in dates, which is within the acceptable range.
```

正如你所注意到的，这使分数从.66 上升到.85，因为我们允许日期匹配的 10 天范围。

上述例子仅检查 LLM 关于预期结果的响应是否正确。然而，我们还需要检查检索到的数据来评估 RAG。

这里有一个 RAG 评估的例子。请注意，我们使用了`evaluation_steps`参数而不是`criteria`。

```py
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase, LLMTestCaseParams

# Define the retrieval context with multiple relevant pieces of information
retrieval_context = [
    "XYZ Corporation announced its plans to acquire ABC Enterprises in a deal valued at approximately $4.5 billion.",
    "The merger is expected to consolidate XYZ's market position and expand its reach into new domains.",
    "The acquisition was completed on January 15, 2024.",
    "Post-acquisition, XYZ, Inc. aims to integrate ABC, Inc.'s technologies to enhance its product offerings.",
    "The regulatory bodies approved the acquisition without any objections."
]

# Create the test case with input, actual output, expected output, and retrieval context
test_case = LLMTestCase(
    input="When did XYZ, Inc. complete the acquisition of ABC, Inc?",
    actual_output="XYZ, Inc. completed the acquisition of ABC, Inc. on January 17, 2024, solidifying its market leadership.",
    expected_output="XYZ, Inc. completed the acquisition of ABC, Inc. on January 15, 2024.",
    retrieval_context=retrieval_context
)

# Define the correctness metric with evaluation steps
correctness_metric_steps = GEval(
    name="Correctness with Evaluation Steps",
    evaluation_steps=[
        "verify the retrieval_context has sufficient information to respond to the input. "
        "Verify if the 'actual_output' provides the a completion date not more than 10 days different of the acquisition as stated in the 'retrieval_context'.",
        "Ensure that the 'actual_output' matches the 'expected_output' in terms of factual accuracy.",
        "Check for any contradictions or omissions between the 'actual_output' and the 'expected_output'."
    ],
    evaluation_params=[
        LLMTestCaseParams.INPUT,
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT,
        LLMTestCaseParams.RETRIEVAL_CONTEXT
    ],
)

# Measure the test case using the defined metric
correctness_metric_steps.measure(test_case)

# Print the evaluation score and reason
print("Score:", correctness_metric_steps.score)
print("Reason:", correctness_metric_steps.reason)

>> Score: 0.7907626389536403
>> Reason: The retrieval_context provides accurate completion info as January 15, 2024\. The actual_output date is close but slightly off, being two days later than in the expected_output. No other significant factual inaccuracies or contradictions are present.
```

在上述例子中，我们使用了`evaluation_steps`而不是`criteria`。但这不是必须的。在单个语句`criteria`参数中解释评估过程也是完全可以的。但将它们分解成步骤总是更容易。

G-Eval 无论步骤如何，都提供了一个单一的评估得分。你给它一个单一的评估步骤，或者是一打步骤，G-Eval 将完成所有工作并产生一个单一的得分。

这很容易理解，通常也足够。但如果我们需要逐个测试 RAG 管道组件，那该怎么办？这就是 RAGAS 发挥作用的地方。

### RAGAS：标准和更细粒度的评估

RAGAS 是 RAG 管道中四个其他评估的组合。它们是答案相关性、忠实度、上下文回忆和上下文精确度。RAGAS 得分只是这些四个的平均值。

这里有一个例子：

在下面的示例中，我同时使用了`RagasMetric`和各个单独的组件。在实际评估中，你可以单独使用这些组件，或者只使用 RagasMetric。

```py
from deepeval import evaluate

from deepeval.test_case import LLMTestCase

from deepeval.metrics.ragas import RagasMetric

# Individual metrics of RagasMetric
from deepeval.metrics.ragas import RAGASAnswerRelevancyMetric
from deepeval.metrics.ragas import RAGASFaithfulnessMetric
from deepeval.metrics.ragas import RAGASContextualRecallMetric
from deepeval.metrics.ragas import RAGASContextualPrecisionMetric

# Define the retrieval context with multiple relevant pieces of information
retrieval_context = [
    "XYZ Corporation announced its plans to acquire ABC Enterprises in a deal valued at approximately $4.5 billion.",
    "The merger is expected to consolidate XYZ's market position and expand its reach into new domains.",
    "The acquisition was completed on January 15, 2024.",
    "Post-acquisition, XYZ, Inc. aims to integrate ABC, Inc.'s technologies to enhance its product offerings.",
    "The regulatory bodies approved the acquisition without any objections."
]

# Create the test case with input, actual output, expected output, and retrieval context
test_case = LLMTestCase(
    input="When did XYZ, Inc. complete the acquisition of ABC, Inc?",
    actual_output="XYZ, Inc. completed the acquisition of ABC, Inc. on January 15, 2025, solidifying its market leadership.",
    expected_output="XYZ, Inc. completed the acquisition of ABC, Inc. on January 15, 2024",
    retrieval_context=retrieval_context
)

# Initialize the RagasMetric with a threshold and specify the model
ragas_metric = RagasMetric(threshold=0.5, model="gpt-4o-mini")
ragas_answer_relavancy_metric = RAGASAnswerRelevancyMetric(threshold=0.5, model="gpt-4o-mini")
ragas_faithfulness_metric = RAGASFaithfulnessMetric(threshold=0.5, model="gpt-4o-mini")
ragas_contextrual_recall_metric = RAGASContextualRecallMetric(threshold=0.5, model="gpt-4o-mini")
ragas_contextual_precision_metric = RAGASContextualPrecisionMetric(threshold=0.5, model="gpt-4o-mini")

# Measure the test case using the RagasMetric
ragas_metric.measure(test_case)
ragas_answer_relavancy_metric.measure(test_case)
ragas_faithfulness_metric.measure(test_case)
ragas_contextrual_recall_metric.measure(test_case)
ragas_contextual_precision_metric.measure(test_case)

# Print the evaluation score
print("Score:", ragas_metric.score)

# Alternatively, evaluate test cases in bulk
result = evaluate([test_case], [
    ragas_metric,
    ragas_answer_relavancy_metric,
    ragas_faithfulness_metric,
    ragas_contextrual_recall_metric,
    ragas_contextual_precision_metric,
])

>>Metrics Summary

  - ✅ RAGAS (score: 0.6664463603011416, threshold: 0.5, strict: False, evaluation model: gpt-4o-mini, reason: None, error: None)
  - ✅ Answer Relevancy (ragas) (score: 0.9988984715390413, threshold: 0.5, strict: False, evaluation model: gpt-4o-mini, reason: None, error: None)
  - ❌ Faithfulness (ragas) (score: 0.0, threshold: 0.5, strict: False, evaluation model: gpt-4o-mini, reason: None, error: None)
  - ✅ Contextual Recall (ragas) (score: 1.0, threshold: 0.5, strict: False, evaluation model: gpt-4o-mini, reason: None, error: None)
  - ❌ Contextual Precision (ragas) (score: 0.3333333333, threshold: 0.5, strict: False, evaluation model: gpt-4o-mini, reason: None, error: None)
```

为了理解这一点，让我们回顾一下 RAG 应用程序如何响应用户的输入。第一步是检索上下文信息。然后 LLM 使用检索到的上下文来回答用户的查询。

**上下文先验**是一个评估与输入相关的陈述是否被排名更高的指标。在我们的例子中，包含获取日期的那个陈述在检索到的上下文中排名第三。因此，它在评估中的得分很低（0.33）。

**上下文回忆**测试检索到的上下文是否足够以获得接近预期输出的答案。在我们的例子中，获取日期在检索到的上下文中可用。因此，这个指标的得分是 1.0。

**忠实度**是衡量响应中幻觉的指标。它检查实际输出与检索到的上下文之间的正确性。在我们的例子中，检索到的上下文明确指出获取是在 2024 年完成的。但输出显示是在 2025 年，这非常错误。因此，它在忠实度得分上得到了 0 分。

最后，**答案相关性**指标衡量生成的响应是否至少是尝试回答正确的问题（输入）。尽管事实上不正确，但在我们的例子中，LLM 回答了正确的问题，因此得到了 0.99 的分数。

**RAGA 得分**是这四个单独指标的加权平均值。

RAGAS 的缺点是，有时即使响应不正确，你也会得到通过。这正是我们例子中的情况。用相差一年的年份回答获取日期并不是微不足道的。但仍然，这只是四个考虑的指标中的一个。因此，整体得分为 0.66，远高于阈值。

因此，我建议使用单个指标来理解组件，而不是使用单个指标（如 RAGAS）来理解整个系统。这有助于更好地调试应用程序。

## 最后的想法

使用 LLM 评估应用程序与评估软件项目不同。评估 RAG 应用程序甚至更加不同和具有挑战性。

这主要是因为 LLM 的响应并不总是相同的。他们有时可能会产生幻觉。

G-Eval 和 RAGAS 在评估 RAG 应用程序方面很受欢迎。尽管这些框架涵盖了 RAG 应用程序的许多方面，它们也发现了大型语言模型（LLM）的幻觉问题。

一旦你发现你的应用程序出现了幻觉，你可以改进工作流程以获取更好的上下文信息（可能使用图数据库）或者改变模型。

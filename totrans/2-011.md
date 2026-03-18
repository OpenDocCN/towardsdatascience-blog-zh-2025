# 使用 DSPy 优化进行系统化 LLM 提示工程

> 原文：[`towardsdatascience.com/systematic-llm-prompt-engineering-using-dspy-optimization/`](https://towardsdatascience.com/systematic-llm-prompt-engineering-using-dspy-optimization/)

<mdspan datatext="el1755899213205" class="mdspan-comment">本文是一次对令人着迷且快速发展的科学——LLM 提示迭代之旅，这是大型语言模型操作（LLMOPs）的基本部分。我们将通过使用真实世界数据集生成客户服务响应的例子，展示如何以系统化的方式使用[DSPy](https://dspy.ai/)开发生成器和 LLM-judge 提示。该项目所有代码，包括几个笔记本和 Claude 编写的优秀的 README 文件，都可以在这里找到[here](https://github.com/rmartinshort/llm_judge_with_dspy/tree/main)。

* * *

应用人工智能领域正在快速发展，通常涉及构建将数据连接到大型语言模型（LLMs）的管道，从而产生商业价值。开发者可以选择大量开源和闭源模型，其中许多最新模型在生成任务（如编码和技术写作）中与专家人类水平相当甚至超过。几个旗舰模型，如 Gemini 2.5，也原生支持多模态，具有视频和音频功能，以近乎人类的方式与用户进行交流。确实，人工智能助手已经迅速渗透到我们的日常生活中——例如，我在过去几个月里越来越多地使用它进行编码、头脑风暴和一般建议。这是一个我们都在学习如何导航的新世界，而且有了这样强大的技术在我们指尖，开始构建原型很容易。但是，由 LLM 驱动的项目在从研究到生产的道路上仍然面临重大挑战。

## 1.0 提示迭代的挑战

提示迭代——构建可信赖的生成式人工智能产品的核心——之所以困难，是因为编写提示的方式有很多，模型对微小变化非常敏感，并且生成任务的成果判断往往具有主观性。随着时间的推移，提示可以通过迭代和边缘情况修复逐渐演变成针对特定模型版本高度优化的复杂文本文件。当团队想要升级到最新版本或切换模型提供商时，这会带来挑战，可能需要大量的重构。如果建立了回归测试流程，那么可以通过逐步调整生成器提示、重新运行测试套件并评估结果来进行迭代。然而，这个过程也很繁琐，评估可能非常主观——即使是那些有幸雇佣了领域专家的团队，当专家意见不一致时也会感到困难。

## 2.0 谁评估输出？

LLMs 是非确定性的生成模型。这些属性是它们功能的核心，但同时也使得评估它们变得困难。传统的自然语言处理指标很少适用，而且通常没有与 LLM 输出进行比较的基准真实值。LLM 评委的概念在这里可以提供帮助，但也会增加复杂性。一个 LLM 评委通常是一个强大的模型，它试图通过确定生成模型输出的质量来执行专家标注员的工作。最先进的基础模型通常非常擅长分类生成的输出是否满足某些预先定义的标准。因此，在理想情况下，我们有一个评委，其提示能够提炼出人类 SME（行业专家）的思维过程，并产生与 SME 共识一致可靠的输出。然后我们可以将此自动应用于代表性的开发集，并比较生成提示版本的结果，以确保我们朝着正确的方向迭代。

## 3.0 增加复杂性

我们如何知道 LLM 评委是可靠的？通常仍然需要人力来标注评委的训练集，然后评委的提示可以调整以生成尽可能匹配人类标签的结果。我们上面讨论的与生成器提示相关的所有复杂性和模型版本依赖性也适用于 LLM 评委，在一个多阶段项目中，如果有几个 LLM 调用，可能还需要几个评委，每个评委都有自己的训练集。

即使对于单一提示工作流程，我们的设置现在也已经变得相当复杂。我们有一个想要迭代的生成器提示，还有一个输入的开发数据集。然后我们有一个单独的输入和输出数据集，用于开发评委。该数据集应由 SME 标注，并分成训练部分——用于构建评委提示——和测试部分，以测试评委提示并防止过拟合。一旦评委被训练，它就被用来针对开发数据集优化生成器提示。然后我们理想上需要另一个保留集，我们可以用它来检查生成器提示是否过度拟合了开发集。提示版本和数据集应该被记录下来，以便实验可以重现。下面的图表说明了这种情况。

![使用自动化技术优化生成器和 LLM 评委提示的实际方法](img/4821534b48822faa3e4a5dff53bc947e.png "A practical methodology for optimizing both generator and LLM judge prompts using automated techniques")

图 1：生成器和 LLM 评委提示优化的高级工作流程。最初看似简单的更新 POC 生成器提示的过程可能变成一个复杂的流程，该流程将受益于版本控制、实验跟踪和其他 LLMOPs 概念。图像由作者生成。

由于单个生成器提示的优化需要许多组件，我们理想情况下需要一个内置日志和跟踪的 LLMOps 框架。此类框架的描述超出了本文的范围，但我推荐 mlflow 文档或本课程后面的模块以获取更多信息。

## 4.0 本文目的

在本文中，我们将为涉及生成有用的客户服务响应的简单玩具问题构建图 1 所示的系统的基本组件。 <mdspan datatext="el1755999685380" class="mdspan-comment">我们将使用此[航空公司客户支持消息数据集](https://www.kaggle.com/datasets/aimack/customer-service-chat-data-30k-rows?resource=download)的修改版，该数据集可在 Kaggle 上以 CC0 许可证获得。该数据集的样本用于生成合成对话集，然后作为本项目的起点。数据生成过程的详细信息将在下一节中分享。</mdspan>

生成器模型的目的是从对话中生成尽可能有帮助的支持代理消息。为了跟踪提示和执行优化，我们将使用 DSPy 编排库。DSPy 的独特之处在于它将原始基于文本的提示抽象为模块化 Python 代码（使用它所称为的签名和模块，我们将在后面讨论），并提供定义成功指标和自动优化提示的工具。DSPy 的承诺是，有了良好的指标和一些计算它的能力（无论是真实数据还是 LLM 评委），可以自动优化提示，而无需手动编辑文本文件。通过这个项目，我们将看到这种方法的一些优缺点。应注意的是，我们在这里只是触及了 DSPy 使用的表面，该库有出色的文档供进一步阅读。

各章节安排如下：

+   首先，我们讨论数据和目标。我们加载数据，进行一些预处理，并采样几百次对话作为我们提示开发集的基础。

+   我们探索基线生成器并使用它来为评委开发集创建输出。我们看看这是如何通过直接 API 调用（即没有 LLM 编排框架）来完成的，以及使用 DSPy 时它看起来像什么。

+   接下来，我们讨论为 LLM 评委创建黄金标准数据集。为了节省时间，我们选择使用强大的闭源 LLM 来生成它，但在实际项目中，这一步将需要人类主题专家的输入。

+   在我们的黄金标准评委数据集就绪后，我们可以使用它来调整 LLM 评委的提示，我们最终将使用它来帮助我们迭代。在这个过程中，我们将触及 DSPy 优化器的工作原理。

+   然后，我们将尝试使用 DSPy 和我们的优化评委来调整基线生成器提示，并在开发数据集上获得尽可能高的分数。

所有这些的目的都是为了朝着建立一个稳健的、可重复且相对自动化的提示开发方法论迈进，这意味着如果选择更换模型提供者，例如，我们可以再次运行它来重新优化生成器的提示。这个方法论远非完美，在结果质量方面，我怀疑它不能替代与行业专家（SMEs）合作进行的仔细手动提示工程。然而，我认为它封装了许多评估驱动开发的原则，并突出了 DSPy 在 LLM 项目中加速迭代的能力。

## 5.0 数据集和目标

我们使用的 Kaggle 客户支持数据集为我们提供了大约 30k 与航空旅行相关的客户问题。为了将其转换成更现实的“客户支持对话”表格，我使用了`gemini-2.5-flash`，利用 Kaggle 数据集的样本来创建一组 5000 个合成对话，以种子话题。每个对话都包含客户和支持消息，并关联一个唯一的 ID 和公司名称。这个[生成过程](https://github.com/rmartinshort/llm_judge_with_dspy/blob/main/00_generate_dataset.ipynb)旨在创建一个合理大小的示例数据集，它类似于可以从真实的客户支持日志中构建的数据集。

这里是一个与美联航相关的合成样本对话示例

```py
Customer: Trying to sort out a friend's return flight from Heathrow but no luck with the usual telephone number. I thought somebody posted another number some time ago but I've searched and can't find anything.
Support:  The main number is 800-433-7300\.  What information do you need regarding your friend's flight?  A confirmation number would be helpful.
Customer:  I don't have a confirmation number.  It's for a friend; I only have her name and the approximate travel dates – sometime in October.  Is there another way to track this down?
Support:  Unfortunately, without a confirmation number or more precise dates, tracking her flight is impossible.  Have her check her email inbox for a confirmation.  If she can't find it, she should contact us directly.
```

我们代理的目标将是扮演客户支持的角色，根据情况尽可能有帮助地回应用户的查询。这是一个具有挑战性的问题，因为客户的问题可能需要高度具体的环境才能得到良好的回答，或者可能完全超出支持代理的控制范围。在这种情况下，模型必须尽力而为，表现出同理心和理解，就像一个经过训练的人类代理一样。它还必须不虚构事实，这是一个非常重要的检查，超出了本文的范围，但在生产中应该进行，理想情况下使用一个可以访问合适知识库的 LLM 评判器。

为了准备数据，我们可以应用一些基本的预处理，这些内容在代码[这里](https://github.com/rmartinshort/llm_judge_with_dspy/blob/main/dspy_judge/data_loader/dataset_loader.py)有详细说明。我们利用 Huggingface 的 datasets 库提供的并行性，允许我们在该数据集上应用基本模型，如 fasttext 语言检测器。我们还需要随机截断对话，以确保最终的表述始终是“客户”消息，因此设置模型生成下一个“支持”响应。为此，我们有一个[这个简单的截断类](http://dspy_judge/processor/conversation_truncator.py)。最后，为模型提供公司名称以提供额外的上下文似乎是有帮助的（这是我们可以使用这里提出的框架测试其准确度提升的东西，所以我们也将它附加到对话中）。

```py
from dspy_judge.data_loader.dataset_loader import CustomerSupportDatasetLoader
from dspy_judge.processor.conversation_truncator import ConversationTruncator
from dspy_judge.processor.utils import concat_company_and_conversation

data_loader = CustomerSupportDatasetLoader()
# load and preprocess 
dataset = data_loader.load_dataset(split="train")
processed_dataset = data_loader.preprocess_dataset(dataset)

truncator = ConversationTruncator(seed=101)
# truncate the conversations
truncated_dataset = truncator.process_dataset(
    processed_dataset,
    min_turns=1,
    ensure_customer_last=True
)
# apply function that concatenates company name to conversation
truncated_dataset = truncated_dataset.map(concat_company_and_conversation)

# sample just a subset of the full dataset
truncated_loaded_sampled = data_loader.get_sample(
truncated_dataset,n_samples=400,seed=10
)
# split that sample into test and train segments
split_dataset = truncated_loaded_sampled.train_test_split(
test_size=0.4, seed=10
)
```

我们用于调整生成器和裁判的开发数据集需要足够小，以便调整快速且高效，但仍然能够代表模型在生产中看到的内容。在这里，我们只选择 400 个截断对话的随机样本，这些样本将在接下来的章节中进一步拆分为测试集和训练集。选择代表性样本进行提示优化的更智能的方法是当前研究的热点，其中一个有趣的例子是[这里](https://aclanthology.org/2025.acl-industry.67/)。

## 6.0 基线生成器和裁判开发集

让我们进一步拆分我们的代表性输入数据集：160 个示例用于裁判开发，240 个示例用于生成器提示开发。这些数字是任意的，但反映了在代表性、时间和成本之间的实际折衷。

要继续进行 LLM 裁判开发，我们需要一些输出。让我们通过我们生成器的基线版本生成一些，我们稍后会对其进行改进。

LLMs 通常在当前我们关注的客户服务任务上表现非常好，因此为了在这个玩具项目中看到有意义的性能提升，让我们使用`gpt-3.5-turbo`作为我们的生成器模型。DSPy 最强大的功能之一是能够在不手动重新优化提示的情况下轻松切换模型，因此一旦系统构建完成，替换为其他模型既容易又有趣。

### 6.1 没有 DSPy 的基本版本

为此项目构建的基线生成器的第一个版本实际上并没有使用 DSPy。它由以下基本手动输入的提示组成

```py
baseline_customer_response_support_system_prompt = "You are a customer service agent whose job is to provide a single, concise response to a customer query. You will receive a transcript of the interaction so far, and your job is to respond to the latest customer message. You'll also be given the name of the company you work for, which should help you understand the context of the messages." 
```

调用一个 LLM，代码中包含多个继承自`LLMCallerBase`的模块，这些模块也可以选择性地使用 instructor 库来强制执行结构化输出（更多关于如何工作的信息请在此处查看）。还有一个名为 ParallelProcessor 的模块，它允许我们并行调用 API，并通过使用回退来自动调节调用，从而最小化过程中的错误。可能有许多方法可以执行这些并行调用——在[之前的文章](https://pub.towardsai.net/data-driven-llm-evaluation-with-statistical-testing-004b1561793f)中，我使用了 Huggingface 数据集的`.map()`功能，而在这里 ParallelProcessor 内部直接使用 python 的 multiprocessing 库，然后从输出列表中重新形成数据集。

如果你处理的是非常大的数据集，这可能不是一个高效的方法，但我已经测试了它对几千个示例的有效性。同时，在测试和运行并行 LLM 调用时，了解 API 成本也非常重要！

将这些事情放在一起，我们可以从截断对话数据集的样本中生成基线结果，如下所示

```py
from dspy_judge.llm_caller import OpenAITextOutputCaller
from dspy_judge.processor.parallel_processor import ParallelProcessor

baseline_model_name = "gpt-3.5-turbo"
baseline_model = OpenAITextOutputCaller(api_key=secrets["OPENAI_API_KEY"])
baseline_processor = ParallelProcessor(baseline_model, max_workers=4)

#split dataset is divided into the generator and judge development segments
dev_dataset = split_dataset["train"]
judge_dataset = split_dataset["test"]

# company_and_transcript is the name of the field generated by the concat_company_and_conversation
# function  
baseline_results_for_judge = baseline_processor.process_dataset(
        judge_dataset,
        system_prompt=baseline_customer_response_support_system_prompt,
        model_name=baseline_model_name,
        input_field="company_and_transcript",
        temperature=1.0
    )

# create a full transcript by adding the latest generated response to the end
# of the input truncated conversation
baseline_results = baseline_results.map(concat_latest_response) 
```

### 6.2 DSPy 看起来如何？

使用 dspy 的感觉与其他 LLM 编排库不同，因为提示组件被抽象化了。然而，我们的代码库允许我们遵循与上面“直接 API 调用”方法相似的样式。

dspy 的核心对象是签名和模块。签名是允许我们定义每个 LLM 调用的输入和输出的类。例如，我们的基线生成器签名看起来像这样

```py
import dspy 

class SupportTranscriptNextResponse(dspy.Signature):

    transcript: str = dspy.InputField(desc="Input transcript to judge")
    llm_response: str = dspy.OutputField(desc="The support agent's next utterance")
```

签名也可以有文档字符串，这本质上是发送给模型的系统指令。因此，我们可以通过简单地添加以下行来使用我们之前已经编写的基线提示

```py
 SupportTranscriptNextResponse.__doc__ = baseline_customer_response_support_system_prompt.strip()
```

DSPy 模块也是一个核心构建块，它为任何给定的签名实现了一种提示策略。和签名一样，它们是完全可定制的，尽管在这个项目中我们主要使用一个名为`ChainOfThought`的签名，该签名实现了思维链提示策略，因此迫使模型在响应签名中指定的响应的同时生成一个推理域。

```py
import dspy 

support_transcript_generator_module = dspy.ChainOfThought(
    SupportTranscriptNextResponse
)

ParallelProcessor has been written to support working with dspy too, so the full code to generate our baseline results with dspy looks like this

# Create DSPy configuration for multiprocessing
dspy_config = {
  "model_name": "openai/gpt-3.5-turbo",
  "api_key": secrets["OPENAI_API_KEY"],
  "temperature": 1
}

generation_processor = ParallelProcessor()

# note judge_dataset is the judge split from Section 5
baseline_results_for_judge = generation_processor.process_dataset_with_dspy(
  judge_dataset,
  input_field="company_and_transcript",
  dspy_module=support_transcript_generator_module,
  dspy_config=dspy_config,
) 
```

您可以通过查看[`process_dataset_with_dspy`](https://github.com/rmartinshort/llm_judge_with_dspy/blob/main/dspy_judge/processor/parallel_processor.py)方法来了解这里设置的细节。为了避免序列化错误，我们从提供的模块中提取 DSPy 签名，然后对其进行解构并在每个工作器上重新组装。然后每个工作器调用

```py
dspy_module.predict(transcript=input_text) 
```

在它接收到的每一行批处理中，然后结果被重新赋值。最终结果应该类似于上一节中用“原生 API”方法生成的结果，唯一的区别来自于高温设置。

DSPy 在这个阶段的重大优势是，`support_transcript_generator_module`模块可以轻松保存、重新加载，并直接输入到其他 DSPy 工具（如 Evaluate 和 Optimize）中，我们将在后面看到。

## 7.0 判决训练数据集

在设置好基线生成器后，我们可以继续进行 LLM 法官的开发。在我们的问题中，我们希望 LLM 法官能够像人类客户支持专家一样，能够阅读其他代理与客户之间的互动，判断代理在解决客户问题上的成功程度，并给出批评以解释推理。为此，拥有一些黄金标准的判断和批评是非常有帮助的。在实际项目中，这可以通过将法官开发数据集通过基线生成器运行，然后由主题专家审查输入和输出，生成带有标签的结果数据集来完成。为了简化问题，通常更喜欢二进制的“是/否”判断，这样我们就可以直接计算准确率、精确度和 Cohen 的 kappa 等指标。为了加快这个玩具项目的这一步骤，我使用了 Claude Opus 4.0 以及一个由 GPT5 辅助精心设计的“黄金标准”[法官提示](https://github.com/rmartinshort/llm_judge_with_dspy/blob/main/dspy_judge/prompts/base_prompts.py)，这是一个强大的工具，但无法替代人类 SME，并且仅用于这个演示项目。

我们再次可以使用 DSPy 的`ChainOfThought`模块，使用如下签名：

```py
import dspy

class SupportTranscriptJudge(dspy.Signature):
    transcript: str = dspy.InputField(desc="Input transcript to judge")
    satisfied: bool = dspy.OutputField(
        desc="Whether the agent satisfied the customer query"
    )
```

由于我们要求思维链，推理字段将自动生成，不需要在签名中指定。

运行我们的“黄金标准法官”以模拟 SME 标签阶段看起来是这样的

```py
import dspy
from dspy_judge.processor.utils import extract_llm_response_fields_dspy

dspy_config = {
      "model_name": "anthropic/claude-sonnet-4-20250514",
      "api_key": secrets["ANTHROPIC_API_KEY"],
      "temperature": 0
}

gold_standard_judge_generator_module = dspy.ChainOfThought(SupportTranscriptJudge)

gold_standard_dspy_judge_processor = ParallelProcessor()

dspy_judge_results_optimized = gold_standard_dspy_judge_processor.process_dataset_with_dspy(
  judge_dataset.select_columns(
    ["conversation_id","output_transcript"]
  ),
  input_field="output_transcript",
  dspy_module=gold_standard_judge_generator_module,
  dspy_config=dspy_config
)
gold_standard_dspy_judge_results = gold_standard_dspy_judge_results.map(
    extract_llm_response_fields_dspy
) 
```

法官训练数据集包含了一系列“正面”和“负面”结果及其相关解释。这是我们所希望的，因为我们需要确保我们的 LLM 法官能够调整以了解如何区分这两者。这也具有优势，可以给我们提供关于基线生成器性能的第一个指标。对于我们的样本数据集，性能并不理想，如图 2 所示，几乎有 50%的失败率。在一个更严肃的项目中，我们希望在 SME 标签阶段暂停，并进行仔细的错误分析，以了解主要失败模式。

![](img/d205358857bfd60bc79bcccb9d32aa07.png)

图 2：根据黄金标准 LLM 法官对基线生成器的性能，该法官针对法官训练数据集运行，并对每个输出对进行二元分类，并生成简短的批评。图像由作者生成。

## 8.0 优化法官提示

在我们的黄金标准法官数据集到位后，我们现在可以继续开发和优化法官提示。一种方法是从基线法官开始，尝试将一些 SME 推理编码到提示中，在法官训练集上运行它，并逐步编辑，直到 SME 分数和法官分数之间的对齐开始趋于平稳。记录每个提示版本及其相关的对齐分数，以便跟踪进度并检测到任何平稳点。另一种方法是首先使用 dspy 的提示优化器为我们完成一些这项工作。

Dspy 优化器接收我们的模块、一个度量函数和一个小型训练集，并尝试优化提示以评分并最大化度量。在我们的案例中，度量将是我们的评判器的二分类与 SME 标记数据集的地面真实值之间的匹配准确率。为此有多种算法，在这里我们关注[MIPROv2](https://dspy.ai/api/optimizers/MIPROv2/)，因为它可以适应系统指令并创建或编辑少量示例。总之，MIPROv2 是一个自动的、迭代的流程，以下步骤：

+   第 1 步：它将提供的模块运行在训练数据集的一个子集上，并筛选出高分轨迹以生成少量示例。

+   第 2 步：它使用 LLM 调用，根据第 1 步的观察结果生成多个候选系统提示。

+   第 3 步：它在训练数据的迷你批次上运行时，寻找最大化度量值的候选系统提示和候选少量示例的组合。

这种算法的承诺是，它们可以帮助我们以数据驱动的模式设计提示，类似于传统 ML 模型的训练方式。缺点是它们将开发人员与数据分开，使得解释为什么选定的提示是最优的变得更加困难，除了“模型说了这样”之外。它们还有许多超参数，其设置可能对结果有很大影响。

根据我到目前为止的经验，在已经拥有地面真实值的情况下，使用优化器是有意义的，它们的输出可以用作手动迭代和与主题专家进一步合作的起点。

让我们看看 MIPROv2 如何应用于我们的项目中以优化 LLM 评判器。我们将选择 Gemini 1.5 闪存作为我们的评判模型，它既便宜又快速。

```py
import dspy
from dspy_judge.prompts.dspy_signatures import SupportTranscriptJudge

judge_model = dspy.LM(
    "gemini/gemini-1.5-flash",
    api_key=secrets["GEMINI_API_KEY"],
    cache=False,
    temperature=0
)
dspy.configure(lm=judge_model,track_usage=True,adapter=dspy.JSONAdapter())
baseline_judge = dspy.ChainOfThought(SupportTranscriptJudge) 
```

注意，这里的基线评判器代表我们根据审查 SME 标记数据集而做出的“最佳尝试”的评判。

出于好奇，让我们首先看看基线评判器和黄金标准评判器之间的初始一致性是什么样的。我们可以通过在一系列示例上运行`dspy.Evaluate`，使用一个度量函数和一个模块来实现这一点。

```py
import dspy
from dspy_judge.processor.utils import convert_dataset_to_dspy_examples

#this is our simple metric function to determine where the judge score matches the gold #standard judge label

def match_judge_metric(example, pred, trace=None):
    example_str = str(example.satisfied).lower().strip()
    # this is going to be True or False
    pred_str = str(pred.satisfied).lower().strip()

    if example_str == pred_str:
        return 1
    else:
        return 0

# Load the SME labelled dataset
dspy_gold_standard_judge_results = data_loader.load_local_dataset("datasets/gold_standard_judge_result")

# Convert HF dataset to list of dspy examples
judge_dataset_examples = convert_dataset_to_dspy_examples(
    dspy_gold_standard_judge_results,
    field_mapping = {"transcript":"output_transcript","satisfied":"satisfied"},
    input_field="transcript"
)

evaluator = dspy.Evaluate(
    metric=match_judge_metric,
    devset=judge_dataset_examples,
    display_table=True,
    display_progress=True,
    num_threads=24,
)
original_score = evaluator(baseline_judge) 
```

对我来说，运行这个给出了大约 60%的基线评判器分数。问题是，我们能否使用 MIPROv2 来提高这个分数？

设置优化运行很简单，但请注意，在此过程中会进行多次 LLM 调用，因此多次运行或在大型训练数据集上运行可能会很昂贵。还建议检查[超参数解释](https://dspy.ai/api/optimizers/MIPROv2/)的文档，并准备好优化可能不会按预期工作。

```py
import dspy

#split the SME labelled dataset into training and testing
training_set = judge_dataset_examples[:110]
validation_set = judge_dataset_examples[110:]

optimizer = dspy.MIPROv2(
    metric=match_judge_metric,
    auto="medium",
    init_temperature=1.0,
    seed=101
)

judge_optimized = optimizer.compile(
    baseline_judge,
    trainset=training_set,
    requires_permission_to_run=False,
)
```

在这个阶段，我们有一个名为 judge_optimized 的新 dspy 模块，我们可以用 dspy 对其进行评估。对训练集和验证集进行评估。当我这样做时，我得到了大约 70%的准确率，这表明优化确实使评判提示与黄金标准标签更加一致。

具体有什么变化？为了找出答案，我们可以运行以下内容

```py
judge_optimized.inspect_history(n=1)
```

这将显示系统的最新版本提示，任何添加的少量示例以及最后的调用。多次运行优化可能会产生相当不同的结果，系统提示从对基线进行的小幅修改到完全重写，所有这些都能达到某种程度上相似的最后得分。少量示例几乎总是从训练集中添加的，这表明这些是任何指标改进的主要驱动因素。鉴于少量示例来自训练集，因此运行最终评估时针对此集和裁判验证集非常重要，以防止过拟合，尽管我怀疑这比传统机器学习中的问题要小。

最后，我们应该保存裁判和基线模块以供将来使用和任何实验的复制。

```py
judge_optimized.save("dspy_modules/optimized_llm_judge",save_program=True)
baseline_judge.save("dspy_modules/baseline_llm_judge",save_program=True)
```

## 9.0 使用优化后的裁判优化生成器

假设我们有一个经过优化的 LLM 裁判，我们对其与 SME 标记数据集的适当对齐有信心，并且我们现在想用它来改进基线生成器提示。这个过程与我们用来构建优化裁判提示的过程类似，只是这次我们使用裁判为我们提供真实标签。如第 6.0 节所述，我们有一个包含 240 个示例的生成器开发集。作为手动提示迭代的组成部分，我们会在这些输入示例上生成生成器提示版本，然后对结果运行裁判并计算准确度。然后我们会审查裁判评论，对提示进行迭代，保存新版本，然后再次尝试。可能需要多次迭代，这就是为什么开发集应该相当小的原因。DSPy 可以通过自动优化帮助我们开始这一过程，代码与第 8.0 节非常相似，唯一的例外是优化指标。

在这个阶段，我们将使用两个 LLM 进行优化：生成器模型，即`gpt-3.5-turbo`，以及裁判模式，即`gemini-1.5-flash`。为了便于操作，我们可以创建一个简单的自定义模块`ModuleWithLM`，它使用特定的 LLM，否则它将使用在最后的`dspy.configure()`调用中定义的任何模型。

```py
import dspy 

optimized_judge = dspy.load(
    "dspy_modules/optimized_llm_judge"
)

# our judge LLM will be gemini 1.5
judge_lm = dspy.LM(
    "gemini/gemini-1.5-flash",
    api_key=secrets["GEMINI_API_KEY"],
    cache=False,
    temperature=0
)

# A helper that runs a module with a specific LM context
class ModuleWithLM(dspy.Module):
    def __init__(self, lm, module):
        super().__init__()
        self.lm = lm
        self.module = module

    def forward(self, **kwargs):
        with dspy.context(lm=self.lm):
            return self.module(**kwargs)

# our new module, which allows us the package a separate llm with a previously
# loaded module
optimized_judge_program = ModuleWithLM(judge_lm, optimized_judge) 
```

在此基础上，我们可以在度量函数内部调用`optimized_judge_program`，这与其在裁判优化过程中执行的`match_judge_metric()`具有相同的目的。

```py
def LLM_judge_metric(example, prediction, trace=None):

    # the input transcript
    transcript_text = str(example.transcript)

    # the output llm response
    output_text = str(prediction.llm_response)

    transcript_text = f"{transcript_text}\nSupport: {output_text}"

    if not transcript_text:
        # Fallback or raise; metric must be deterministic
        return False

    judged = optimized_judge_program(transcript=transcript_text)
    return bool(judged.satisfied) 
```

优化过程本身看起来与我们用于 LLM 裁判的过程相似

```py
import dspy
from dspy_judge.processor.utils import convert_dataset_to_dspy_examples

generate_response = dspy.load(
"dspy_modules/baseline_generation",save_program=True
)

#see section 1.3 for the description of dev_dataset
 dev_dataset_examples = convert_dataset_to_dspy_examples(
    dev_dataset,
    field_mapping = {"transcript":"company_and_transcript"},
    input_field="transcript"
)

# A somewhat arbitrary split into test and train
# The split allows us to check for overfitting on the training examples

optimize_training_data =  dev_dataset_examples[:200]
optimize_validation_data = dev_dataset_examples[200:]

optimizer = dspy.MIPROv2(
    metric=LLM_judge_metric,
    auto="medium",
    init_temperature=1.0,
    seed=101
)

generate_response_optimized = optimizer.compile(
    generate_response,
    trainset=optimize_training_data,
    requires_permission_to_run=False,
)
```

优化完成后，我们可以使用 dspy.Evaluate 生成整体准确度评估。

```py
evaluator = dspy.Evaluate(
    metric=LLM_judge_metric,
    devset=optimize_training_data,
    display_table=True,
    display_progress=True,
    num_threads=24,
)
overall_score_baseline = evaluator(generate_response)
latest_score_optimized = evaluator(generate_response_optimized)
```

在测试期间，我能够通过这种方法将准确率从约 58%提高到约 75%。在我们的小型数据集中，一些这种提升归因于少数几个法官结果从“不满意”切换到“满意”，对于这种变化，检查它们是否在法官在相同数据集上多次运行时的自然变异性内是值得的。在我看来，生成器提示优化最大的优势是它为我们提供了一个新的基于最佳实践的基线提示，比原始基线更具可辩护性，而原始基线通常只是开发者对良好提示的最佳猜测。然后，就为更多的手动迭代、使用优化后的 LLM 法官进行反馈以及进一步与领域专家进行错误分析和边缘情况处理做好了准备。

![图片](img/5054ca25154f5d64fae19abda0edba3b.png)

图 3：使用优化法官测量的基线和优化生成器提示的性能比较。性能提升似乎主要来自 DSPy 优化器添加的少量示例。图像由作者生成。

在这个阶段，仔细查看数据以了解这些微小的质量提升来自何处是明智的。我们可以将基线生成器和优化生成器在会话 ID 上的输出数据集合并，并寻找法官分类发生变化的实例。在分类从`satisfied=False`切换到`satisfied=True`的情况下，我的总体感觉是生成器的回答变得更长、更礼貌，但并没有真正传达更多信息。这并不奇怪，因为对于大多数客户支持问题，生成器没有足够的上下文来添加有意义的细节。此外，由于 LLM 法官倾向于对较长的输出表现出偏见，优化过程似乎推动了生成器朝这个方向发展。

## 10.0 重要学习

本文探讨了使用 DSPy 对 LLM 法官和生成器精炼的提示优化。这个过程有些冗长，但它强制执行了关于开发数据集整理、提示记录和检查过拟合的良好实践。由于 LLM 法官对齐过程通常需要人工在循环中生成标记数据，DSPy 优化过程在这里可以特别强大，因为它减少了与领域专家昂贵的迭代。尽管本文没有真正讨论，但重要的是要注意，由于 LLMs 的自然变异性，优化带来的小幅度性能提升可能并不具有统计学意义。

在设置好裁判后，它也可以被 DSPy 用来优化生成器的提示（从而绕过更多人工标注的需求）。我的感觉是，在某些项目中，即使不是为了其他原因，仅仅是为了自动整理好的少量示例，这也可能值得追求。但不应替代人工评估和错误分析。优化在代币方面也可能相当昂贵，因此应小心避免高额的 API 账单！

感谢您阅读到最后。一如既往，欢迎提供反馈，如果您尝试过类似的提示工程框架，我非常想了解您的体验！我也很想知道将此框架扩展到使用 ReACT 或提示链来完成更大任务的复杂系统的优缺点。

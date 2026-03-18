# GPT-4o 中的可避免和不可避免随机性

> 原文：[`towardsdatascience.com/avoidable-and-unavoidable-randomness-in-gpt-4o/`](https://towardsdatascience.com/avoidable-and-unavoidable-randomness-in-gpt-4o/)

当然，GPT-4o 的输出中存在随机性。毕竟，在选择每个标记时，模型是从一个概率分布中进行采样的。但我没有理解的是，那些概率本身并不是确定的。即使有一致的提示、固定的种子和温度设置为零，GPT-4o 仍然引入了微妙而令人沮丧的随机性。

对于这个问题，没有解决办法，而且即使 OpenAI 想要解决这个问题，也可能无法解决，这样我们就可以清楚地了解这篇文章的方向。在这个过程中，我们将检查 GPT-4o 输出中所有随机性的来源，这将需要我们将采样过程分解到低级别。我们将指出问题——概率在变化——并批判性地审查 OpenAI 关于确定性的官方指南。

首先，让我们谈谈为什么确定性很重要。确定性意味着相同的输入总是产生相同的输出，就像一个数学函数。虽然 LLM 的创造力通常是可取的，但确定性具有至关重要的作用：研究人员需要它来进行可重复的实验，开发者需要它来验证报告的结果，提示工程师需要它来调试他们的更改。没有它，你只能猜测不同的输出是由于你的调整还是随机数生成器的情绪波动。

## 抛硬币

我们将保持这里的操作极其简单，并使用以下内容提示 GPT-4o 的最新版本（API 中的`gpt-4o-2024-08-06`）：

` 抛硬币。只返回正面或反面。`

使用 LLM 抛硬币本身就是一个有趣的话题（例如，参见参考文献中的 Van Koevering & Kleinberg，2024），但在这里，我们将使用它作为一个简单的二元问题来探索确定性，或者缺乏确定性。

这是我们的第一次尝试。

```py
import os
from openai import OpenAI
client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))

prompt = 'Flip a coin. Return Heads or Tails only.'

response = client.chat.completions.create(
    model='gpt-4o-2024-08-06',
    messages=[{'role': 'user', 'content': prompt}],
)

print(response.choices[0].message.content)
```

运行代码给了我“正面”。也许你会得到“反面”，或者如果你真的很幸运，可能会得到一些更有趣的结果。

代码首先使用环境变量`OPENAI_API_KEY`（为了避免在这里共享计费凭证）初始化一个 OpenAI 客户端。主要操作发生在`client.chat.completions.create`，在那里我们指定要使用的模型，并将提示（作为名为 messages 的非常简单的对话的一部分）发送到服务器。我们从服务器返回一个名为 response 的对象。这个对象包含大量信息，如下所示，因此我们需要深入挖掘以提取 GPT-4o 对消息的实际响应，即`response.choices[0].message.content`。

```py
>>> response
ChatCompletion(id='chatcmpl-B48EqZBLfUWtp9H7cwnchGTJbBDwr', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='Heads', refusal=None, role='assistant', audio=None, function_call=None, tool_calls=None))], created=1740324680, model='gpt-4o-2024-08-06', object='chat.completion', service_tier='default', system_fingerprint='fp_eb9dce56a8', usage=CompletionUsage(completion_tokens=2, prompt_tokens=18, total_tokens=20, completion_tokens_details=CompletionTokensDetails(accepted_prediction_tokens=0, audio_tokens=0, reasoning_tokens=0, rejected_prediction_tokens=0), prompt_tokens_details=PromptTokensDetails(audio_tokens=0, cached_tokens=0)))
```

> 现在让我们抛硬币十次。如果这是一个真实、公平的硬币，当然，由于大数定律，随着时间的推移，我们会期望头和尾大致相等。但 GPT-4o 的硬币并不完全是这样工作的。
> 
> ```py
> import os
> from openai import OpenAI
> client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
> 
> prompt = 'Flip a coin. Return Heads or Tails only.'
> 
> for _ in range(10):
>     response = client.chat.completions.create(
>         model='gpt-4o-2024-08-06',
>         messages=[{'role': 'user', 'content': prompt}],
>     )
>     print(response.choices[0].message.content)
> ```
> 
> 运行此代码给了我以下输出，尽管你可能会得到不同的输出，当然。
> 
> ```py
> Heads
> Heads
> Heads
> Heads
> Heads
> Heads
> Tails
> Heads
> Heads
> Heads
> ```
> 
> > GPT-4o 的硬币显然是有偏的，但人类也是如此。Bar-Hillel、Peer 和 Acquisti（2014）发现，人们在抛想象中的硬币时 80%的时间选择“头”。也许 GPT-4o 从我们这里学到了这一点。但无论原因如何，我们只是用这个简单的例子来探索确定性。
> > 
> > ## GPT-4o 的硬币有多偏？
> > ## 
> > 假设我们想知道 GPT-4o 硬币抛掷中“头”出现的精确百分比。
> > 
> > 与抛一百万次（虽然昂贵）的明显方法相比，有一种更聪明的方法。对于具有少量可能答案的分类任务，我们可以提取标记概率而不是生成完整的响应。有了正确的提示，第一个标记就包含了所有必要的信息，这使得这些 API 调用非常便宜：大约每美元 30,000 次调用，因为每次调用只需要 18（缓存的）输入标记和 1 个输出标记。
> > 
> > OpenAI 为我们提供（自然）对数概率。在代码中，这些被称为 logprobs，我们通过指数运算将它们转换为常规概率。（我们很快会讨论温度，但请注意，直接像这样指数运算 logprobs 对应于温度设置为 1.0，这是我们在这篇文章中计算概率的方式）。OpenAI 允许我们请求前 20 个最可能标记的对数概率，所以我们就是这样做的。
> > 
> > ```py
> > import os
> > import math
> > from openai import OpenAI
> > from tabulate import tabulate
> > 
> > client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
> > 
> > prompt = 'Flip a coin. Return Heads or Tails only.'
> > 
> > response = client.chat.completions.create(
> >     model='gpt-4o-2024-08-06',
> >     max_tokens=1,
> >     logprobs=True,
> >     top_logprobs=20,
> >     messages=[{'role': 'user', 'content': prompt}],
> > )
> > 
> > logprobs_list = response.choices[0].logprobs.content[0].top_logprobs
> > 
> > data = []
> > total_pct = 0.0
> > 
> > for logprob_entry in logprobs_list:
> >     token = logprob_entry.token
> >     logprob = logprob_entry.logprob
> >     pct = math.exp(logprob) * 100  # Convert logprob to a percentage
> >     total_pct += pct
> >     data.append([token, logprob, pct])
> > 
> > print(
> >     tabulate(
> >         data,
> >         headers=["Token", "Log Probability", "Percentage (%)"],
> >         tablefmt="github",
> >         floatfmt=("s", ".10f", ".10f")
> >     )
> > )
> > print(f"\nTotal probabilities: {total_pct:.6f}%")
> > ```
> > 
> > 如果您运行此代码，您将得到类似以下输出，但实际数字会*变化*。
> > 
> > ```py
> > | Token     |   Log Probability |   Percentage (%) |
> > |-----------|-------------------|------------------|
> > | Heads     |     -0.0380541235 |    96.2660836887 |
> > | T         |     -3.2880542278 |     3.7326407467 |
> > | Sure      |    -12.5380544662 |     0.0003587502 |
> > | Head      |    -12.7880544662 |     0.0002793949 |
> > | Tail      |    -13.2880544662 |     0.0001694616 |
> > | Certainly |    -13.5380544662 |     0.0001319768 |
> > | "T        |    -14.2880544662 |     0.0000623414 |
> > | I'm       |    -14.5380544662 |     0.0000485516 |
> > | heads     |    -14.5380544662 |     0.0000485516 |
> > | Heads     |    -14.9130544662 |     0.0000333690 |
> > | "         |    -15.1630544662 |     0.0000259878 |
> > | _heads    |    -15.1630544662 |     0.0000259878 |
> > | tails     |    -15.5380544662 |     0.0000178611 |
> > | HEAD      |    -15.7880544662 |     0.0000139103 |
> > | TAIL      |    -16.2880535126 |     0.0000084370 |
> > | T         |    -16.7880535126 |     0.0000051173 |
> > | ```  |  -16.7880535126 |  0.0000051173 |
> > 
> > | 这里  |  -16.9130535126 |  0.0000045160 |
> > | --- | --- | --- |
> > | 我  |  -17.2880535126 |  0.0000031038 |
> > | 是  |  -17.2880535126 |  0.0000031038 |
> > 
> > 总概率：99.999970%
> > 
> > ```py
> > 
> > > Looking at these probabilities, we see Heads at ≈96% and T at ≈4%. Our prompt is doing pretty well at constraining the model’s responses. Why `T` and not `Tails`? This is the tokenizer splitting `Tails` into `T` + `ails`, while keeping `Heads` as one piece, as we can see in this Python session:
> > > 
> > > ```
> > 
> > > >>> 导入 tiktoken
> > > 
> > > >>> encoding = tiktoken.encoding_for_model("gpt-4o-2024-08-06")
> > > 
> > > >>> encoding.encode('Tails')
> > > 
> > > [51, 2196]
> > > 
> > > >>> encoding.decode([51])
> > > 
> > > 'T'
> > > 
> > > >>> encoding.encode('Heads')
> > > 
> > > [181043]
> > > 
> > > ```py
> > > 
> > > ## These probabilities are not deterministic
> > > 
> > > Run the code to display the probabilities for the top 20 tokens again, and you’ll likely get different numbers. Here’s what I got on a second running.
> > > 
> > > ```
> > > 
> > > | 标记  |  对数概率 |  百分比 (%) |
> > > | --- | --- | --- |
> > > 
> > > |-----------|-------------------|------------------|
> > > 
> > > | 头  |  -0.0110520627 |  98.9008786933 |
> > > | --- | --- | --- |
> > > | T  |  -4.5110521317 |  1.0986894433 |
> > > | 当然  |  -14.0110521317 |  0.0000822389 |
> > > | 头  |  -14.2610521317 |  0.0000640477 |
> > > | 当然  |  -14.2610521317 |  0.0000640477 |
> > > | 尾  |  -14.3860521317 |  0.0000565219 |
> > > | heads  |  -15.3860521317 |  0.0000207933 |
> > > | 头  |  -15.5110521317 |  0.0000183500 |
> > > 
> > > | ```py       |    -15.5110521317 |     0.0000183500 |
> > > | _heads    |    -15.6360521317 |     0.0000161938 |
> > > | tails     |    -15.6360521317 |     0.0000161938 |
> > > | I'm       |    -15.8860521317 |     0.0000126117 |
> > > | "T        |    -15.8860521317 |     0.0000126117 |
> > > | As        |    -16.3860511780 |     0.0000076494 |
> > > | "         |    -16.5110511780 |     0.0000067506 |
> > > | HEAD      |    -16.6360511780 |     0.0000059574 |
> > > | TAIL      |    -16.7610511780 |     0.0000052574 |
> > > | Here's    |    -16.7610511780 |     0.0000052574 |
> > > | ``        |    -17.1360511780 |     0.0000036133 |
> > > | T         |    -17.6360511780 |     0.0000021916 |
> > > 
> > > Total probabilities: 99.999987%
> > > ```
> > > 
> > > > 在他们的[食谱](https://cookbook.openai.com/examples/reproducible_outputs_with_the_seed_parameter)中，OpenAI 提供了以下关于接收“大部分相同”输出的建议：
> > > > 
> > > > > 如果在您的请求中`seed`、请求参数和`system_fingerprint`都匹配，那么模型输出将大部分是相同的。即使请求参数和`system_fingerprint`匹配，由于我们模型固有的非确定性，响应也可能有所不同。
> > > > > 
> > > > 他们还在其文档的“可重复输出部分”（[reproducible outputs section](https://platform.openai.com/docs/advanced-usage/reproducible-outputs)）中给出了“大部分相同”的建议。
> > > > 
> > > > 可能影响随机性的请求参数是`temperature`和`seed`。OpenAI 还建议我们跟踪`system_fingerprint`，因为这里的差异可能会导致输出差异。我们将在下面逐一检查这些参数，但剧透一下：它们都无法解决这个问题，甚至无法解释这种非确定性。
> > > > 
> > > > ## 温度，以及为什么它无法解决这个问题
> > > > ## 
> > > > 温度控制模型响应的随机程度。低温（<0.5）使其变得机械和可预测，中等温度（0.7–1.3）允许一些创造性，而高温（>1.5）会产生无意义的文本。温度通常被称为“创造性参数”，但这是一种过度简化的说法。在他们的分析中，Peeperkorn、Kouwenhoven、Brown 和 Jordanous（2024）从创造性的四个维度评估了 LLM 的输出：新颖性（原创性）、连贯性（逻辑一致性）、凝聚力（文本的流畅性）和典型性（与预期模式的契合度）。他们观察到：
> > > > 
> > > > > 温度与新颖性呈弱相关，并且不出所料，与不连贯性中度相关，但与凝聚力和典型性都没有关系。
> > > > > 
> > > > 但是，这与抛硬币无关。在底层，对数概率在重新归一化和指数化以转换为概率之前会被除以温度。这产生了一个非线性效应：`temperature=0.5`平方了概率，使可能的标记占主导地位，而`temperature=2.0`则应用平方根，使分布平坦化。
> > > > 
> > > > 那么`temperature=0.0`呢？模型不是通过除以零来破坏数学，而是简单地选择概率最高的标记。听起来是确定性的，对吧？但并不完全是这样。这里的关键是：温度只有在计算了对数概率之后，在我们将它们转换为概率时才会发挥作用。
> > > > 
> > > > 总结：**如果对数概率不是确定性的，将温度设置为 0.0 不会使模型变得确定性**。
> > > > 
> > > > 事实上，因为我们只是直接请求模型提供原始的对数概率，而不是生成完整的响应，所以在我们的代码中温度设置根本不起作用。
> > > > 
> > > > ## 种子，以及为什么它们无法解决这个问题
> > > > ## 
> > > > 在使用温度来计算概率之后，模型会从这些概率中采样以选择下一个标记。OpenAI 通过允许我们为随机数生成器设置`seed`参数，给我们提供了一点点对采样过程的控制。在一个理想的世界里，设置种子会在任何温度下给我们确定性。但是，种子只会影响采样，而不会影响采样之前的对数概率。
> > > > 
> > > > 总结：**如果对数概率不是确定性的，设置种子不会使模型变得确定性**。
> > > > 
> > > > 事实上，`seed`只有在非零温度下才有意义。当`temperature=0.0`时，模型总是选择最高概率的 token，而不管 seed 是多少。再次强调，由于我们只是直接请求模型的原始 logprobs 而不是采样，这两个设置都不能帮助我们实现决定论。
> > > > 
> > > > ## 系统指纹，我们的最后希望
> > > > ## 
> > > > `system_fingerprint`识别 OpenAI 后端当前模型权重、基础设施和配置选项的组合。至少，这是 OpenAI 告诉我们的。系统指纹的变化确实可能解释 logprobs 的变化。但事实并非如此，正如我们下面将要验证的。
> > > > 
> > > > ## 没有什么能让你获得决定论
> > > > ## 
> > > > 让我们确认我们一直在努力构建的内容。我们将运行 10 次相同的请求，并确保所有安全措施都到位。即使这些参数*不应该*对我们正在做的事情产生影响，但你永远不能太过安全，所以我们将设置`temperature=0.0`和`seed=42`。为了查看基础设施差异是否解释了我们的不同 logprobs，我们将打印`system_fingerprint`。以下是代码：
> > > > 
> > > > ```py
> > > > import os
> > > > import math
> > > > from openai import OpenAI
> > > > from tabulate import tabulate
> > > > from tqdm import tqdm
> > > > 
> > > > client = OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
> > > > 
> > > > prompt = 'Flip a coin. Return Heads or Tails only.'
> > > > 
> > > > data = []
> > > > 
> > > > for _ in tqdm(range(10), desc='Generating responses'):
> > > >     response = client.chat.completions.create(
> > > >         model='gpt-4o-2024-08-06',
> > > >         temperature=0.0,
> > > >         seed=42,
> > > >         max_tokens=1,
> > > >         logprobs=True,
> > > >         top_logprobs=20,
> > > >         messages=[{'role': 'user', 'content': prompt}],
> > > >     )
> > > > 
> > > >     fingerprint = response.system_fingerprint
> > > >     logprobs_list = response.choices[0].logprobs.content[0].top_logprobs
> > > >     heads_logprob = next(
> > > >         entry.logprob for entry in logprobs_list if entry.token == 'Heads'
> > > >     )
> > > >     pct = math.exp(heads_logprob) * 100
> > > >     data.append([fingerprint, heads_logprob, f"{pct:.10f}%"])
> > > > 
> > > > headers = ["Fingerprint", "Logprob", "Probability"]
> > > > print(tabulate(data, headers=headers, tablefmt="pipe"))
> > > > ```
> > > > 
> > > > 运行 10 次，以下是 token Heads 的 logprobs 和概率：
> > > > 
> > > > ```py
> > > > | Fingerprint   |    Logprob | Probability    |
> > > > |---------------|------------|----------------|
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.160339  | 85.1854886858% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0110521 | 98.9008786933% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > | fp_f9f4fb6dbf | -0.0380541 | 96.2660836887% |
> > > > ```
> > > > 
> > > > ## 混合专家使决定论成为不可能
> > > > ## 
> > > > OpenAI 对于 GPT-4o 背后的架构决不开放。然而，普遍认为 GPT-4o 使用混合专家（MoE）架构，具有 8 个或 16 个专家。
> > > > 
> > > > 根据谷歌 DeepMind 研究人员 Puigcerver、Riquelme、Mustafa 和 Houlsby 的一篇论文（感谢 OpenAI 论坛上的用户 elmstedt[用户 elmstedt 在 OpenAI 论坛上的帖子](https://community.openai.com/t/why-the-api-output-is-inconsistent-even-after-the-temperature-is-set-to-0/329541/9)），混合专家架构可能会添加一个不可避免的非决定论级别：
> > > > 
> > > > > 在容量限制下，所有稀疏 MoE 方法都以固定大小的组路由 token，并在组内强制（或鼓励）平衡。当组包含来自不同序列或输入的 token 时，这些 token 会竞争专家缓冲区中的可用位置。因此，模型在序列级别上不再是决定论的，但在批量级别上是。
> > > > > 
> > > > 换句话说，当你的提示（如上所述，是一个*token 序列*）到达 OpenAI 的服务器时，它会被与一组其他提示一起批量处理（OpenAI 没有公开其他提示的数量）。然后，批量中的每个提示都会被路由到模型内的一个“专家”。然而，由于可以路由到同一专家的提示数量有限，你的提示被路由到的专家将取决于批量中的所有其他提示。
> > > > 
> > > > 这种“竞争”专家的行为引入了完全超出我们控制的现实世界随机性。
> > > > 
> > > > ## 混合专家之外的非决定论
> > > > ## 
> > > > 虽然非决定论可能是现实世界混合专家模型固有的，但这似乎并不是 OpenAI 模型中非决定论的唯一来源。
> > > > 
> > > > 通过对我们的代码进行一些修改（切换到`gpt-3.5-turbo-0125`，寻找标记`He`，因为 GPT-3.5 的标记器将“正面”分割得不同，并且忽略 system_fingerprint，因为该模型没有它），我们发现 GPT-3.5-turbo**也**表现出非确定性的对数概率：
> > > > 
> > > > ```py
> > > > |     Logprob | Probability    |
> > > > |-------------|----------------|
> > > > | -0.00278289 | 99.7220983436% |
> > > > | -0.00415331 | 99.5855302068% |
> > > > | -0.00258838 | 99.7414961980% |
> > > > | -0.00204034 | 99.7961735289% |
> > > > | -0.00240277 | 99.7600117933% |
> > > > | -0.00204034 | 99.7961735289% |
> > > > | -0.00204034 | 99.7961735289% |
> > > > | -0.00258838 | 99.7414961980% |
> > > > | -0.00351419 | 99.6491976144% |
> > > > | -0.00201214 | 99.7989878007% |
> > > > ```
> > > > 
> > > > > 没有人声称 GPT-3.5-turbo 使用的是专家混合架构。因此，除了专家混合之外，还必须有其他因素导致这种非确定性。
> > > > > 
> > > > > ## 10,000 次 GPT-4o 抛硬币概率告诉我们什么
> > > > > ## 
> > > > > 为了更好地理解这种非确定性的模式和幅度，我使用 GPT-4o 进行了更广泛的实验，进行了 10,000 次“抛硬币”实验，同时记录了每次实验中分配给“正面”的概率。
> > > > > 
> > > > > 结果揭示了一些令人着迷的事情。在 10,000 次具有相同参数的 API 调用中，GPT-4o 不仅产生了几个不同的概率值，而且产生了**42 个不同的概率值**。如果混合专家假设是 GPT-4o 中非确定性的完整解释，我们可能会期望每个专家都有一个独特的概率值。但 GPT-4o 被认为有 8 个或 16 个专家，**而不是 42 个**。
> > > > > 
> > > > > 在下面的输出中，我将这些概率进行了聚类，确保每个聚类与其他聚类之间隔开 0.01（作为一个原始百分比）。这把输出分成了**12 个聚类**。
> > > > > 
> > > > > ```py
> > > > > Probability          Count           Fingerprints
> > > > > ------------------------------------------------------------------
> > > > > 85.1854379113%       5               fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 85.1854455275%       74              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 85.1854886858%       180             fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > ------------------------------------------------------------------
> > > > > 88.0662448207%       31              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 88.0678628883%       2               fp_f9f4fb6dbf
> > > > > ------------------------------------------------------------------
> > > > > 92.3997629747%       1               fp_eb9dce56a8
> > > > > 92.3997733012%       4               fp_eb9dce56a8
> > > > > 92.3997836277%       3               fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 92.4128943690%       1               fp_f9f4fb6dbf
> > > > > 92.4129143363%       21              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 92.4129246643%       8               fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > ------------------------------------------------------------------
> > > > > 93.9906837191%       4               fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 95.2569999350%       36              fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 96.2660836887%       3391            fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 96.2661285161%       2636            fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > ------------------------------------------------------------------
> > > > > 97.0674551052%       1               fp_eb9dce56a8
> > > > > 97.0674778863%       3               fp_eb9dce56a8
> > > > > 97.0675003058%       4               fp_eb9dce56a8
> > > > > 97.0675116963%       1               fp_eb9dce56a8
> > > > > 97.0680739932%       19              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 97.0681293191%       6               fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 97.0681521003%       74              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 97.0682421405%       4               fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 97.7008960695%       1               fp_f9f4fb6dbf
> > > > > 97.7011122645%       3               fp_eb9dce56a8
> > > > > 97.7011462953%       3               fp_eb9dce56a8
> > > > > 97.7018178132%       1               fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 98.2006069902%       426             fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.2006876548%       6               fp_f9f4fb6dbf
> > > > > 98.2007107019%       1               fp_eb9dce56a8
> > > > > 98.2009525133%       5               fp_eb9dce56a8
> > > > > 98.2009751945%       1               fp_eb9dce56a8
> > > > > 98.2009867181%       1               fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 98.5930987656%       3               fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.5931104270%       235             fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.5931222721%       4               fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.5931340253%       9               fp_eb9dce56a8
> > > > > 98.5931571644%       159             fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.5931805790%       384             fp_eb9dce56a8
> > > > > ------------------------------------------------------------------
> > > > > 98.9008436920%       95              fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.9008550214%       362             fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > 98.9008786933%       1792            fp_eb9dce56a8, fp_f9f4fb6dbf
> > > > > ```
> > > > > 
> > > > > > （以 0.001 为阈值，有 13 个聚类，以 0.0001 为阈值，有 17 个聚类。）
> > > > > > 
> > > > > > 如上图所示，这些众多结果不能由`system_fingerprint`值来解释。在所有 10,000 次调用中，我只收到了两种不同的系统指纹：4488 次结果带有`fp_f9f4fb6dbf`，5512 次带有`fp_eb9dce56a8`，而且大部分情况下，这两个系统指纹返回了相同的概率集，而不是每个指纹产生其自己的独特概率集。
> > > > > > 
> > > > > > 它**可能**是这 12 个概率聚类代表了**12 个不同的专家**。即使假设如此，聚类内的变化仍然令人困惑。这些看起来不太可能是简单的舍入误差，因为它们太系统和一致了。以大约 96.266%的巨大聚类为例，其中有两个不同的概率代表了超过一半的抛硬币。这两个概率之间的差异，0.0000448274%，非常小但持续存在。
> > > > > > 
> > > > > > ## 结论：非确定性是固有的
> > > > > > ## 
> > > > > > 所有目前可用的非思考型 OpenAI 模型（GPT-4o、GPT-4o-mini 以及两种 GPT-3.5-turbo 版本）返回的对数概率中存在一种潜在的随机性。由于这种非确定性已经嵌入到对数概率中，用户无法绕过它。温度和种子值都没有效果，系统指纹也无法解释这一点。
> > > > > > 
> > > > > > 虽然混合专家架构在本质上引入了专家竞争中的某些随机性，但 GPT-4o 中的非确定性似乎远超于此，GPT-3.5-turbo 中的非确定性也无法用这一点来解释，因为 GPT-3.5-turbo 不是一个混合专家模型。
> > > > > > 
> > > > > > 尽管我们无法再验证这一说法，因为模型不再提供服务，但根据[OpenAI 论坛上的用户 _j](https://medium.com/r/?url=https%3A%2F%2Fcommunity.openai.com%2Ft%2Fnon-deterministic-probabilities-for-first-generated-token-in-chat-completion%2F726074%2F5)的说法，这种行为在 GPT-3 中并未出现：
> > > > > > 
> > > > > > > 这是一个在之前的 GPT-3 AI 模型中未曾见过的症状，在数百次试验以调查采样时，你从未需要怀疑 logprobs 会相同。即使你通过 API 找到了返回相同 logprob 值的 top-2 答案，你也永远不会看到它们的位置交换或返回不同的值。
> > > > > > > 
> > > > > > 这表明导致这种随机性的原因最早出现在 GPT-3.5 或 GPT-3.5-turbo 中。
> > > > > > 
> > > > > > 但无论何时出现，这种非确定性都是理解这些模型的严重障碍。如果你想研究一个模型——它如何泛化，它如何偏置响应，它如何为不同的标记分配概率——你需要一致性。但正如我们所见，即使我们锁定 OpenAI 让我们能触及的每一个旋钮，我们仍然无法得到对最简单问题的答案：**“GPT-4o 说硬币落地为头的概率是多少？”**
> > > > > > 
> > > > > > 更糟糕的是，虽然混合专家解释了部分这种非确定性，但显然还有其他隐藏的随机性来源，我们无法看到、控制或理解。在一个理想的世界里，API 会通过告诉我们哪个专家处理了我们的请求或提供额外的参数来控制这个路由过程，从而提供更多的透明度。没有这种可见性，我们只能猜测这种变异性真正的本质。
> > > > > > 
> > > > > > ## 参考文献
> > > > > > ## 
> > > > > > Bar-Hillel, M., Peer, E., & Acquisti, A. (2014)。 “正面还是反面？”——二元选择中的可达性偏差。实验心理学：学习、记忆和认知，40(6)，1656–1663。[`doi.org/10.1037/xlm0000005`](https://doi.org/10.1037/xlm0000005)。
> > > > > > 
> > > > > > Peeperkorn, M., Kouwenhoven, T., Brown, D., & Jordanous, A. (2024)。温度是大型语言模型的创造力参数吗？在第 15 届国际计算创造力会议（ICCC’24）中。[arXiv:2405.00492](https://arxiv.org/abs/2405.00492)。
> > > > > > 
> > > > > > Puigcerver, J., Riquelme, C., Mustafa, B., & Houlsby, N. (2024). 从稀疏到软混合专家。在第十二届国际学习表示会议（ICLR 2024）中。[`openreview.net/forum?id=jxpsAj7ltE`](https://openreview.net/forum?id=jxpsAj7ltE)。[arXiv:2308.00951](https://arxiv.org/abs/2308.00951)。Van Koevering, K., & Kleinberg, J. (2024)。随机性有多随机？评估 LLM 的硬币翻转的随机性和人性。[arXiv:2406.00092](https://arxiv.org/abs/2406.00092)。

# 工具屏蔽：MCP 遗忘的层级

> 原文：[`towardsdatascience.com/tool-masking-the-layer-mcp-forgot/`](https://towardsdatascience.com/tool-masking-the-layer-mcp-forgot/)

由 Frank Wittkampf 和 Lucas Vieira 撰写

## <mdspan datatext="el1757011845328" class="mdspan-comment">简介</mdspan>

MCP 和类似服务在 AI 连接性方面是一个突破：当我们需要快速且几乎毫不费力地将服务暴露给 LLM 时，这是一个巨大的进步。但这也带来了问题：这是一种自下而上的思维方式。**“嘿，我们为什么不一次性将所有东西都暴露出来呢？”**

**原始 API 暴露存在成本**：直接推送到代理的每个工具表面都会膨胀提示，增加选择熵，并降低执行质量。一个设计良好的 AI 代理从用例开始而不是从技术开始。如果您从头开始设计您的 LLM 调用，您永远不会提供 API 的全表面。您添加了不必要的 token、无关信息、更多失败模式，并总体上降低了质量。经验表明，广泛的工具定义消耗大量的 token 预算：例如，一个 28 参数的工具≈1,633 个 token；37 个工具≈6,218 个 token，这会降低准确性并增加延迟/成本⁶。

在我们的工作中，为最大的科技公司（MSFT、AWS、Databricks 等）构建企业级 AI 解决方案，我们每分钟向我们的 AI 提供商发送数百万个 token，这些**细微之处很重要**。如果您优化工具暴露，这意味着您优化了您的 LLM 执行上下文，这意味着您正在提高质量、准确性、一致性、成本和延迟，**同时进行**。

本文将定义新的概念**工具屏蔽**。许多人可能已经隐式地尝试过这个概念，但迄今为止，这个主题在在线出版物中并未得到很好的探索。工具屏蔽是当前代理堆栈中一个必要且缺失的层级。工具屏蔽塑造了模型在执行前后实际看到的内容，因此您的 AI 代理不仅可以连接，而且实际上可以启用。

因此，总结我们的简介：**使用原始 MCP 会污染您的 LLM 执行**。您如何优化特定代理或任务的工具面向模型的面板？您使用工具屏蔽。这是一个简单的概念，但正如往常一样，魔鬼在细节中。

## MCP 做得好的地方和不好的地方

MCP 做得很好。它是一个开放协议，Anthropic 将其称为“AI 的 USB-C”。一种连接 LLM 应用与外部工具和数据的无摩擦方式。它解决了基础问题：标准化工具、资源和提示的描述、发现和调用方式，无论您是使用 JSON-RPC over stdio 还是通过 HTTP 进行流式传输²。身份验证在传输层³得到妥善处理。这就是为什么您会看到它在从 OpenAI 的 Agents SDK 到 VS Code 中的 Copilot，再到 AWS 指南⁴等各个地方都有应用。**MCP 是真实的，并且应用广泛**。

但是，了解 MCP 不做什么同样重要——这正是差距所在。**MCP 的焦点是上下文交换**。它不关心你的应用程序或代理如何实际*使用*你传递的上下文，或者你如何根据代理或任务管理并塑造该上下文。它暴露了完整的工具表面，但不对其进行塑造或过滤以优化质量或相关性。根据架构文档，MCP“仅关注上下文交换的协议。它不规定 AI 应用程序如何使用 LLM 或管理提供上下文²。”你得到一个可发现的目录和模式，但没有在*协议*中内置的机制来优化上下文的呈现方式。

注意：一些 SDK 现在添加了可选的过滤功能——例如，OpenAI 的 Agents SDK 支持静态/动态 MCP 工具过滤⁵。这是朝着正确方向迈出的一步，但仍然留下了太多空间。

1. [Anthropic MCP 概述](https://docs.anthropic.com/en/docs/mcp)

2. [模型上下文协议——架构](https://modelcontextprotocol.io/docs/learn/architecture)

3. [MCP 规范——授权](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization)

4. [OpenAI Agents SDK (MCP)](https://openai.github.io/openai-agents-python/mcp); [VS Code MCP GA](https://github.blog/changelog/2025-07-14-model-context-protocol-mcp-support-in-vs-code-is-generally-available); [AWS——解锁 MCP](https://aws.amazon.com/blogs/machine-learning/unlocking-the-power-of-model-context-protocol-mcp-on-aws)

5. [GitHub——PR #861 (MCP 工具过滤)](https://github.com/openai/openai-agents-python/pull/861)

6. [Medium——一个 AI 代理可以有多少工具/函数？](https://achan2013.medium.com/how-many-tools-functions-can-an-ai-agent-have-21e0a82b7847)

## 实践中的问题

为了说明这一点，让我们以（非官方的）Yahoo Finance API 为例。像许多 API 一样，它返回一个包含数十个指标的巨大 JSON 对象。对于分析来说很强大，但当你的代理只需要检索一个或两个关键数字时，就会感到压倒性。为了说明我的观点，这里是一个代理在调用 API 时可能收到的片段：

```py
yahooResponse = {
  "quoteResponse": {
    "result": [
      {
        "symbol": "AAPL",
        "regularMarketPrice": 172.19,
        "marketCap": ...,
        …
        …
        # … roughly 100 other fields

# Other fields: regularMarketChangePercent, currency, marketState, exchange,
  fiftyTwoWeekHigh/Low, trailingPE, forwardPE, earningsDate, 
  incomeStatementHistory, financialData (with revenue, grossMargins, etc.), 
  summaryProfile, etc.
```

对于一个代理来说，获取 100 个字段的数据，包括其他工具输出，是令人压倒性的：无关数据、膨胀的提示和浪费的标记。很明显，**随着工具数量和模式大小的增加，准确性会下降**；研究人员已经表明，随着工具集的扩大，检索和调用可靠性会急剧下降¹，而且由于上下文长度和延迟限制，将每个工具输入到 LLM 中很快就会变得不切实际²。这显然取决于模型，但随着模型能力的增强，工具需求也在增加。即使是最先进的模型，在大型工具库中有效地选择工具仍然很困难³。

问题不仅限于工具输出。**更重要的是 API 输入模式**。回到我们的例子，对于 Yahoo Finance API，你可以请求任何模块的组合：assetProfile、financialData、price、earningsTrend 以及更多。如果你通过 MCP（或 fastAPI 等）以原始方式向代理暴露此模式，你已大量污染了代理上下文。在巨大规模下，这变得更加具有挑战性；最近的工作指出，在非常大的工具图上运行的 LLMs 需要新的方法，如结构化范围界定或基于图的方法⁴。

工具定义在每次对话回合中都会消耗 token；经验基准显示，大型多参数工具和**大型工具集很快就会耗尽你的提示预算**⁵。没有过滤或重写层，你的 AI 代理的准确性和效率会下降⁶。

1.  [Re-Invoke：零样本工具检索的重写](https://arxiv.org/abs/2408.01875) *“随着工具集大小的增加，识别最相关的工具…成为了一个关键瓶颈，阻碍了可靠工具的利用。”*

1.  [面向 LLMs 的完整性导向工具检索](https://arxiv.org/abs/2405.16089) *“…由于长度限制和延迟约束，将所有工具输入到 LLMs 中是不切实际的。”*

1.  [决定是否使用工具以及使用哪个工具](https://arxiv.org/abs/2310.03128) *“…大多数[LLMs]仍然难以有效地选择工具…”*

1.  [ToolNet：通过工具图将 LLMs 与大量工具连接起来](https://arxiv.org/abs/2403.00839) *“LLMs 在大型工具库上操作仍然具有挑战性，”*这促使基于图的范围界定。

1.  [一个 AI 代理可以有多少工具/函数？](https://achan2013.medium.com/how-many-tools-functions-can-an-ai-agent-has-21e0a82b7847)（2025 年 2 月）：报告称一个具有 28 个参数的工具消耗了 1,633 个 token；一组 37 个工具消耗了 6,218 个 token。

1.  [LLMs 的工具检索基准测试（ToolRet）](https://arxiv.org/abs/2503.01763) 大规模基准测试表明，即使是强大的 IR 模型，工具检索也很困难。

这里是一个示例工具定义，如果你不为其创建自定义工具而直接暴露原始 API（为了可读性已缩短）：

```py
yahooFinanceTool = {
 "name": "yahoo.quote_summary",
 "parameters": {
    "type": "object",
    "properties": {
      "symbol": {"type": "string"},
      "modules": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Select any of: assetProfile, financialData, price, 
          earningsHistory, incomeStatementHistory, balanceSheetHistory, 
          cashflowStatementHistory, summaryDetail, quoteType, 
          recommendationTrend, secFilings, fundOwnership, 
          … (and dozens more modules)"
      },
    # … plus more parameters: region, lang, overrides, filters, etc.
    },
  "required": ["symbol"]
  }
}
```

## 修复问题

这里是真正的突破：*通过工具掩码，你控制着呈现给代理的表面*。你不必被迫暴露整个 API，也不必为每个新的用例重新编码你的集成。

想让代理只获取最新的股票报价？构建一个只展示该操作的简单工具的掩码。

需要支持多个不同的任务，比如获取报价、仅提取收入，或者可能在价格类型之间切换？你可以设计**多个窄工具**，每个工具都拥有自己覆盖在相同底层工具处理器上的掩码。

或者，你可以将相关的操作组合成一个单一的工具，并为代理提供一个明确的切换或枚举，无论哪种接口都符合代理的上下文和任务。

如果代理只能看到非常简单、专为特定目的设计的工具，比如这些，岂不是更好？

```py
# Simple Tool: Get latest price and market cap
fetchPriceAndCap = {
  "name": "get_price_and_marketcap",
  "parameters": {
    "type": "object",
    "properties": {
    "symbol": {"type": "string"}
   },
   "required": ["symbol"]
  }
}
```

或者

```py
# Simple Tool 2: Get company revenue only
fetchRevenue = {
  "name": "get_revenue",
  "parameters": {
    "type": "object",
    "properties": {
      "symbol": {"type": "string"}
    },
    "required": ["symbol"]
  }  
}
```

基础代码使用相同的处理器。无需重复逻辑或迫使代理对整个模块表面进行推理。只需为不同的工作提供不同的屏蔽——无需模块列表，无需膨胀，无需重新编码*。

* 这与使用仅必要工具、最小化参数以及尽可能动态激活给定交互的工具的指导方针相一致。

## 工具屏蔽的力量

重点：**工具屏蔽不仅仅是隐藏复杂性**。它是关于*为当前任务设计正确的代理界面*。

+   你可以将 API 作为单一工具或多个工具暴露。

+   你可以调整所需的、可选的，甚至固定的（硬编码的值）。

+   你可以根据角色、上下文或业务逻辑向不同的代理展示不同的屏蔽。

+   你可以在任何时候重构表面——而不需要重写处理器或后端代码。

这不仅仅是技术卫生，而是一个战略设计决策。它让你能够发布更干净、更精简、更健壮的代理，这些代理正好完成所需的工作，不多也不少。

这就是**工具屏蔽**的力量：

+   从一个宽泛、混乱的 API 表面开始

+   需要定义尽可能多的窄屏蔽，每个代理用例一个。

+   只向模型展示重要的内容（而不是更多）。

结果？更小的提示，更快的响应，更少的误操作——以及每次都能做对的代理。为什么这在企业规模上如此重要？

+   **选择熵**：当模型被选项过载时，它更有可能出错或选择错误的字段。

+   **性能**：额外的标记意味着更高的成本、更多的延迟、更低的性能、更低的准确性、更低的一致性

+   **企业规模**：当你每分钟发送数百万个标记时，小的低效很快就会累积。精确度很重要。容错性较低。（*大型工具输出也可能在历史中回响并增加支出*）¹

1. [MCP 的所有错误](https://blog.sshh.io/p/everything-wrong-with-mcp)

## 解决方案

强健的工具屏蔽的核心在于关注点的清晰分离。

首先，你有**工具处理器**——这是原始集成，无论是第三方 API、内部服务还是直接函数调用。处理器的任务仅仅是暴露*完整*的能力表面，包括其所有的功能和复杂性。

接下来是**工具屏蔽**。屏蔽定义了*模型界面*——一个狭窄的架构，定制输入和输出，以及针对代理用例或角色的合理默认值。这就是底层工具的广泛、混乱表面被缩减到正好所需的内容（而不是更多）。

在中间位置是**工具服务**。这是应用屏蔽、验证输入、将代理请求转换为处理器调用，并在返回给模型之前验证或清理响应的调解者。

![](img/a83afd896f6860932a28cf754ad53883.png)

*^(高级概述——工具屏蔽)*

理想情况下，你将工具掩码存储和管理在与所有其他代理/系统提示相同的地方，因为在实践中，向 LLM 展示工具是一种提示工程。

让我们回顾一个实际的工具掩码示例。我们过去几年对工具掩码的定义已经发生了变化。最初是一个简单的过滤器，后来发展成为一个完整的企业服务，被世界上最大的科技公司使用。

1. 初始阶段（在 2023 年），我们开始使用简单的输入/输出适配器，但随着我们在多个公司和许多用例中工作，它已经发展成为一个完整的提示工程界面。

### 工具掩码示例

```py
tool_name: stock_price
description: Retrieve the latest market price for a stock symbol via Yahoo Finance.

handler_name: yahoo_api

handler_input_template:
  session_id: "{{ context.session_id }}"
  symbol: "{{ input.symbol }}"
  modules:
    - price

output_template: |
  {
    "data": {
      "symbol": "{{ result.quoteResponse.result[0].symbol }}",
      "market_price": "{{ result.quoteResponse.result[0].regularMarketPrice }}",
      "currency": "{{ result.quoteResponse.result[0].currency }}"
    }
  }

input_schema:
  type: object
  properties:
    symbol:
      type: string
      description: "The stock ticker symbol (e.g., AAPL, MSFT)"
  required: ["symbol"]

custom_validation_template: |
  {% set symbol_str = input.symbol | string %}
  {% if not symbol_str or symbol_str|length > 6 or symbol_str != symbol_str.upper() %}
      { "success": false, "error": "Symbol must be 1–6 uppercase letters." }
  {% endif %} 
```

上述示例应该已经足够说明问题，但让我们强调几个特点：

+   掩码将输入（由 AI 代理提供）转换为处理程序输入（API 将接收的内容）

+   这个特定工具的处理程序是一个 API，它也可以是任何其他服务。该服务可以在其上添加其他掩码，从而从同一个 API 中提取其他数据

+   掩码允许使用 Jinja*。这允许强大的提示工程

+   如果你想要添加特定的引导，使 AI 代理自我纠正错误，自定义验证非常强大

+   session_id 和模块被硬编码到模板中。AI 代理无法修改这些

*注意：如果你在 nodeJS 环境中做这件事，EJS 也非常适合。*

在这种架构下，你可以灵活地添加、删除或修改工具掩码，而无需触及底层处理程序或代理代码。工具掩码成为了一个“可配置的提示工程”层，支持快速迭代、测试和稳健的角色或用例特定的代理行为。

嘿，这几乎就像一个工具变成了提示...

## 被忽视的提示工程表面

工具是提示。有趣的是，在今天的 AI 博客中，对此的参考很少。一个 LLM 接收文本然后生成文本。工具名称、工具描述以及它们的输入模式都是输入文本的一部分。**工具是提示，只是带有特殊的味道。**

当你的代码调用 LLM 时，模型会读取完整的提示输入，然后决定是否以及如何调用工具¹²³。如果我们得出结论，工具本质上就是提示，那么我希望你在阅读这篇文章时，会有以下的认识：

*工具需要经过提示工程，因此我拥有的任何提示工程技术也应该应用到我的工具上：*

+   *工具是上下文相关的！工具描述应该与提示上下文的其他部分相匹配。*

+   *工具命名非常重要！*

+   *工具输入界面增加了标记和复杂性，因此需要优化。*

+   *同样，对于工具输出界面。*

+   *工具错误响应的框架和措辞很重要，如果提供正确的响应，代理将自我纠正。*

在实践中，我看到许多工程师在代理的主要提示中提供了关于特定工具使用的详细说明。这是一个我们应该质疑的做法。**使用工具的说明应该位于更大的代理提示中，还是与工具一起**？一些工具只需要简短的摘要；而其他工具则受益于更丰富的指导、示例或边缘情况说明，以便模型能够可靠地选择它们并正确格式化参数。通过面具，您可以通过针对每个面具定制工具描述和模式，将相同的底层 API 适应不同的代理和上下文。将此指导与工具界面保持一致，可以稳定合同并避免聊天提示漂移（参见 Anthropic 的*工具使用*和*工具定义的最佳实践*）。当您还指定输出结构时，您可以提高一致性和可解析性¹。**面具使提示工程师可以编辑它，而不是将其埋藏在（Python）代码中**。

在操作层面，我们应该将面具视为工具的可配置提示。实际上，我们建议您将面具存储在与您的提示相同的层中。理想情况下，这是一个支持模板（例如，Jinja）、变量和评估的配置系统。这些概念对于工具面具和常规提示同样适用。此外，我们建议您对其进行版本控制，按代理或角色进行范围划分，并使用这些工具面具来设置默认值、隐藏未使用的参数，或将一个宽泛的处理程序拆分为多个清晰的界面。工具面具还具有安全优势，允许系统提供特定的参数，而不是 LLM。(*独立的评论也突出了无界工具输出的成本/安全风险。这是限制界面的另一个原因⁴.*)

如果做得好，面具可以将提示工程扩展到模型实际作用的工具边界，从而产生更干净的行为和更一致的执行。

1\. [Anthropic — 工具使用概述](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/overview)

2\. [OpenAI — 工具指南](https://platform.openai.com/docs/guides/tools)

3\. [OpenAI 食谱 — 提示指南](https://cookbook.openai.com/examples/gpt4-1_prompting_guide)

4\. [MCP 的所有错误](https://blog.sshh.io/p/everything-wrong-with-mcp)

## 设计模式

几种简单的模式可以覆盖大多数面具需求。从最小的有效界面开始，然后在真正需要时再进行扩展。

+   **模式收缩**：限制参数到任务所需的范围；约束类型和范围；预先填充不变量。

+   **角色范围视图**：向不同的代理或上下文展示不同的面具；相同的处理程序，定制界面。

+   **能力门控**：暴露操作的一个专注子集；将大型工具拆分为单一用途的工具；执行允许列表。

+   **默认参数**：设置智能默认值并隐藏非必要选项以减少令牌和变异性。

+   **系统提供的参数**: 从系统中注入租户、账户、区域或策略值；LLM 无法更改它们，这提高了安全性和一致性。

+   **切换/枚举表面**: 将相关操作组合成一个具有显式枚举或模式的工具；不允许自由文本开关。

+   **类型化输出**: 返回一个小型、严格的模式；标准化单位和键以实现可靠的解析和评估。

+   **渐进式披露**: 首先发送最小掩码；仅在需要时通过新的掩码版本添加可选字段。

+   **验证**: 允许在工具掩码级别进行自定义输入验证；设置建设性的验证响应以引导代理走向正确的方向

## 结论

连接性解决了“什么”。执行是“如何”。像 MCP 这样的服务连接工具。**工具掩码**通过调整模型面对的表面以适应任务和与之一起工作的代理来使它们表现良好。

**从用例出发，而不是从技术出发**。一个处理程序，多个掩码。狭窄的输入、输出，并实验和提示工程师您的工具表面以达到完美。将描述与工具一起保留，而不是埋藏在聊天文本或代码中。将掩码视为可配置的提示，您可以对其进行版本控制、测试，并按代理分配。

如果您暴露原始表面，**您将支付熵的成本**：更多令牌，更慢的延迟，更低的准确性，行为不一致。掩码翻转这一曲线。更小的提示。更快的响应。更高的通过率。更少的误操作。这种方法在企业规模上的影响是累积的。（*甚至 MCP 的倡导者也指出，发现列表包含所有内容，但没有经过筛选，并且代理发送/考虑的数据太多。*）

**那么，我们应该做什么？**

+   在代理和每个广泛 API 之间放置一个掩码层

+   尝试在一个处理程序上使用多个掩码，并定制掩码以查看它对性能的影响

+   将掩码与您的提示一起存储在配置中；进行版本控制和迭代

+   将工具说明移入工具表面，并从系统提示中移除。

+   提供合理的默认值，并隐藏模型不应接触的内容

停止分发大型工具。分发表面。这是 MCP 遗忘的层。将代理从连接转变为**启用**的步骤。

*如果您喜欢这篇文章，请在[LinkedIn](https://www.linkedin.com/in/wittkampf/)上给我们留言！*

* * *

> *关于作者：*
> 
> 卢卡斯和弗兰克在多家公司的 AI 基础设施上紧密合作（并为少数其他公司提供咨询）——从一些最早的多代理团队到 LLM 提供商管理，再到文档处理，再到企业 AI 自动化。我们在 Databook 工作，这是一个为世界上最大的科技公司（MSFT、AWS、Databricks、SalesForce 等）提供前沿 AI 自动化平台的公司，我们通过使用被动/主动/引导 AI 为现实世界、企业生产应用提供一系列解决方案。

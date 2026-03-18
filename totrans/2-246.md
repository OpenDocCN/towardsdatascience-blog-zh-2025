# 智能体、API 和互联网的下一层

> 原文：[`towardsdatascience.com/agents-apis-and-the-next-layer-of-the-internet/`](https://towardsdatascience.com/agents-apis-and-the-next-layer-of-the-internet/)

## 第一部分：<mdspan datatext="el1749877782833" class="mdspan-comment">思想集装箱</mdspan>

有时一个简单的想法会彻底改变一切。集装箱运输不仅优化了物流；它使地球变得扁平，消除了时区，并重写了贸易的经济。在其几何简约中，是一场静悄悄的革命：标准化。

同样，HTML 和 HTTP 并非创造了信息交换——就像装货箱并非创造了贸易——但通过在混乱中强加秩序，它们改变了它。

对于 RESTful API 来说，它们标准化了软件-网络交互，并使服务可编程。网络不仅可浏览，而且可构建——这是自动化、编排和集成的基石，围绕这一理念涌现出整个行业。

现在，“智能体网络”——其中 AI 智能体调用 API 和其他 AI 智能体——需要自己的标准。

这不仅仅是对上一时代的扩展——它是对计算工作方式的转变。

对于智能体-网络交互，已经出现了两种有前景的方法：模型上下文协议 (MCP) 和调用网络。

+   **[模型上下文协议](https://modelcontextprotocol.io/introduction) (MCP):** 一个为跨多个智能体、工具和模型链式推理而设计的通信标准。

+   **[调用网络](https://invoke.network/):** 一个轻量级、开源的框架，允许模型在推理时直接与真实世界的 API 交互——无需编排、后端或智能体注册表。

这篇文章比较了这两种范例——MCP 和调用网络（*披露：我是调用网络的贡献者*）——并认为智能体互操作性不仅需要架构和标准，还需要简单性、无状态和运行时发现。 

* * *

## 第二部分：模型上下文协议：使用同一种语言的智能体

### 起源：从本地工具到共享语言

[**模型上下文协议**](https://modelcontextprotocol.io/introduction) (MCP) 诞生于一个简单而强大的想法：大型语言模型 (LLMs) 应该能够相互交流——并且它们的交互应该是模块化、可组合和可检查的。

它始于 GitHub 和 Twitter 上的 AI 工程师社区——一个松散但充满活力的开发者集体，探索当模型获得智能时会发生什么。早期项目如 OpenAgents 和 LangChain 已经引入了工具的概念：给予 LLMs 对函数的受控访问。但 MCP 将这一想法进一步推进。

MCP 提出了一种标准——一个共享的语法——允许任何代理动态地暴露其功能并接收结构化、可解释的请求。目标：使代理可组合和互操作。不仅仅是使用工具的一个代理，而是代理调用代理，工具调用工具，推理像接力棒一样在模型之间传递。

由 Anthropic 于 2024 年 11 月引入，MCP 不是一个产品。它是一个协议。一个关于代理如何通信的社会契约——就像 HTTP 对于网页一样。

* * *

### MCP 是如何工作的

在其核心，MCP 是一个基于 JSON 的接口描述和调用/响应格式。每个代理（或工具、或模型）通过返回一组结构化函数来宣传其功能——类似于 OpenAPI 模式，但针对 LLM 解释进行了定制。

一个典型的 MCP 交换有三个部分：

1.  **列出功能**代理暴露一组可调用的函数——它们的名称、参数、返回类型和描述。这些可以是真实工具（如 get_weather）或委托给其他代理（如 research_topic）。

1.  **发起调用**另一个模型（或用户）使用定义的格式向该代理发送请求。MCP 保持有效载荷结构化和最小化，避免在重要之处使用模糊的自然语言。

1.  **处理响应**接收代理执行函数（或提示另一个模型），并返回一个结构化响应，通常带有理由或后续上下文。

这听起来很抽象，但在实践中却出奇地优雅。让我们看看一个真实例子——并利用它来揭示 MCP 的优势和局限性。

### 一个工作示例：相互调用的代理

让我们想象两个代理：

+   WeatherAgent：提供天气数据。

+   TripPlannerAgent：计划一次一日游，并通过 MCP 使用 WeatherAgent 来检查天气。

在这个场景中，TripPlannerAgent 没有硬编码的如何获取天气的知识。它只是询问另一个说 MCP 的代理。

* * *

#### 第 1 步：WeatherAgent 描述其功能

```py
{
  "functions": [
    {
      "name": "get_weather",
      "description": "Returns the current weather in a given city",
      "parameters": {
        "type": "object",
        "properties": {
          "city": {
            "type": "string",
            "description": "The city to get weather for"
          }
        },
        "required": ["city"]
      }
    }
  ]
}
```

这个 JSON 模式符合 MCP 规范。任何其他代理都可以内省这个模式，并确切地知道如何调用天气函数。

* * *

#### 第 2 步：TripPlannerAgent 进行结构化调用

```py
{
  "call": {
    "function": "get_weather",
    "arguments": {
      "city": "San Francisco"
    }
  }
}
```

代理不需要知道如何获取天气——它只需要遵循协议。

* * *

#### 第 3 步：WeatherAgent 以结构化数据响应

```py
{
  "response": {
    "result": {
      "temperature": "21°C",
      "condition": "Sunny"
    },
    "explanation": "It’s currently sunny and 21°C in San Francisco."
  }
}
```

TripPlannerAgent 现在可以使用这个结果在自己的逻辑中——也许根据天气建议野餐或博物馆之旅。

* * *

### 这是什么使它成为可能

这个小小的例子展示了几个强大的功能：

✅ **代理组合** — 代理可以将其他代理作为工具调用

✅ **可检查性** — 功能在模式中定义，而不是在散文中

✅ **可重用性** — 代理可以为多个客户端提供服务

✅ **LLM 原生设计** — 响应仍然可以被模型解释

但 MCP 有其局限性——我们将在下一部分探讨。

**何时使用 MCP（以及何时不使用）**

模型上下文协议（MCP）以其简洁性而优雅：一个用于描述工具并在代理之间委派任务的协议。但是，像所有协议一样，它在某些环境中表现出色，在其他环境中则遇到挑战。

### ✅ MCP 的亮点

**1. LLM 到 LLM 通信**MCP 从一开始就是为了支持代理间的调用而设计的。如果你正在构建一个可以相互调用、查询或咨询的 AI 代理网络，MCP 是理想的。每个代理都成为一个服务端点，拥有其他代理可以推理的模式。

**2. 去中心化、模型无关的系统**因为 MCP 只是一个模式约定，它不依赖于任何特定的运行时、框架或模型。你可以使用 OpenAI、Claude 或你本地的 LLM——如果它可以解析 JSON，它就可以使用 MCP。

**3. 多跳规划**当与一个*规划器*代理结合使用时，MCP 特别强大。想象一个中央规划器，通过根据代理的模式动态选择代理来编排工作流程。这可以实现高度模块化和动态的系统。

* * *

### ❌ MCP 的挑战

**1. 没有真正的“运行时”**MCP 是一个协议——不是一个框架。它定义了*接口*，但不是执行引擎。这意味着你需要实现自己的粘合逻辑：

+   认证

+   输入/输出映射

+   路由

+   错误处理

+   重试

+   速率限制

MCP 不会为你管理这些——它只是代理使用的通信语言。

**2. 需要结构化思维**LLMs 喜欢模糊性。MCP 则不然。它迫使开发者（和模型）明确：这是一个工具，这是它的模式，这是如何调用它的。这对于清晰度来说很好——但需要比在 OpenAI 代理上简单地添加 `.tools = [...]` 更多的前期思考。

**3. 工具发现和版本控制**

MCP 仍然处于早期阶段——没有代理的中央注册表，没有版本控制或命名空间的真实系统。在实践中，开发者经常手动传递模式或硬编码引用。

* * *

| **用例** | **你应该使用 MCP 吗？** |
| --- | --- |
| 代理呼叫另一个代理 | ✅ 完美匹配 |
| 构建一个大型、模块化的代理网络 | ✅ 理想 |
| 调用一个 REST API 或 webhook | ❌ 过度 |
| 需要内置路由、OAuth、重试 | ❌ 使用框架 |
| 推理时的工具发现 | ❌ 使用 Invoke 网络 |

正是在这里，**调用网络**介入——不是作为竞争对手，而是作为补充。如果 MCP 是代理的**WebSockets**（点对点、结构化、低级别），那么 Invoke 就像是**HTTP**——为 LLMs 提供一个一次性 API 表面。

* * *

## 第三部分：调用网络

### LLM 的 HTTP

虽然 MCP 是为了协调代理而出现的，但**[Invoke](https://invoke.network)**则源于一个更简单、更尖锐的痛点：LLM 与真实世界之间的鸿沟。

语言模型可以进行推理、写作和规划——但没有工具，它们就被困在沙盒中。Invoke 始于一个问题：

*如果任何大型语言模型（LLM）都能像人类浏览网页一样发现和使用任何真实世界的 API 会怎样？*

现有的方法——MCP、OpenAI 函数、AgentOps——虽然功能强大，但要么臃肿，要么过于僵化，或者对愿景来说过于脆弱。工具的使用感觉就像是用胶带将 SDK 粘贴到自然语言上。模型必须预先连接到散布的工具，每个工具都有其自身的怪癖。

Invoke 以不同的方式处理：

+   一个工具，多个 API

+   一个标准、无限的接口

只需在清晰、易读的 JSON 中定义你的端点——方法、URL、参数、身份验证、示例。就是这样。现在 **任何模型**（GPT、Claude、Mistral）都可以自然、安全、重复地调用它。

### ⚙️ Invoke 的工作原理

在其核心，**Invoke** 是一个为 LLMs 构建的工具路由器。它就像 openapi.json，但更精简——专为推理而建，而非工程。

这就是它的工作方式：

1.  你可以使用以下方式编写结构化的工具定义（agents.json）：

    +   方法、URL、身份验证、参数以及一个示例。

1.  Invoke 将其解析为任何支持工具使用的模型的可调用函数。

1.  模型看到工具，决定何时使用它，并填写参数。

1.  Invoke 处理其余部分——身份验证、格式化、执行——并返回结果。

没有自定义包装器。没有链。没有脚手架。只有一个干净的接口。如果出了问题？我们不会预先编程重试。我们让模型来决定。结果证明：它在这一点上做得相当不错。

这里有一个真实示例：

```py
{
  "agent": "openweathermap",
  "label": "🌤 OpenWeatherMap API",
  "base_url": "https://api.openweathermap.org",
  "auth": {
    "type": "query",
    "format": "appid",
    "code": "i"
  },
  "endpoints": [
    {
      "name": "current_weather",
      "label": "☀️ Current Weather Data",
      "description": "Retrieve current weather data for a specific city.",
      "method": "GET",
      "path": "/data/2.5/weather",
      "query_params": {
        "q": "City name to retrieve weather for (string, required)."
      },
      "examples": [
        {
          "url": "https://api.openweathermap.org/data/2.5/weather?q=London"
        }
      ]
    }
  ]
}
```

从模型的角度来看，这是对“Invoke”工具的一次使用，而不是为每个添加的端点添加一个新的。不是自定义插件。只有一个可发现的接口。这意味着模型可以动态地发现 API，就像人类浏览网页一样。用运输集装箱的比喻来说，如果 MCP 允许在选定港口之间进行高度协调的工作流程和紧密协调的基础设施，那么 Invoke 使你能够随时随地发送任何包裹。

现在，我们可以使用：

```py
# 1\. Install dependencies:
# pip install langchain-openai invoke-agent

from langchain_openai import ChatOpenAI
from invoke_agent.agent import InvokeAgent

# 2\. Initialize your LLM and Invoke agent
llm = ChatOpenAI(model="gpt-4.1")
invoke = InvokeAgent(llm, agents=["path-or-url/agents.json"])

# 3\. Chat loop—any natural-language query that matches your agents.json
user_input = input("📝 You: ").strip()
response = invoke.chat(user_input)

print("🤖 Agent:", response)
```

在一分钟内，你的模型正在获取实时数据——没有包装器，没有样板代码。

你会说：“查看伦敦的天气。”

代理将前往 openweathermap.org/agents.json，读取文件，然后……就这样做了。

正如 `robots.txt` 允许爬虫安全地导航网络一样，agents.json 允许 LLM 安全地对其采取行动。Invoke 将网络转变为 LLM 可读的 API 生态系统。就像 HTML 允许人类动态发现网站和服务一样，Invoke 允许 LLM 在推理时发现 API。

想要看看它的实际应用？查看 Invoke 仓库的 [示例笔记本](https://github.com/mercury0100/invoke/tree/master/notebooks)，了解如何 [定义](https://invoke.network/create-agents-txt) `agents.json`，配置身份验证，并在一分钟内从任何 LLM 调用 API。（一旦采用达到临界质量，完整的“网络”式发现将纳入路线图。）

* * *

### 何时使用 Invoke：优势和权衡

Invoke 在现实世界中最为闪耀。

其核心前提——模型可以从单个模式中安全且准确地调用任何 API——解锁了惊人的使用场景：日历助手、电子邮件分类、天气机器人、自动化界面、客户支持代理、企业副驾驶，甚至是全栈 LLM 驱动的工作流程。并且它可以直接与 OpenAI、Claude、LangChain 等一起使用。

**优势：**

+   **简单性**。定义一次工具，到处使用。你不需要一打 Python 包装器或代理配置。

+   **模型无关性**。Invoke 与任何支持结构化工具使用的模型兼容——包括开源 LLM。

+   **开放和可扩展**。从本地配置、托管注册表或未来的公共端点（example.com/agents.json）提供工具。

+   **可组合的**。模型可以推理工具元数据，检查认证要求，甚至决定何时探索新功能。

+   **面向开发者**。与需要复杂编排的代理框架不同，Invoke 可以完美地融入现有的堆栈——前端、后端、工作流程、RAG 管道等。

+   **上下文高效**。API 配置在执行链中定义，不使用宝贵的上下文。

+   **运行时发现**。Invoke 连接在编译时不是硬编码的，在设置时也没有限制。

但像任何系统一样，Invoke 也有权衡。

**局限性：**

+   **没有中心记忆或状态**。它不管理长期计划、上下文窗口或递归子任务。这留给你——或者留给其他堆叠在顶部的框架。

+   **没有重试、超时或内置的多步骤工作流程**。Invoke 信任模型处理部分失败。在实践中，GPT-4 和 Claude 做得非常出色——但这仍然是一个哲学选择。

+   **无状态性**。工具按调用进行评估。虽然这保持了事物的简洁性和原子性，但它可能不适合需要额外支撑的复杂、多步骤代理。

* * *

### MCP 与 Invoke：通往代理网络的两种道路

MCP 和 Invoke 都旨在将 LLM 带入现实世界——但它们从相反的方向接近这个问题。

| **功能** | **模型上下文协议（MCP）** | **Invoke** |
| --- | --- | --- |
| **核心目标** | 通过消息传递进行代理到代理的协调 | 通过结构化工具使用进行 LLM 到 API 的集成 |
| **设计起源** | 可与 WebSocket 和 JSON-RPC 等协议相媲美 | 受 REST/HTTP 和 OpenAPI 的启发 |
| **主要用例** | 组合多代理工作流程和消息管道 | 直接将 LLM 连接到现实世界的 API |
| **沟通风格** | 代理之间交换的状态会话和消息 | 无状态的、基于模式的工具调用 |
| **工具发现** | 代理必须预先配置能力 | 工具模式可以在运行时发现（agents.json） |
| **错误处理** | 委派给代理框架或编排层 | 由模型处理，可选地由上下文指导 |
| **依赖关系** | 需要兼容 MCP 的基础设施和代理 | 只需要模型+JSON 工具定义 |
| **可组合性** | AutoGPT 风格的生态系统，自定义代理图 | LangChain，OpenAI 工具的使用，自定义脚本 |
| **优势** | 精细控制，可扩展的路由，代理记忆 | 简单性，开发者的人体工程学，现实世界的兼容性 |
| **局限性** | 实现起来更复杂，需要全栈，上下文膨胀，工具过多 | 设计上无状态，没有代理记忆或递归 |

* * *

## 结论：代理式网络的形态

我们正在见证互联网新层的出现——它不是由人类的点击或软件调用定义的，而是由自主代理进行推理、计划和行动定义的。

如果早期的网络建立在可读页面（HTML）和程序端点（REST）之上，那么代理式网络则需要一个新的基础：标准和框架，使得模型能够像人类曾经使用超链接一样流畅地与世界互动。

两种方法——**模型上下文协议（Model Context Protocol）**和**Invoke**——提供了关于这种交互应该如何工作的不同愿景：

+   **MCP** 在需要多个代理之间的协调、会话状态或递归推理时非常理想——它是代理式网络中的 WebSocket。

+   **Invoke** 在需要使用轻量级、一次性工具与真实世界的 API 交互时非常理想——它是代理式网络中的 HTTP。

没有一个是**唯一的**解决方案。就像早期的互联网需要 TCP 和 HTTP 一样，代理层也将是多元的。但历史表明：**获胜的工具是那些最容易采用的工具**。

Invoke 已经证明对那些只想将大型语言模型（LLM）连接到他们已使用的服务的开发者非常有用。MCP 正在为更复杂的代理系统打下基础。它们共同勾勒出了未来发展的轮廓。

代理式网络不会在一天之内建成，也不会由一家公司建成。但有一点是明确的：网络的未来不再仅仅是人类可读或机器可读——它是模型可读的。

**关于作者**

*我是 Commonwealth Bank AI Labs 的首席研究员，也是 Invoke Network（一个开源的代理式网络框架）的贡献者。欢迎反馈和分支：[`github.com/mercury0100/invoke`](https://github.com/mercury0100/invoke)*

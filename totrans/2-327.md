# 在 Ruby 中构建 AI 应用

> 原文：[`towardsdatascience.com/building-ai-applications-in-ruby/`](https://towardsdatascience.com/building-ai-applications-in-ruby/)

*📗 这是关于使用生成式 AI 集成创建 Web 应用的系列文章的第二部分。<mdspan datatext="el1747799691394" class="mdspan-comment">第一部分重点介绍了 AI 框架以及为什么应用层是框架中最佳的位置。您可以在这里*[*查看*](https://towardsdatascience.com/layers-ai-stack/)*.**

**目录**

+   简介

+   我以为水疗中心应该是放松的？

+   微服务适用于大型企业

+   Ruby 和 Python：同一枚硬币的两面

+   基于 AI 的最新宝石

+   总结

## 简介

当讨论 AI 时，很少听到 Ruby 语言被提及。

当然，Python 是这个领域的王者，而且有很好的理由。社区已经围绕这个语言聚集起来。如今，大多数模型训练都是在 PyTorch 或 TensorFlow 中完成的。Scikit-learn 和 Keras 也非常受欢迎。LangChain 和 LlamaIndex 等 RAG 框架主要针对 Python。

然而，当涉及到构建具有 AI 集成的 Web 应用时，我相信 Ruby 是更好的语言。

作为一家致力于将生成式 AI 集成到 MVP 中的应用程序开发机构的联合创始人，我经常听到潜在客户抱怨两件事：

+   应用程序构建时间过长

+   开发者报价建造定制网络应用的价格离谱

这些抱怨有一个共同的原因：**复杂性**。现代 Web 应用比过去有更多的复杂性。但这是为什么？复杂性带来的好处是否值得其成本？ 

## 我以为水疗中心应该是放松的？

拼图中的一块重要部分是单页应用（SPAs）的近期兴起。目前用于构建现代 SPAs 最受欢迎的框架是**MERN**（MongoDB, Express.js, React.js, Node.js）。这个框架之所以受欢迎，有几个原因：

+   这是一个仅使用 JavaScript 的框架，涵盖前端和后端。只需要用一种语言编写代码是非常好的！

+   SPAs 可以提供动态设计和“流畅”的用户体验。这里的“流畅”意味着当某些数据发生变化时，只有网站的一部分需要更新，而不是整个页面都需要重新加载。当然，如果你没有现代智能手机，SPAs 可能不会感觉那么流畅，因为它们通常相当重。所有这些 JavaScript 开始拖慢性能。

+   有一个庞大的库和开发者生态系统，他们在这个框架上有经验。这是一个相当循环的逻辑：这个框架之所以受欢迎是因为生态系统，还是因为生态系统才有了它的流行？无论如何，这个观点是成立的。

    React 是由 Meta 创建的。

+   已经投入了大量资金和努力到这个库中，帮助其完善和推广产品。

不幸的是，在 MERN 栈中工作有一些缺点，其中最重要的是其**复杂性**。

传统的 Web 开发使用的是模型-视图-控制器（MVC）范式。在 MVC 中，管理用户会话的所有逻辑都在后端，即服务器上处理。例如，获取用户数据是通过后端的功能调用和 SQL 语句完成的。然后后端向浏览器提供完全构建好的 HTML 和 CSS，浏览器只需显示它。因此得名“服务器”。

在 SPA 中，这种逻辑由用户的浏览器在前端处理。SPA 必须在浏览器中处理 UI 状态、应用状态，有时甚至还要处理服务器状态。API 调用必须发送到后端以获取用户数据。后端仍然有很多逻辑，主要是通过 API 公开数据和功能。

为了说明这种差异，让我用一个商业厨房的类比。顾客将是前端，厨房将是后端。

![mvs vs spa](img/45ea0d1b941e41b58d426255055f78e6.png)

MVC 与 SPA 的比较。图像由 ChatGPT 生成。

传统的 MVC 应用就像在一家全服务餐厅用餐。是的，后端有很多复杂性（如果相信《熊》的话，还会有很多喊叫）。但前端体验简单且令人满意：顾客只需拿起一把叉子，吃他们的食物即可。

SPA 就像在自助餐厅用餐。厨房里仍然有很多复杂性。但现在顾客还必须决定要拿什么食物，如何搭配它们，如何将它们摆放在盘子上，完成用餐后把盘子放在哪里等。

安德烈·卡帕西最近发了一条推文，讨论了他试图在 2025 年构建 Web 应用的挫败感。对于新手来说，这可能会让人感到不知所措。

> 2025 年构建 Web 应用的现实情况有点像组装宜家家具。没有“全栈”产品附带电池，你必须拼凑和配置许多单个服务：
> 
> – 前端 / 后端（例如 React, Next.js, APIs）
> 
> – 域名托管…
> 
> — 安德烈·卡帕西 (@karpathy) [2025 年 3 月 27 日](https://twitter.com/karpathy/status/1905051558783418370?ref_src=twsrc%5Etfw)

为了快速构建具有 AI 集成的 MVP，我们的机构决定放弃 SPA，转而采用传统的 MVC 方法。特别是，我们发现 Ruby on Rails（通常简称为 Rails）是快速开发和部署具有 AI 集成的高质量应用的框架。Ruby on Rails 由 David Heinemeier Hansson 于 2004 年开发，长期以来一直被誉为优秀的 Web 框架，但我认为它最近在将 AI 集成到应用中的能力上取得了飞跃，正如我们将看到的。

Django 是最受欢迎的 Python 网络框架，同时也拥有更传统的开发模式。不幸的是，在我们的测试中我们发现 Django 的功能并不像 Rails 那样全面或“内置电池”。一个简单的例子是，**Django 没有内置的后台作业系统**。我们的大部分应用程序都集成了后台作业，因此没有包含这一点令人失望。我们还更喜欢 Rails 强调的简洁性，Rails 8 鼓励开发者**轻松自托管应用程序**（[`kamal-deploy.org/`](https://kamal-deploy.org/)），而不是通过像 Heroku 这样的提供商。他们最近还发布了一系列旨在替代外部[服务如 Redis](https://rubyonrails.org/2024/11/7/rails-8-no-paas-required)的工具。

“但是，用户体验如何保持流畅呢？”你可能会问。事实是，现代 Rails 包括几种方法来创建类似 SPA 的体验，而不需要所有的重型 JavaScript。主要工具是 [Hotwire](https://hotwired.dev/)，它捆绑了像 **Turbo** 和 **Stimulus** 这样的工具。Turbo 允许你动态地更改网页上的 HTML 片段，而无需编写自定义 JavaScript。当你确实需要包含自定义 JavaScript 时，Stimulus 是一个最小的 JavaScript 框架，允许你做到这一点。即使你想使用 React，你也可以通过 [react-rails 珠宝](https://github.com/reactjs/react-rails) 来做到这一点。所以你可以既享受蛋糕，又吃掉它！

然而，单页面应用（SPAs）并不是复杂性增加的唯一原因。另一个原因与**微服务架构**的出现有关。

## 微服务适用于大型企业

再次，我们发现自己在比较过去的简单与今天的复杂性。

在过去，软件主要是以**单体**的形式开发的。单体应用程序意味着你的应用程序的所有不同部分——例如用户界面、业务逻辑和数据处理——都是作为一个单一单元开发、测试和部署的。代码通常都存放在单个仓库中。

与单体应用一起工作既简单又令人满意。为测试目的运行开发环境很容易。你正在使用一个包含所有表的单一数据库模式，这使得查询和连接变得简单直接。部署也很简单，因为你只需要查看和修改一个容器。

然而，一旦你的公司规模扩大到谷歌或亚马逊那样的大小，真正的难题就开始出现了。当数百或数千名开发者同时向单个代码库贡献时，协调更改和管理合并冲突变得越来越困难。部署也变得更加复杂和风险，因为即使是微小的更改也可能使整个应用程序崩溃！

为了管理这些问题，大型公司开始围绕**微服务架构**聚集在一起。这是一种编程风格，你将你的代码库设计成一系列小型、自主的服务。每个服务拥有自己的代码库、数据存储和部署管道。作为一个简单的例子，你不必将所有关于 OpenAI 客户端的逻辑都塞进你的主应用中，你可以将其逻辑移动到自己的服务中。要调用该服务，你通常会进行 REST 调用，而不是函数调用。这增加了复杂性，但解决了合并冲突和部署问题，因为组织中的每个团队都可以在自己的代码岛上工作。

使用微服务的另一个好处是它们允许使用**多语言技术栈**。这意味着每个团队都可以使用他们喜欢的任何语言来编写他们的服务。如果一个团队喜欢 JavaScript，而另一个团队喜欢 Python，这没问题。当我们最初开始我们的代理机构时，这种多语言栈的想法推动我们使用微服务架构。不是因为我们的团队很大，而是因为我们每个人都想为每个功能使用“最好的”语言。这意味着：

+   使用**Ruby on Rails**进行 Web 开发。在这个领域已经经过几十年的实战检验。

+   使用**Python**进行 AI 集成，可能部署类似于 FastAPI 的东西。我被告知，严肃的 AI 工作需要 Python。

两种不同的语言，各自专注于其专业领域。可能会出什么问题？

不幸的是，我们发现开发过程令人沮丧。仅仅设置我们的开发环境就花费了大量的时间。不得不处理 Docker compose 文件和管理服务间的通信让我们渴望回到单体应用的美丽和简洁。不得不进行 REST 调用并在 FastAPI 中设置适当的路由，而不是简单地做一个函数调用，这让人感到很糟糕。

“我们当然不能只用纯 Ruby 来开发 AI 应用，”我想。然后我试了一下。

我很高兴我做到了。

我发现使用 Ruby 开发具有 AI 集成的 MVP 的过程非常令人满意。我们能够加速开发，而之前我们只是在慢跑。我喜欢 Ruby 社区对美观、简洁和开发者幸福感的重视。我发现 Ruby 的 AI 生态系统出人意料地成熟，并且每天都在变得更好。

如果你是一名 Python 程序员，像我一样被学习新语言吓到，让我通过讨论 Ruby 和 Python 语言之间的相似性来安慰你。

## Ruby 和 Python：同一枚硬币的两面

我认为 Python 和 Ruby 就像堂兄妹。这两种语言都包含了：

+   **高级解释**：这意味着他们抽象掉了许多低级编程细节的复杂性，例如内存管理。

+   **动态类型**：这两种语言都不需要你指定变量是`int`、`float`、`string`等类型。类型检查是在运行时进行的。

+   **面向对象编程**：两种语言都是面向对象的。两者都支持类、继承、多态等。Ruby 更“纯粹”，在字面上，一切都是对象，而 Python 中有一些东西（如 `if` 和 `for` 语句）不是对象。

+   **可读且简洁的语法**：两者都认为容易学习。对初学者来说都很棒。

+   **广泛的包生态系统**：两种语言都提供了各种酷炫功能的包。在 Python 中它们被称为库，在 Ruby 中它们被称为 gems。

两种语言之间的主要区别在于它们的**哲学和设计原则**。Python 的核心哲学可以描述为：

> 应该只有一个——最好是只有一个——明显的做事方式。

理论上，这应该强调简单性、可读性和清晰性。Ruby 的哲学可以描述为：

> 总是会有不止一种方法来做某事。最大化开发者快乐。

当我从 Python 转换到 Python 时，这对我来说是个震惊。看看这个强调这种哲学差异的简单例子：

```py
# A fight over philosophy: iterating over an array
# Pythonic way
for i in range(1, 6):
    print(i)

# Ruby way, option 1
(1..5).each do |i|
  puts i
end

# Ruby way, option 2
for i in 1..5
  puts i
end

# Ruby way, option 3
5.times do |i|
  puts i + 1
end

# Ruby way, option 4
(1..5).each { |i| puts i } 
```

这两种语言之间的另一个区别是**语法风格**。Python 主要使用缩进来表示代码块，而 Ruby 使用 `do…end` 或 `{…}` 块。大多数 Ruby 块内部都包含缩进，但这完全是可选的。这些语法差异的例子可以在上面的代码中看到。

有很多其他的小差异需要学习。例如，在 Python 中字符串插值使用 f-strings：`f"Hello, {name}!"`，而在 Ruby 中使用井号：`"Hello, #{name}!"`。几个月内，我认为任何合格的 Python 程序员都可以将他们的技能转移到 Ruby 上。

## 近期基于 AI 的亮点

尽管在讨论 AI 时没有参与对话，Ruby 在 gems 领域最近也有一些进步。我将突出我们机构在构建 AI 应用程序时使用的一些最令人印象深刻的最新发布：

**RubyLLM ([链接](https://rubyllm.com/))** — 任何在发布几周内获得超过 2k 星标的 GitHub 仓库都值得提及，RubyLLM 确实值得。我使用过许多来自 LangChain 和 LlamaIndex 等库的笨拙的 LLM 提供者实现，所以使用 RubyLLM 就像一股清新的空气。作为一个简单的例子，让我们看看一个演示多轮对话的教程：

```py
require 'ruby_llm'

# Create a model and give it instructions
chat = RubyLLM.chat
chat.with_instructions "You are a friendly Ruby expert who loves to help beginners."

# Multi-turn conversation
chat.ask "Hi! What does attr_reader do in Ruby?"
# => "Ruby creates a getter method for each symbol...

# Stream responses in real time
chat.ask "Could you give me a short example?" do |chunk|
  print chunk.content
end
# => "Sure!  
#     ```ruby

# class Person

# attr...

```py

Simply amazing. Multi-turn conversations are handled automatically for you. Streaming is a breeze. Compare this to a similar implementation in LangChain:

```

from langchain_openai import ChatOpenAI

from langchain_core.schema import SystemMessage, HumanMessage, AIMessage

from langchain_core.callbacks.streaming_stdout import StreamingStdOutCallbackHandler

SYSTEM_PROMPT = "You are a friendly Ruby expert who loves to help beginners."

chat = ChatOpenAI(streaming=True, callbacks=[StreamingStdOutCallbackHandler()])

history = [SystemMessage(content=SYSTEM_PROMPT)]

def ask(user_text: str) -> None:

    """按字符流输出答案并保持上下文在内存中。”

    history.append(HumanMessage(content=user_text))

    # .stream 在消息到达时产生消息块

    for chunk in chat.stream(history):

        print(chunk.content, end="", flush=True)

    print()                                          # 答案后换行

    # 最后一段包含完整的消息内容

    history.append(AIMessage(content=chunk.content))

ask("嗨！在 Ruby 中 attr_reader 是做什么用的？")

ask("太好了 - 你能展示一个使用 attr_accessor 的简短示例吗？")

```

哎呀。重要的是要注意，这是一个 [grug](https://grugbrain.dev/) 实现。想知道 LangChain 真正期望你如何管理内存吗？查看 [out](https://python.langchain.com/api_reference/core/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html) [这些](https://langchain-ai.github.io/langgraph/concepts/persistence/) [链接](https://langchain-ai.github.io/langgraph/concepts/memory/)，但先拿个桶；你可能要生病了。

**Neighbors ([link](https://github.com/ankane/neighbor))** — 这是一个用于在 Rails 应用中进行最近邻搜索的优秀库。在 RAG 设置中非常有用。它集成了 Postgres、SQLite、MySQL、MariaDB 等。它是由 [Andrew Kane](https://github.com/ankane) 编写的，同一个人还编写了 pgvector 扩展，该扩展允许 Postgres 作为向量数据库运行。

**Async ([link](https://socketry.github.io/async/))** — 这个宝石在 2024 年 12 月首次正式发布，并在 Ruby 社区引起了轰动。Async 是一个基于纤程的 Ruby 框架，它可以在执行非阻塞 I/O 任务的同时让你编写简单、顺序化的代码。纤程就像迷你线程，每个都有自己的迷你调用栈。虽然它不是一个专门用于 AI 的宝石，但它帮助我们创建了像网络爬虫这样的功能，这些爬虫可以在数千个页面上以极快的速度运行。我们还用它来处理从 LLMs 流出的块。

**Torch.rb ([link](https://github.com/ankane/torch.rb))** — 如果你感兴趣于训练深度学习模型，那么你肯定听说过 PyTorch。好吧，PyTorch 是基于 LibTorch 构建的，本质上在底层有很多 C/C++ 代码来快速执行 ML 操作。Andrew Kane 在 LibTorch 上创建了一个 Ruby 适配器，即 Torch.rb，本质上是一个 PyTorch 的 Ruby 版本。Andrew Kane 在 Ruby AI 世界中是一位英雄，他 [编写了数十个](https://ankane.org/new-ml-gems) ML 宝石用于 Ruby。

## 摘要

**简而言之：快速且低成本地将 AI 集成到 Web 应用程序中需要单体架构。** 单体架构要求应用是单语的，这对于你最终目标是快速交付高质量应用来说是必要的。你的主要选择是 Python 或 Ruby。如果你选择 Python，你可能会使用 Django 作为你的 Web 框架。如果你选择 Ruby，你将使用 Ruby on Rails。在我们的代理机构中，我们发现 Django 的功能不足令人失望。Rails 以其功能集和对简单性的重视给我们留下了深刻印象。我们在 AI 方面几乎没有发现任何问题。

当然，有些时候你可能不想使用 Ruby。如果你正在进行人工智能研究或从头开始训练机器学习模型，那么你可能会想坚持使用 Python。研究几乎从不涉及构建网络应用程序。最多你可能会在笔记本中构建一个简单的界面或仪表板，但不会是用于生产的。你可能还希望获取最新的 PyTorch 更新以确保你的训练运行得更快。你甚至可能会深入到低级别的 C/C++ 编程，以从你的硬件中获得尽可能多的性能。也许你还会尝试你的手艺去尝试[Mojo](https://www.modular.com/mojo)。

但如果你的目标是将最新的 LLM（无论是开源还是闭源）集成到网络应用程序中，那么我们认为 Ruby 是一个远优于其他选项的选择。自己试试看吧！

在本系列的第三部分，我将进行一个有趣的实验：**我们如何能够使具有人工智能集成的网络应用程序变得尽可能简单？**请保持关注。

🔥* 如果你想定制一个具有生成式人工智能集成的网络应用程序，请访问[*losangelesaiapps.com*](https://losangelesaiapps.com/)

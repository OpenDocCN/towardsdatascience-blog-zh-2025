# 生成式 AI 的文化抵制

> 原文：[`towardsdatascience.com/the-cultural-backlash-against-generative-ai-30372d3b9080/`](https://towardsdatascience.com/the-cultural-backlash-against-generative-ai-30372d3b9080/)

![由 Joshua Hoehne 在 Unsplash 上的照片](img/8160559e2e88f6e090a2ddac950c8180.png)

由[Joshua Hoehne](https://unsplash.com/@joshua_hoehne?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)上的照片

最近，中国一家公司（也名为 DeepSeek）开发的深度学习模型 DeepSeek-R1 的公布，对我们这些花费时间观察和分析 AI 周围文化和社会现象的人来说，是一个非常有趣的事件。[证据表明，R1 的训练成本仅为训练 ChatGPT（实际上是其任何最近模型）的一小部分](https://www.bbc.com/news/articles/c5yv5976z9po)，并且有几个原因可能是真实的。但这并不是我想在这里讨论的真正问题——许多[深思熟虑的](https://www.wheresyoured.at/deep-impact/) [作家](https://www.wired.com/story/deepseek-china-model-ai/) [已经](https://arstechnica.com/ai/2025/01/how-does-deepseek-r1-really-fare-against-openais-best-reasoning-models/) [评论](https://www.theguardian.com/commentisfree/2025/jan/28/deepseek-r1-ai-world-chinese-chatbot-tech-world-western)了 DeepSeek-R1 是什么，以及训练过程中真正发生了什么。

我目前更感兴趣的是这条新闻如何改变了 AI 领域的一些动力。当 DeepSeek-R1 的新闻发布时，[英伟达和其他相关股票大幅下跌](https://www.cnbc.com/2025/01/27/nvidia-sheds-almost-600-billion-in-market-cap-biggest-drop-ever.html)，这似乎主要是因为它不需要最新的 GPU 进行训练，并且通过更有效的训练，所需的电力比 OpenAI 模型要少。我已经在思考大型生成式 AI 面临的文化抵制，而类似的事情为人们批判生成式 AI 公司的实践和承诺开辟了更多的空间。

在批判生成式 AI 作为商业或技术方面，我们目前处于什么位置？这种批判从何而来，为什么可能发生？

## 思想流派

我认为最有趣的两个经常重叠的批评角度是首先，社会或社区利益视角，其次，实用视角。从社会利益的角度来看，对生成式 AI 作为商业和行业的批判多种多样，[我在这里的写作中已经谈了很多关于它们的内容](https://medium.com/towards-data-science/environmental-implications-of-the-ai-boom-279300a24184)。将生成式 AI 变成一种无处不在的东西，代价是巨大的，从环境到经济，以及更多。

实际上，可能最简单的方法就是将其归结为“这项技术并没有按照我们承诺的方式工作”。生成式 AI 对我们撒谎，或者说“产生幻觉”，并且在许多我们需要技术帮助的任务上表现不佳。我们被引导相信可以信任这项技术，但它未能达到期望，同时被用于合成 CSAM 和深度伪造等令人痛苦和犯罪的事情，以破坏民主。

所以当我们一起看这些时，你可以发展出一个相当有力的论点：这项技术并没有达到过度炒作的期望，为了这种令人失望的表现，我们放弃了电力、水、气候、金钱、文化和工作。这并不是许多人眼中值得的交易，至少可以说是不值得的！

我确实喜欢在这个领域带来一点细微差别，因为我认为当我们接受生成式 AI 能做什么以及它可能造成的伤害的限制，并且不玩过度炒作的游戏时，我们可以找到一个可接受的折中点。我认为，除非结果真的非常、非常值得，否则我们不应该为这些模型的训练和推理支付高昂的价格。为医学研究开发新分子？也许，是的。帮助孩子们（拙劣地）作弊作业？不，谢谢。我甚至不确定帮助我在工作中更高效地编写代码是否值得，除非我正在做真正有价值的事情。我们需要对创造和使用这项技术的真实成本保持诚实和现实。

## 我们是如何走到这一步的

那么，话虽如此，我想深入探讨一下这种情况是如何形成的。我在 2023 年 9 月就写过，机器学习有一个公共认知问题，在生成式 AI 的情况下，我认为这一点已经被事件证明。具体来说，如果人们对 LLMs 擅长什么以及不擅长什么没有现实期望和理解，他们就会反弹，随之而来的是反冲。

> *"我的论点大致如下：*
> 
> *1. 人们并不天生准备好理解和与机器学习互动。*
> 
> *2. 如果不了解这些工具，一些人可能会避免或怀疑它们。*
> 
> *3. 更糟糕的是，一些人可能由于错误信息而误用这些工具，导致不良后果。*
> 
> *4. 经历了误用的负面后果后，人们可能会不愿意采用未来可能增强他们生活和社区生活的机器学习工具。“*
> 
> [《机器学习的公共认知问题，2023 年 9 月》](https://medium.com/towards-data-science/machine-learnings-public-perception-problem-48daf587e7a8)

那么，发生了什么事？好吧，生成式 AI 行业一头扎进了这个问题，我们现在看到了后果。

## 生成式 AI 应用未能满足人们的需求

问题的一部分在于[生成式 AI 实际上并不能有效地做到炒作中所声称的每一件事](https://hbr.org/2023/06/the-ai-hype-cycle-is-distracting-companies)。一个 LLM 不能可靠地用来回答问题，因为它不是一个“事实机器”。它是一个“句子中可能的下一个词机器”。但我们看到的各种承诺都忽略了这些限制，科技公司正在将生成式 AI 功能强加到你能想到的每一种软件中。人们讨厌微软的 Clippy，因为它不好用，他们不希望有人把这种东西强加给他们——有人可能会说[他们正在用改进版做同样的事情，我们可以看到有些人仍然可以理解地对此表示反感](https://www.theverge.com/2025/1/16/24345051/microsoft-365-personal-family-copilot-office-ai-price-rises)。

当有人今天去 LLM 询问他们当地杂货店现在食谱中食材的价格时，模型绝对没有机会正确、可靠地回答这个问题。这不在它的能力范围内，因为关于这些价格的真实数据对模型是不可用的。模型可能会意外地猜测在 Publix 超市一袋胡萝卜的价格是 1.99 美元，但这只是个意外。在未来，通过以代理形式将模型链在一起，我们有可能开发出能够正确完成这类任务的窄模型，但到目前为止，这绝对是胡说八道。

但如今人们却在向 LLMs 提出这些问题！当他们到达商店时，他们对被这种他们认为能提供神奇答案的技术欺骗感到非常失望。如果你是 OpenAI 或 Anthropic，你可能会耸耸肩，因为如果那个人在向你支付月费，那么，你已经得到了现金。如果他们没有，那么，你得到了用户数量增加一个，这就是增长。

然而，这实际上是一个重大的商业问题。当你的产品以这种明显、可预测（不可避免！）的方式失败时，你开始烧毁用户与你的产品之间的桥梁。它可能不会一次性烧毁，但它正在逐渐破坏用户与你的产品之间的关系，在你失去用户之前，你只有这么多的机会。在生成式 AI 的情况下，在我看来，你几乎没有任何机会。此外，一种模式的失败可能会使人们对所有形式的技术都产生怀疑。几年后，当你的 LLM 后端连接到实时价格 API 并实际上可以正确返回杂货店的价格时，那个用户会信任或相信你吗？我不这么认为。那个用户甚至可能不会让你在它未能完成其他任务后帮助修改与同事的电子邮件。

从我所见到的来看，科技公司认为他们可以仅仅通过消耗人们的耐心，迫使他们接受生成式 AI 现在是他们所有软件中不可避免的一部分，无论它是否有效。也许他们可以，但我认为这是一种自毁长城的策略。用户可能会勉强接受现状，但因此他们不会对技术或你的品牌持积极态度。勉强的接受并不是你希望你的品牌在用户中激发的能量！

## 硅谷与此有何关系

你可能会想，嗯，这已经很明确了——让我们减少软件中的生成式 AI 功能，只将其应用于可以给用户带来震撼且效果良好的任务。他们会有良好的体验，然后随着技术的进步，我们将在合理的地方添加更多功能。这将会是一种相对合理的思考（尽管，如我之前提到的，对我们世界和社区的外部成本将会非常高）。

然而，我认为大型生成式 AI 玩家真的做不到这一点，原因如下。科技领导者们已经投入了巨额资金来创造和尝试改进这项技术——从投资开发它的公司[《路透社》：OpenAI 融资轮估值达 3400 亿美元，WSJ 报道 2025 年 1 月 30 日](https://www.reuters.com/technology/artificial-intelligence/openai-talks-investment-round-valuing-it-up-340-billion-wsj-reports-2025-01-30/)，到[建设发电厂和数据中心](https://www.cnn.com/2025/01/21/tech/openai-oracle-softbank-trump-ai-investment/index.html)，再到游说以规避版权法，已经有数百亿美元的资金投入到这个领域，而且还有更多的资金即将到来。

在科技行业，盈利预期与其他行业相比有很大不同——[一家风险投资支持的软件初创公司必须收回 10-100 倍的投资（取决于阶段）才能被视为真正的成功](https://kruzeconsulting.com/blog/what-vcs-return-expectations/)。因此，科技行业的投资者会明确或隐性地推动公司采取更大的赌注和更大的风险，以使更高的回报变得可行。[这开始发展成我们所说的“泡沫”——估值与实际经济可能性脱节，越来越高，没有任何希望成为现实。](https://www.washingtonpost.com/technology/2024/07/24/ai-bubble-big-tech-stocks-goldman-sachs/) [正如华盛顿邮报的 Gerrit De Vynck 指出](https://www.washingtonpost.com/technology/2024/07/24/ai-bubble-big-tech-stocks-goldman-sachs/)，“……华尔街分析师预计，到 2026 年，大型科技公司每年将在开发 AI 模型上投入约 600 亿美元，但到那时仅从 AI 中获得约 200 亿美元的年收入……风险投资家还向数千家 AI 初创公司投入了数十亿美元。AI 的繁荣有助于推动风险投资公司在美国初创公司上的投资达到 556 亿美元，这是两年内单季度的最高金额，据风险投资数据公司 PitchBook 的数据显示。”

[因此，考虑到投入的数十亿美元，有严重的论点可以提出，迄今为止投入开发生成式 AI 的金额是无法与回报相匹配的。](https://www.wheresyoured.at/oai-business/) 在这里通过这项技术赚取的金钱并不多，当然与已经投入的金额相比更是微不足道。但是，公司肯定会尝试。我认为这就是为什么我们看到生成式 AI 被应用到各种可能并不特别有帮助、有效或受欢迎的使用场景中的部分原因。从某种意义上说，“我们在这项技术上投入了这么多钱，所以我们必须找到一种方法来销售它”似乎是一种框架。同时，也要记住，投资仍在继续，试图让这项技术工作得更好，但任何 LLM 的进步在当今都证明是非常缓慢和渐进的。

## 接下来该怎么做？

生成式 AI 工具并没有证明对人们的生活是必需的，因此经济计算并没有促使产品上市并说服人们购买它。因此，我们看到公司正在转向生成式 AI 的“功能”模式，正如我在 2024 年 8 月发表的文章中[理论化可能会发生的情况](https://medium.com/towards-data-science/economics-of-generative-ai-75f550288097)。然而，这种方法采取了非常强硬的手段，就像微软将生成式 AI 添加到 Office365 中，并使功能和相应的价格增加都成为强制性的。我承认我直到最近才意识到公共形象问题与功能与产品模式问题之间的联系——但现在我们可以看到它们是相互交织的。提供人们具有功能问题的功能，然后对其收费，这对公司来说仍然是一个真正的问题。也许当某件事根本不适合一项任务时，它既不是产品也不是功能？如果这种情况成立，那么生成式 AI 的投资者将面临真正的问题，因此公司承诺提供生成式 AI 功能，无论它们是否有效。

我将非常关注这个领域的发展。我不期望生成式 AI 功能有大的飞跃，尽管根据 DeepSeek 的结果，我们可能会看到一些效率上的飞跃，至少是在训练方面。如果公司倾听用户的投诉并转型，将生成式 AI 应用于它真正有用的应用，他们可能更有机会应对反击，无论好坏。然而，对我来说，这似乎与公司面临的绝望的利润动机高度不兼容。在这个过程中，我们将浪费大量资源在生成式 AI 的愚蠢用途上，而不是将我们的努力集中在真正值得交易的技术应用上。

* * *

在[www.stephaniekirmer.com](http://www.stephaniekirmer.com)阅读我的更多作品。

* * *

## 进一步阅读

[`www.bbc.com/news/articles/c5yv5976z9po`](https://www.bbc.com/news/articles/c5yv5976z9po)

[`www.cnbc.com/2025/01/27/nvidia-sheds-almost-600-billion-in-market-cap-biggest-drop-ever.html`](https://www.cnbc.com/2025/01/27/nvidia-sheds-almost-600-billion-in-market-cap-biggest-drop-ever.html)

[`medium.com/towards-data-science/environmental-implications-of-the-ai-boom-279300a24184`](https://medium.com/towards-data-science/environmental-implications-of-the-ai-boom-279300a24184)

[`hbr.org/2023/06/the-ai-hype-cycle-is-distracting-companies`](https://hbr.org/2023/06/the-ai-hype-cycle-is-distracting-companies)

[`www.theverge.com/2025/1/16/24345051/microsoft-365-personal-family-copilot-office-ai-price-rises`](https://www.theverge.com/2025/1/16/24345051/microsoft-365-personal-family-copilot-office-ai-price-rises)

[2025 年 1 月 30 日，路透社报道：OpenAI 正在洽谈投资轮融资，估值高达 3400 亿美元，据 WSJ 报道](https://www.reuters.com/technology/artificial-intelligence/openai-talks-investment-round-valuing-it-up-340-billion-wsj-reports-2025-01-30/)

[2025 年 1 月 21 日，CNN 报道：OpenAI、Oracle、Softbank 和特朗普的 AI 投资](https://www.cnn.com/2025/01/21/tech/openai-oracle-softbank-trump-ai-investment/index.html)

[2024 年 7 月 24 日，华盛顿邮报报道：AI 泡沫、大型科技公司股票和 Goldman Sachs](https://www.washingtonpost.com/technology/2024/07/24/ai-bubble-big-tech-stocks-goldman-sachs/)

[`www.wheresyoured.at/oai-business/`](https://www.wheresyoured.at/oai-business/)

[生成式 AI 的经济效益](https://medium.com/towards-data-science/economics-of-generative-ai-75f550288097)

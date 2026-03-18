# 你是否对 LLMs 不公平？

> 原文：[`towardsdatascience.com/are-you-being-unfair-to-llms/`](https://towardsdatascience.com/are-you-being-unfair-to-llms/)

在当前围绕人工智能的炒作中，一些关于 LLM 智能本质的错误观念正在流传，我想就其中一些进行说明。我将提供来源——大多数是预印本——并欢迎您对此事的看法。

为什么我认为这个话题很重要？首先，我感觉我们正在创造一种新的智能，它在许多方面与我们竞争。因此，我们应该公正地评判它。其次，人工智能的话题非常深刻。它提出了关于我们的思维过程、我们的独特性和我们相对于其他生物的优越感的问题。

Millière 和 Buckner 写道[1]：

> 尤其是我们需要理解 LLMs 关于它们产生的句子以及这些句子所涉及的世界所代表的内容。这种理解不能仅仅通过闲坐推测就能达到；它需要仔细的实证调查。

## LLMs 不仅仅是预测机

深度神经网络可以形成复杂的结构，具有线性-非线性路径。神经元可以在叠加中承担多个功能[2]。此外，LLMs 构建了它们所分析上下文的内部世界模型和心智图[3]。因此，它们不仅仅是预测下一个单词的预测机。它们内部的激活会提前思考一个陈述的结尾——它们心中有一个基本的计划[4]。

然而，所有这些能力都取决于模型的大小和性质，因此它们可能有所不同，尤其是在特定环境中。这些通用能力是一个活跃的研究领域，可能比拼写检查器的算法（如果您需要在这两者之间选择一个）更类似于人类的思维过程。

## LLMs show signs of creativity

面对新任务时，LLMs 所做的不仅仅是重复记忆的内容。相反，它们可以产生自己的答案[5]。Wang 等人分析了模型输出与[Pile 数据集](https://pile.eleuther.ai/)的关系，发现更大的模型在回忆事实和创造更多新颖内容方面都取得了进步。

然而，Salvatore Raieli 最近在[TDS](https://towardsdatascience.com/can-machines-dream-on-the-creativity-of-large-language-models-d1d20cf51939/)上[报告](https://towardsdatascience.com/can-machines-dream-on-the-creativity-of-large-language-models-d1d20cf51939/)称，大型语言模型（LLMs）并不具有创造力。引用的研究主要关注 ChatGPT-3。相比之下，Guzik、Erike 和 Byrge 发现，GPT-4 在人类创造力的前百分之一[6]。Hubert 等人也同意这一结论[7]。这适用于原创性、流畅性和灵活性。生成与模型训练数据中未见过的全新想法可能又是另一回事；这可能是在某些方面，卓越的人类仍然具有优势的地方。

无论哪种方式，关于这些迹象的争论太多，不能完全否定。要了解更多关于这个主题的信息，你可以查阅[计算创造力](https://en.wikipedia.org/wiki/Computational_creativity)。

## LLMs 有一个关于情感的概念

LLMs 可以分析情感上下文，并以不同的风格和情感调子写作。这表明它们具有代表情感的内部关联和激活。确实，有这样的相关性证据：可以通过探查其神经网络中某些情感的激活，甚至可以通过**引导向量**人工诱导它们[8]。（识别这些引导向量的方法之一是在模型处理具有相反属性（例如，悲伤与快乐）的陈述时确定对比激活。）

因此，情感属性及其与内部世界模型可能存在的关系似乎属于 LLM 架构可以表示的范围。情感表示与后续推理之间存在关联，即 LLM 理解的世界。

此外，情感表示局限于模型的某些区域，并且许多适用于人类的直观假设也可以在 LLMs 中观察到——甚至心理和认知框架也可能适用[9]。

注意，上述陈述并不暗示**现象学**，即 LLMs 有主观体验。

## 是的，LLMs 在训练后不会学习（post-training）

LLMs 是具有**静态权重**的神经网络。当我们与 LLM 聊天机器人聊天时，我们正在与一个不会改变的模型进行交互，并且只在当前聊天的**上下文**中学习。这意味着它可以从网络或数据库中提取额外数据，处理我们的输入等。但它的**本质**、内置知识、技能和偏见保持不变。

除了为静态 LLMs 提供额外上下文数据的长期记忆系统之外，未来的方法可以通过调整核心 LLM 的权重来实现自我修改。这可以通过持续使用新数据进行预训练或通过持续微调和叠加额外权重来实现[10]。

正在探索许多替代神经网络架构和适应方法，以有效地实现连续学习系统[11]。这些系统是存在的；只是目前还不可靠和经济。

## 未来发展

我们不要忘记，我们现在看到的 AI 系统非常新。“它不擅长 X”这个说法可能会很快变得无效。此外，我们通常是在评判低价的消费产品，而不是那些运行成本过高、不受欢迎或仍然被锁在门后的顶级模型。去年半年的许多 LLM 开发都集中在为消费者创建更便宜、更容易扩展的模型上，而不仅仅是更智能、价格更高的模型。

虽然计算机在某些领域可能缺乏原创性，但它们擅长快速尝试不同的选项。现在，LLMs 可以自我判断。当我们缺乏直觉答案而进行创造性思考时，我们不是在做同样的事情——循环思考并挑选最佳方案？LLMs 固有的创造力（或者你想要称之为什么），加上快速迭代想法的能力，已经为科学研究带来了好处。参见我之前关于[AlphaEvolve](https://towardsdatascience.com/googles-alphaevolve-getting-started-with-evolutionary-coding-agents/)的文章以获取示例。

像幻觉、偏见和绕过 LLMs 安全防护的越狱等弱点，以及安全和可靠性问题，仍然普遍存在。尽管如此，这些系统如此强大，以至于可能有许多应用和改进。LLMs 也不必孤立使用。当与传统方法相结合时，一些缺点可能得到缓解或变得无关紧要。例如，LLMs 可以为随后用于工业自动化的传统 AI 系统生成逼真的训练数据。即使发展放缓，我相信从药物研究到教育等领域，还有数十年的好处可以探索。

## LLMs 只是算法。或者，它们是吗？

许多研究人员现在正在发现人类思维过程与 LLM 信息处理之间的相似性（例如，[12]）。长期以来，人们已经接受 CNN 可以与人类视觉皮层的层相提并论[13]，但现在我们谈论的是新皮层[14, 15]！请别误会；也存在明显的差异。尽管如此，LLMs 的[能力爆炸](https://arstechnica.com/ai/2025/07/how-a-big-shift-in-training-llms-led-to-a-capability-explosion/)是无法否认的，而且我们关于独特性的主张似乎并不成立。

现在的问题是这将引导我们走向何方，以及极限在哪里——在什么点上我们必须讨论意识？像 Geoffrey Hinton 和 Douglas Hofstadter 这样的知名思想领袖已经开始欣赏基于最近 LLM 突破的人工智能意识的可能性[16, 17]。其他人，如 Yann LeCun，则持怀疑态度[18]。

James F. O’Brien 教授去年在 TDS 上[分享了他的观点](https://towardsdatascience.com/an-illusion-of-life-5a11d2f2c737/)，关于 LLM 意识的话题，并问道：

> 我们将有一种测试意识的方法吗？如果是这样，它将如何工作，如果结果为阳性，我们应该做什么？

## 继续前进

当我们将人类特质归因于机器时，我们应该小心——拟人化现象发生得太容易了。然而，也容易忽视其他生物。我们已经看到这种情况在动物身上发生得太频繁了。

因此，无论当前 LLMs 最终是否具有创造力、拥有世界模型或具有意识，我们可能都不想贬低它们。下一代 AI 可能三者兼具[19]。

你怎么看？

## 参考文献

1.  Millière, Raphaël, and Cameron Buckner, [《语言模型哲学导论——第一部分：与经典辩论的连续性》](https://arxiv.org/abs/2401.03910) (2024), arXiv.2401.03910

1.  Elhage, Nelson, Tristan Hume, Catherine Olsson, Nicholas Schiefer, Tom Henighan, Shauna Kravec, Zac Hatfield-Dodds, et al., [《叠加的玩具模型》](https://arxiv.org/abs/2209.10652v1) (2022), arXiv:2209.10652v1

1.  Kenneth Li, [《大型语言模型是学习世界模型还是仅仅学习表面统计？》](https://thegradient.pub/othello/) (2023), The Gradient

1.  Lindsey, et al., [《大型语言模型的生物学》](https://transformer-circuits.pub/2025/attribution-graphs/biology.html) (2025), Transformer Circuits

1.  Wang, Xinyi, Antonis Antoniades, Yanai Elazar, Alfonso Amayuelas, Alon Albalak, Kexun Zhang, and William Yang Wang, [《泛化与记忆：追踪语言模型能力回到预训练数据》](http://arxiv.org/abs/2407.14985) (2025), arXiv:2407.14985

1.  Guzik, Erik & Byrge, Christian & Gilde, Christian, [《机器的原创性：AI 通过托兰斯测试》](https://www.researchgate.net/publication/373313932_The_Originality_of_Machines_AI_Takes_the_Torrance_Test) (2023), Journal of Creativity

1.  Hubert, K.F., Awa, K.N. & Zabelina, D.L, [《人工智能生成语言模型当前状态在发散思维任务上比人类更具创造力》](https://doi.org/10.1038/s41598-024-53303-w) (2024), Sci Rep 14, 3440

1.  Turner, Alexander Matt, Lisa Thiergart, David Udell, Gavin Leech, Ulisse Mini, and Monte MacDiarmid, [《激活添加：无需优化的语言模型引导》](https://arxiv.org/abs/2308.10248v3) (2023), arXiv:2308.10248v3

1.  Tak, Ala N., Amin Banayeeanzade, Anahita Bolourani, Mina Kian, Robin Jia, and Jonathan Gratch, [《大型语言模型中情感推理的机制可解释性》](http://arxiv.org/abs/2502.05489) (2025), arXiv:2502.05489

1.  Albert, Paul, Frederic Z. Zhang, Hemanth Saratchandran, Cristian Rodriguez-Opazo, Anton van den Hengel, and Ehsan Abbasnejad, [《RandLoRA：大型模型的满秩参数高效微调》](http://arxiv.org/abs/2502.00987) (2025), arXiv:2502.00987

1.  Shi, Haizhou, Zihao Xu, Hengyi Wang, Weiyi Qin, Wenyuan Wang, Yibin Wang, Zifeng Wang, Sayna Ebrahimi, and Hao Wang, [《大型语言模型的持续学习：全面调查》](https://arxiv.org/abs/2404.16789) (2024), arXiv:2404.16789

1.  Goldstein, A., Wang, H., Niekerken, L. et al., [《统一声学到语音到语言嵌入空间捕捉日常对话中自然语言处理的神经基础》](https://doi.org/10.1038/s41562-025-02105-9) (2025)*,* Nat Hum Behav 9, 1041–1055

1.  Yamins, Daniel L. K., Ha Hong, Charles F. Cadieu, Ethan A. Solomon, Darren Seibert, and James J. DiCarlo, [性能优化的分层模型预测高级视觉皮层的神经反应](https://www.pnas.org/doi/10.1073/pnas.1403112111) *(2014), 美国国家科学院院刊* 111(23): 8619–24

1.  Granier, Arno, 和 Walter Senn, [皮质-丘脑回路中的多头自注意力](https://arxiv.org/abs/2504.06354) (2025), arXiv:2504.06354

1.  Han, Danny Dongyeop, Yunju Cho, Jiook Cha, 和 Jay-Yoon Lee, [注意差距：将大脑与语言模型对齐需要非线性和多模态方法](https://arxiv.org/abs/2502.12771) (2025), arXiv:2502.12771

1.  [`www.cbsnews.com/news/geoffrey-hinton-ai-dangers-60-minutes-transcript/`](https://www.cbsnews.com/news/geoffrey-hinton-ai-dangers-60-minutes-transcript/)

1.  [`www.lesswrong.com/posts/kAmgdEjq2eYQkB5PP/douglas-hofstadter-changes-his-mind-on-deep-learning-and-ai`](https://www.lesswrong.com/posts/kAmgdEjq2eYQkB5PP/douglas-hofstadter-changes-his-mind-on-deep-learning-and-ai)

1.  Yann LeCun, [通往自主机器智能的道路](https://openreview.net/pdf?id=BZ5a1r-kVsf) (2022), OpenReview

1.  Butlin, Patrick, Robert Long, Eric Elmoznino, Yoshua Bengio, Jonathan Birch, Axel Constant, George Deane, 等人, [人工智能中的意识：来自意识科学的见解](https://arxiv.org/abs/2308.08708v3) (2023), arXiv: 2308.08708

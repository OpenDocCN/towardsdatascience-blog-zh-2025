# 我在 2025 年人工智能伦理课程中的更新内容

> 原文：[`towardsdatascience.com/what-im-updating-in-my-ai-ethics-class-for-2025-27cd55aa9587/`](https://towardsdatascience.com/what-im-updating-in-my-ai-ethics-class-for-2025-27cd55aa9587/)

新技术发展迅速，但哪些具有长期伦理或价值观影响的问题将会出现？

我一直在为 2025 年关于人工智能价值观和伦理的课程更新而努力。这门课程是约翰霍普金斯大学专业教育项目的一部分，也是人工智能硕士学位课程的一部分。

## 我所做的变更概述：

我正在基于 2024 年的发展对三个主题进行重大更新，并对多个小主题进行更新，整合其他新闻并填补课程中的空白。

### 主题 1：LLM 可解释性。

Anthropic 在可解释性方面的工作是可解释人工智能（XAI）的一个突破。我们将讨论这种方法如何在实践中应用，以及对我们思考 AI 理解的影响。

### 主题 2：以人为本的 AI。

快速的人工智能发展使得以下问题变得更加紧迫：我们如何设计人工智能来赋能人类而不是取代人类？我在整个课程中添加了相关内容，包括两个新的设计练习。

![](img/b82a0296e5daf97d4a355d9f1a67e284.png)

### 主题 3：AI 法律与治理。

重大发展包括欧盟的 AI 法案和一系列加州立法，包括针对深度伪造、虚假信息、知识产权、医疗通信和未成年人使用“成瘾性”社交媒体等法律。我为课程开发了一些评估 AI 立法的启发式方法，例如研究定义，并解释立法只是解决 AI 治理难题的一个方面。

![加利福尼亚州州长加文·纽瑟姆签署了多项新 AI 法案之一；公共领域照片由加州州政府提供](img/a731faf73872532f81f8794c9bc50150.png)

加利福尼亚州州长加文·纽瑟姆签署了多项新 AI 法案之一；公共领域 [照片由加州州政府提供](https://www.gov.ca.gov/2024/09/17/governor-newsom-signs-bills-to-protect-digital-likeness-of-performers/)

### 其他新材料：

我将新闻报道中的材料整合到现有的关于版权、风险、隐私、安全和社交媒体/智能手机危害的主题中。

## **主题 1：生成式 AI 可解释性**

**新内容：**

Anthropic 在 2024 年对可解释性的开创性工作让我着迷。他们在这里发布了一篇博客文章 [here](https://www.anthropic.com/research/mapping-mind-language-model)，还有一篇 [论文](https://transformer-circuits.pub/2024/scaling-monosemanticity/index.html)，以及一个交互式功能浏览器。大多数技术熟练的读者应该能够从博客和论文中获得一些东西，尽管有些内容具有技术性，而且论文标题（“Scaling Monosemanticity”）令人望而生畏。

下面是一张展示发现的一个特征的截图，‘拍马屁’。我喜欢这个，因为它有心理上的微妙之处；令我惊讶的是，他们能够将这个抽象概念与简单的‘奉承’或‘赞扬’区分开来。

![来自论文《扩展单义性：从 Claude 3 Sonnet 中提取可解释特征》的图表](img/cd215caf4885a72a57bdf9de095ba678.png)

来自论文《扩展单义性：从 Claude 3 Sonnet 中提取可解释特征》的图表。

**什么是重要的：**

**可解释 AI：**对于我的伦理学课程来说，这最相关于可解释 AI（XAI），它是以人为本设计的关键组成部分。我将向课堂提出的问题是如何利用这种新能力在使用 LLMs 时促进人类理解和赋权。SAEs（稀疏自动编码器）成本太高且难以训练，不能成为 XAI 问题的完整解决方案，但它们可以为多方面的 XAI 策略增添深度。

**安全影响：**Anthropic 在安全领域的工作也值得提及。他们将‘拍马屁’特征作为其安全工作的一部分，特别相关于这个问题：一个非常强大的 AI 能否隐藏其意图，可能通过奉承用户使其自满？这个方向在最近的研究中尤其突出：[前沿模型能够进行情境策略。](https://arxiv.org/abs/2412.04984)（注意：也参见 Anthropic 关于[对齐伪装](https://www.anthropic.com/research/alignment-faking)的工作。感谢 Sean Sica 的提示。）

**AI‘理解’的证据？**可解释性是否终结了‘随机鹦鹉’？我已经确信了一段时间，LLMs 必须有一些关于复杂和相互关联概念的内部表示。他们不能像‘随机鹦鹉’那样，仅仅通过记忆多少模式就能做到他们所做的事情。Anthropic 所识别的复杂抽象的使用符合我对‘理解’的定义，尽管有些人只将这个术语用于人类理解。也许我们应该为‘AI 理解’添加一个限定词。这不是我在伦理学课程中明确覆盖的主题，但它确实在相关主题的讨论中出现了。

**需要 SAE 可视化。**我仍在寻找一个很好的视觉说明，展示深度网络中的复杂特征是如何映射到一个非常薄、非常宽的具有稀疏表示特征的 SAEs 上。我现在所拥有的只是我为课堂使用创建的 Powerpoint 近似图，如下所示。感谢 Brendan Boycroft 的 LLM 可视化器，它帮助我更好地理解了 LLMs 的机制。[`bbycroft.net/llm`](https://bbycroft.net/llm)

![作者对 SAE 映射的描述](img/a88300992d0df013a26b701895d09405.png)

作者对 SAE 映射的描述

## 主题 2：以人为本的 AI（HCAI）

**有什么新内容？**

到 2024 年，人工智能将影响每一个人类事业，并且似乎比蒸汽动力或计算机等以前的技术变化得更快。变化的速度几乎比变化的性质更重要，因为人类文化、价值观和伦理通常不会快速改变。现在设定的不适应模式和先例将越来越难以在以后改变。

**什么是重要的？**

以人为中心的人工智能需要不仅仅是一个学术兴趣，它需要成为一个被广泛理解和实践的价值观、实践和设计原则的集合。我喜欢的一些人和组织，包括前面提到的 Anthropic 可解释性工作，有斯坦福大学的以人为中心的人工智能 [Stanford’s Human-Centered AI](https://hai.stanford.edu/)，谷歌的[People + AI](https://pair.withgoogle.com/) 努力工作，以及 [Ben Schneiderman](https://www.cs.umd.edu/users/ben/) 早期的领导和社区组织。

对于我的工作型人工智能工程师班级，我正试图专注于实用和具体的设计原则。我们需要对抗我似乎无处不在的无效设计原则：“尽可能快地自动化一切”，以及“隐藏一切，以免用户搞砸”。我正在寻找挑战人们站起来并以赋予人类前所未有的智慧、智慧和更好能力的方式使用人工智能的案例和例子。

我为关于工作未来、HCAI 和致命自主武器等课程模块编写了虚构案例。[案例 1](https://medium.com/@nathanbos/2c4885b9560e) 是关于一个面向客户的 LLM 系统，它试图做得太多太快，并排除了专家人类。 [案例 2](https://medium.com/@nathanbos/0ad4caf4787f) 是关于一位高中教师，她发现大多数学生都在用 LLM 作弊申请露营申请论文，并希望以更好的方式使用 GenAI。

![图片](img/ce3c262707f522e6680e3625c10fa31e.png)

这些案例在 Medium 的单独页面上 [[这里](https://medium.com/@nathanbos/0ad4caf4787f)](https://medium.com/@nathanbos/2c4885b9560e) 和这里，我非常欢迎反馈！感谢 Sara Bos 和 Andrew Taylor 已经收到的评论。

第二个案例可能会引起争议；有些人认为，在学生学会在没有 AI 的情况下写作之前，先学会用 AI 写作是可以的。我不同意，但无疑这场辩论将继续下去。

尽可能时，我更喜欢现实世界的案例设计，但好的 HCAI 案例很难找到。我的同事[约翰（伊恩）·麦克库洛赫](https://www.youtube.com/@dr_smoke)最近给了我一些很好的想法，这些想法来自他在课堂讲座中使用的例子，包括[器官捐赠案例](https://www.youtube.com/watch?v=A__M4O0Z5mY)，这是一个安永项目，帮助医生和患者快速、合理地做出时间敏感的肾脏移植决定。伊恩和我教的是同一个项目。我希望与伊恩合作，把这个案例变成明年的互动案例。

## 第三课：人工智能治理

大多数人同意 AI 的发展需要通过法律或其他手段进行管理，但对于如何管理存在很多分歧。

### 有什么新内容？

[欧盟的 AI 法案](https://artificialintelligenceact.eu/high-level-summary/)开始生效，为 AI 风险提供分层系统，并禁止一系列最高风险应用，包括社会评分系统和远程生物识别。AI 法案与欧盟的[数字市场法案](https://digital-markets-act.ec.europa.eu/index_en)和[通用数据保护条例](https://gdpr.eu/what-is-gdpr/)一起，形成了世界上最广泛和最全面的 AI 相关立法。

加利福尼亚通过了一系列与 AI 治理相关的法律，这可能具有全国性的影响，就像加利福尼亚关于环境等问题的法律经常设定先例一样。我喜欢来自 White & Case 律师事务所的[这篇（不完整）评论](https://www.whitecase.com/insight-alert/raft-california-ai-legislation-adds-growing-patchwork-us-regulation)。

对于隐私方面的国际比较，我喜欢 DLA Piper 的网站[世界数据保护法](https://www.dlapiperdataprotection.com/)。

### 什么重要？

我的课程将关注两个方面：

1.  我们应该如何评估新的立法

1.  立法如何适应 AI 治理的更大背景

### **如何评估新的立法？**

鉴于变化的步伐，我认为我能给我的班级提供的最有用的事情是一套评估新治理结构的启发式方法。

**注意定义。**每个新的法律都面临定义确切覆盖范围的问题；一些定义可能过于狭窄（通过改变方法的小幅调整就可以轻易规避），一些过于宽泛（容易受到滥用），而一些可能很快就会过时。

加利福尼亚必须解决一些困难的定义问题，以试图规范诸如“成瘾性媒体”（见[SB-976](https://leginfo.legislature.ca.gov/faces/billTextClient.xhtml?bill_id=202320240SB976#:~:text=(b)%20(1)%20%E2%80%9C,provides%20users%20with%20an%20addictive))、“AI 生成媒体”（见[AB-1836](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=202320240AB1836)）等问题，并为“生成性 AI”制定单独的立法（见[SB-896](https://leginfo.legislature.ca.gov/faces/billNavClient.xhtml?bill_id=202320240SB896)）。每个问题都有一些可能引起争议的方面，值得课堂讨论。例如，数字复制品法案将 AI 生成媒体定义为“一个具有不同自主程度的工程或机器系统，可以，为了明确或隐含的目标，从其接收的输入中推断出如何生成可以影响物理或虚拟环境的输出。”这里有很多解释的空间。

**谁受覆盖以及有哪些处罚？** 处罚是财务的还是刑事的？对于执法或政府使用是否有例外？它是如何跨越国际边界的？它是否有基于组织规模的分层系统？关于最后一点，技术监管通常试图通过合规的阈值或层级来保护初创公司和中小企业。但加利福尼亚州州长否决了 SB 1047 号关于人工智能安全的法案，以豁免小型公司，理由是“较小的、专业化的模型可能同样危险，甚至更危险”。这是一个明智的决定，还是他只是在保护加利福尼亚的技术巨头？

**它是否可执行、灵活，并且‘未来证明’？** 技术立法非常难以正确实施，因为技术是一个快速移动的目标。如果它过于具体，可能会很快过时，或者更糟，阻碍创新。但是，如果它更普遍或含糊，可能更难以执行，或者更容易被‘操纵’。一种策略是要求公司定义自己的风险和解决方案，这提供了灵活性，但只有在立法机构、法院和公众后来关注公司实际做什么时才能奏效。这是对一个高效运作的司法系统和积极参与、赋权的公民社会的赌注……但民主总是如此。

## 立法如何融入人工智能治理的更大图景？

并非每个问题都可以或应该通过立法来解决。人工智能治理是一个多层次的系统。它包括人工智能框架和独立的人工智能指导文件的激增，这些框架比立法做得更多，并提供非约束性、有时是理想主义的目标。我认为以下几项很重要：

+   NIST 的[人工智能风险管理框架](https://www.nist.gov/itl/ai-risk-management-framework)。NIST 在联邦政府内部享有良好的声誉，这个框架被用作许多其他工作的基础。

+   [圣克拉拉原则](https://santaclaraprinciples.org/)，专注于内容审核，得到了一些行业的认可，并具体指出了[政府监管者](https://santaclaraprinciples.org/regulators/)和[马克·扎克伯格](https://santaclaraprinciples.org/open-letter/)的合规性。

+   [人工智能暂停令](https://moratorium.ai/)针对的是长期、存在性的安全风险。‘[人工智能灾难性风险概述](https://arxiv.org/abs/2306.12001)’是一个很好的后续阅读材料。

+   专业协会也非常活跃；IEEE 在发布标准方面非常积极，并有一个关于[道德一致设计](https://standards.ieee.org/wp-content/uploads/import/documents/other/ead_v2.pdf)的标准，ACM 有一个专业的[道德准则](https://www.acm.org/code-of-ethics)。

+   [微软负责任的人工智能标准](https://blogs.microsoft.com/wp-content/uploads/prod/sites/5/2022/06/Microsoft-Responsible-AI-Standard-v2-General-Requirements-3.pdf)是具有具体要求、工具和实践的 AI 企业文件的范例。

+   这里是一个[许多其他框架](https://www.aiethicist.org/frameworks-guidelines-toolkits)的部分列表。

## 其他杂项主题

这里有一些其他新闻项目和主题，我将它们整合到我的课程中，其中一些是 2024 年新出现的，而另一些则不是。我将：

+   包含 2024 年畅销书[《焦虑一代》](https://www.anxiousgeneration.com/book)的摘要，这是一本关于社交媒体、智能手机及其相关生活方式变化相关危害的重要综合著作。我是乔纳森·海伊特工作的忠实粉丝。

+   注意在学期内是否会有关于[NY Times/ OpenAI 案件](https://hls.harvard.edu/today/does-chatgpt-violate-new-york-times-copyrights/)的裁决。

+   在隐私模块中提及[OnStar/ Lexus-Nexus 隐私侵犯](https://www.nytimes.com/2024/03/11/technology/carmakers-driver-tracking-insurance.html)的警示故事。

+   探讨 AI 的[日益增长的动力需求](https://e360.yale.edu/features/artificial-intelligence-climate-energy-emissions)以及与[AI 内容审核员](https://www.theguardian.com/technology/article/2024/jul/06/mercy-anita-african-workers-ai-artificial-intelligence-exploitation-feeding-machine)和数据标注员相关的可疑劳动实践问题。

+   给学生布置关于合成生物学危险的作业，从 RAND 的“[新生物武器](https://www.rand.org/pubs/external_publications/EP70594.html)”文章和一篇（付费墙）来自中国研究团队的文章“合成生物学时代生物安全风险治理的挑战和最新进展”（DOI：[10.1016/j.jobb.2022.02.002](http://dx.doi.org/10.1016/j.jobb.2022.02.002)）开始。学生可以通过“[合成生物学人工智能特刊](https://doi.org/10.1021/acssynbio.3c00760)”进行更深入的探讨。

+   包含更多关于 LLM 辅助错误信息的材料，包括优秀的入门文章“[在 LLM 时代对抗错误信息](https://onlinelibrary.wiley.com/doi/10.1002/aaai.12188)”和“[关于在假新闻中使用大型语言模型（LLMs）的调查](https://www.mdpi.com/1999-5903/16/8/298)”。通过“[科学相关错误信息校正效果的元分析](https://doi.org/10.1038/s41562-023-01623-8)”进行深入探讨。

+   添加关于[内容真实性倡议](https://contentauthenticity.org/how-it-works)和相关[C2PA](https://c2pa.org/)的材料。

+   将学生的预测与“[数千名 AI 作者关于 AI 未来的看法](https://arxiv.org/abs/2401.02843)”的汇总意见进行比较，这是我对每年使用的预测工作的更新。如果其他人有学生班级并想比较预测，我可以发送一个 Qualtrics 链接。

感谢阅读！我总是很欣赏与其他教授类似课程或对相关领域有深入了解的人建立联系。我也总是很欣赏点赞和评论！

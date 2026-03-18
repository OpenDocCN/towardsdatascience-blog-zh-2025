# 端到端数据科学家的提示手册

> 原文：[`towardsdatascience.com/the-end-to-end-data-scientists-prompt-playbook/`](https://towardsdatascience.com/the-end-to-end-data-scientists-prompt-playbook/)

<mdspan datatext="el1757129029805" class="mdspan-comment">你是否曾为自己的**代码**、**建模**和取得的**准确性**感到自豪，知道这真的可以为你的团队带来**差异**，但你却**挣扎**着与团队和利益相关者分享这些发现？

这是一种数据科学家和机器学习工程师中非常常见的感受。

在本文中，我将分享我的常用提示、工作流程和微小的技巧，这些技巧可以将密集、有时抽象的模型**输出**转化为清晰和明确的、人们真正关心的**商业叙事**。

如果你与**利益相关者**或**经理**合作，他们并非整天都生活在笔记本中，那么这篇文章就是为你准备的。就像我的其他指南一样，我会让它既实用又可复制粘贴。

本文是关于数据科学家提示工程的 3 篇文章系列的第三部分和最后一部分。

**端到端**数据科学提示工程系列包括：

+   **第一部分：**[规划、清理和 EDA 的提示工程](https://towardsdatascience.com/become-a-better-data-scientist-with-these-prompt-engineering-hacks/)

+   [**第二部分：**特征、建模和评估的提示工程](https://towardsdatascience.com/advanced-prompt-engineering-for-data-science-projects/)

+   **第三部分：文档、DevOps 和利益相关者沟通的提示**（本文）

👉 本文中的所有**提示**都可在文章末尾作为速查表获取 😉

在本文中：

1.  为什么 LLM 对数据故事讲述是一个颠覆性的变革

1.  使用 LLM 重新构想的沟通生命周期

1.  文档、DevOps 和利益相关者沟通的提示

1.  提示工程速查表

* * *

## 1) 为什么 LLM 对数据故事讲述是一个颠覆性的变革

LLM 将流畅的**写作**与**情境推理**相结合。在实践中，这意味着它们可以：

1.  用简单的英语（或任何其他语言）重新表述棘手的**指标**，

1.  几秒钟内起草高层**摘要**，

1.  为任何受众——董事会、产品、法律等——调整**语气**和**格式**。

早期研究表明，GPT 风格的模型实际上可以通过两位数的提升来增强**非技术**读者的**理解**。与仅仅盯着原始图表或图形相比，这是一个相当大的飞跃。

由于 LLM“**会说利益相关者的语言**”，它帮助你在不使人们淹没在术语中时捍卫决策。

如果提示工程之前感觉像是炒作，那么现在它成为了一个真正的优势：**清晰的故事**、更少的会议、更快的认可。

## 2) 使用 LLM 重新构想沟通生命周期

在训练和评估模型之后，你可能：

+   解释模型结果（SHAP、系数、混淆矩阵）。

+   总结 EDA 并指出注意事项。

+   起草高层简报、幻灯片脚本和“下一步要做什么”。

+   在备忘录和演示文稿中统一语气。

+   使用版本化的提示和快速更新来闭环。

现在：想象一个**助手**，它能撰写初稿，解释**权衡**，指出缺失的上下文，并保持作者声音的**一致性**。

这就是大型语言模型（LLM）能够做到的，如果你正确地提示它们的话！

## 3) 解释、报告和利益相关者参与的提示与模式

#### 3.1 SHAP & 特征重要性叙述

**最佳实践**：向模型提供结构化表格，并请求一份可供执行摘要以及行动方案。

```py
## System  
You are a senior data storyteller experienced in risk analytics and executive communication.  

## User  
Here are SHAP values in the format (feature, impact): {shap_table}.  

## Task  
1\. Rank the top-5 drivers of risk by absolute impact.  
2\. Write a ~120-word narrative explaining:  
   - What increases risk  
   - What reduces risk  
3\. End with two concrete mitigation actions.  

## Constraints & Style  
- Audience: Board-level, non-technical.  
- Format: Return output as Markdown bullets.  
- Clarity: Expand acronyms if present; flag and explain unclear feature names.  
- Tone: Crisp, confident, and insight-driven.  

## Examples  
- If a feature is named `loan_amt`, narrate it as "Loan Amount (the size of the loan)".  
- For mitigation, suggest actions such as "tighten lending criteria" or "increase monitoring of high-risk segments".  

## Evaluation Hook  
At the end, include a short self-check: "Confidence: X/10\. Any unclear features flagged: [list]." 
```

**为什么它有效**：结构迫使进行排名→叙述→行动。利益相关者得到的是“那又如何？”而不是图表上的条形图。

#### 3.2 混淆矩阵澄清

想象一下你的项目是针对一个金融平台的**欺诈检测**。

你已经训练了一个好的模型，你的精确度和召回率得分看起来很棒，你对它的表现感到自豪。但现在是你需要向你的**团队**解释这些结果的时候了，或者更糟糕的是，向一个对模型指标不太了解的**利益相关者**满屋子的听众解释。

这里有一个方便的表格，将混淆矩阵术语解释成简单的英文说明：

| 指标 | 平实的翻译 | 提示片段 |
| --- | --- | --- |
| 假阳性 | “警报了但实际上不是欺诈” | *解释 FP 为浪费的审查成本。* |
| 假阴性 | “错过了真实欺诈” | *将 FN 视为收入损失/风险暴露。* |
| 精确度 | “有多少警报是正确的” | *与 QA 假警报相关。* |
| Recall | “我们抓住了多少真实案例” | *使用“渔网孔洞”的类比。* |

**提示以简单解释模型结果**

```py
## System  
You are a data storyteller skilled at explaining model performance in business terms.  

## User  
Here is a confusion matrix: [[TN:1,500, FP:40], [FN:25, TP:435]].  

## Task  
- Explain this matrix in ≤80 words.  
- Stress the business cost of false positives (FP) vs false negatives (FN).  

## Constraints & Style  
- Audience: Call-center VP (non-technical, focused on cost & operations).  
- Tone: Clear, concise, cost-oriented.  
- Output: A short narrative paragraph.  

## Examples  
- "False positives waste agent time by reviewing customers who are actually fine."  
- "False negatives risk missing real churners, costing potential revenue."  

## Evaluation Hook  
End with a confidence score out of 10 on how well the explanation balances clarity and business relevance. 
```

#### 3.3 ROC & AUC—使权衡具体化

ROC 曲线和 AUC 分数是数据科学家最喜欢的一些指标之一，非常适合评估模型性能，但它们通常对于商业对话来说过于抽象。

为了让事情变得真实，将模型的**灵敏度**和**特异性**与实际业务限制（如时间、金钱或人力工作量）联系起来。

**提示**：

```py
“Highlight the trade-off between 95% sensitivity and marketing cost; suggest a cut-off if we must review ≤60 leads/day.”
```

这种框架将抽象的指标转化为具体的、可操作的决策。

#### 3.4 回归指标速查表

当你与回归模型一起工作时，这些指标可能会感觉像是一组随机的字母（MAE、RMSE、R²）。这对于模型调整很好，但对于讲故事来说就不那么好了。

这就是为什么使用简单的商业类比重新解释这些数字很有帮助：

| 指标 | 商业类比 | 一句话模板 |
| --- | --- | --- |
| MAE | “每份报价的平均美元误差” | *“我们的 MAE 为 2 美元意味着典型的报价误差是 2 美元。”* |
| RMSE | “大失误的惩罚增加” | *“RMSE 3.4 → 稀有但代价高昂的失误很重要。”* |
| R² | “我们解释的方差份额” | *“我们捕捉了 84% 的价格驱动因素。”* |

* * *

💥不要忘记检查本系列的[**第二部分**](https://towardsdatascience.com/advanced-prompt-engineering-for-data-science-projects/)，在那里你将学习如何改进你的**建模**和**特征工程**。

* * *

#### 4) 总结 EDA—提前提出警告

EDA 是真正的侦探工作开始的地方。但让我们面对现实：那些自动生成的配置文件报告（如 `pandas-profiling` 或摘要 JSON）可能会令人不知所措。

下一个提示有助于将 EDA 输出转换为简短且易于理解的人类友好型摘要。

**引导式 EDA 叙事者（pandas-profile 或摘要 JSON 输入，简要输出）：**

```py
## System  
You are a data-analysis narrator with expertise in exploratory data profiling.  

## User  
Input file: pandas_profile.json.  

## Task  
1\. Summarize key variable distributions in ≤150 words.  
2\. Flag variables with >25% missing data.  
3\. Recommend three transformations to improve quality or model readiness.  

## Constraints & Style  
- Audience: Product manager (non-technical but data-aware).  
- Tone: Accessible, insight-driven, solution-oriented.  
- Format:  
  - Short narrative summary  
  - Bullet list of flagged variables  
  - Bullet list of recommended transformations  

## Examples  
- Transformation examples: "Standardize categorical labels", "Log-transform skewed revenue variable", "Impute missing age with median".  

## Evaluation Hook  
End with a self-check: "Confidence: X/10\. Any flagged variables requiring domain input: [list]." 
```

### 5) 执行摘要、视觉大纲和幻灯片叙述

在数据建模和洞察力生成之后，还有一个最后的挑战：以决策者真正**关心**的方式讲述你的数据**故事**。

**框架快照**

+   **执行摘要指南提示：** 简介，要点，建议（≤500 字）。

+   **故事风格摘要：** 主要观点，关键统计数据，趋势线（≈200 字）。

+   **每周“强力提示”：** 两段简短段落 + “下一步”项目符号。

**复合提示**

```py
## System  
You are the Chief Analytics Communicator, expert at creating board-ready summaries.  

## User  
Input file: analysis_report.md.  

## Task  
Draft an executive summary (≤350 words) with the following structure:  
1\. Purpose (~40 words)  
2\. Key findings (Markdown bullets)  
3\. Revenue or risk impact estimate (quantified if possible)  
4\. Next actions with owners and dates  

## Constraints & Style  
- Audience: C-suite executives.  
- Tone: Assertive, confident, impact-driven.  
- Format: Structured sections with headings.  

## Examples  
- Key finding bullet: "Customer churn risk rose 8% in Q2, concentrated in enterprise accounts."  
- Action item bullet: "By Sept 15: VP of Sales to roll out targeted retention campaigns."  

## Evaluation Hook  
At the end, output: "Confidence: X/10\. Risks or assumptions that need executive input: [list]." 
```

#### 6) 语气、清晰度和格式

你已经有了洞察力和结论。现在是时候使它们清晰、自信且易于理解。

经验丰富的数据科学家知道，你如何表达某事有时甚至比你说什么更重要！

| 工具/提示 | 它的作用 | 典型用途 |
| --- | --- | --- |
| “语气重写器” | 正式 ↔ 非正式，或“董事会准备” | 客户更新，执行备忘录 |
| 海明威式编辑 | 缩短，增强动词 | 演示文稿副本，电子邮件 |
| “语气与清晰度审查” | 断言性语气，较少的保留 | 董事会材料，PRR 摘要 |

**通用重写提示**

```py
Revise the paragraph for senior-executive tone; keep ≤120 words. 
Retain numbers and units; add one persuasive stat if missing.
```

#### 7) 端到端 LLM 通信管道

1.  **模型输出 →** SHAP/指标 → 解释提示。

1.  **EDA 发现 →** 摘要提示或 LangChain 链。

1.  **自我检查 →** 让模型标记不清晰的特性或缺失的 KPI。

1.  **语气与格式通过 →** 专用重写提示。

1.  **版本控制 →** 将 `.prompty` 文件与笔记本一起存储以实现可重复性。

#### 8) 案例研究

| 组织 / 项目 | LLM 使用 | 结果 |
| --- | --- | --- |
| 金融科技信用评分 | SHAP 到叙事（“SHAPstories”）在仪表板内 | 增加了对利益相关者的理解 +20%；文档速度提高 10 倍 |
| 医疗保健初创公司 | Shiny 应用程序中的 ROC 解释器 | 临床医生在几分钟内就 92% 的敏感性截止点达成一致 |
| 零售分析 | 嵌入式表格摘要 | 3 小时撰写缩短到 ~12 分钟 |
| 大型财富部门 | 研究问答助手 | 每月 200k 查询；≈90% 满意度 |
| 全球 CMI 团队 | 通过 LLM 进行情感汇总 | 30 个地区的跨市场报告速度更快 |

#### 9) 最佳实践清单

+   在每个提示的前两行中定义受众、长度和语气。

+   提供结构化输入（JSON/表格）以减少幻觉。

+   嵌入 **自我评估**（“评分清晰度 0–1”；“标记缺失 KPI”）。

+   保持 **温度 ≤0.3** 以获得确定性的摘要；提高它以进行创意故事板。

+   **永远不要在没有单位的情况下改写数字**；保持原始指标可见。

+   版本控制提示 + 输出；将它们与 **模型版本** 相关联以进行审计跟踪。

* * *

#### 10) 常见陷阱与防护措施

| 障碍 | 症状 | 缓解措施 |
| --- | --- | --- |
| 发明驱动因素 | 叙事声称 SHAP 中没有的特征 | 通过严格的**特征白名单** |
| 过于技术化 | 利益相关者失去兴趣 | 添加“8 级阅读水平”+商业类比 |
| 语气不匹配 | 演示文稿/备忘录听起来不一样 | 运行批量语气重写流程 |
| 隐藏的警告 | 高管忽略小样本或抽样偏差 | 在每个提示中强制添加**限制**项目 |

这种“先讲错误”的习惯反映了我是如何结束我的数据科学生命周期作品的，因为误用几乎总是发生在早期，即在提示的时候。

* * *

**偷取此工作流程要点**：将每个指标视为一个等待讲述的故事，然后使用提示来标准化你的讲述方式。保持行动贴近，警告更近，让你的声音独一无二。

感谢您的阅读！

* * *

👉 **获取完整的提示清单** + 订阅**[Sara 的 AI 自动化摘要](https://saranfn.substack.com/)**时每周更新的实用 AI 工具——帮助技术专业人士每周用 AI 自动化实际工作。您还将获得访问 AI 工具库的权限。

我在这里提供 **职业成长和转型** 的**指导** [链接](https://topmate.io/sara_nobrega)。

**如果你想支持我的工作**，你可以 [买我喜欢的 **咖啡**](https://buymeacoffee.com/saranobregu)：一杯卡布奇诺。 😊

* * *

### 参考文献

[使用大型语言模型增强 SHAP 值的可解释性](https://arxiv.org/pdf/2409.00079)

[如何轻松总结数据表：提示嵌入式 LLM](https://www.quadratichq.com/blog/how-to-summarize-a-data-table-easily-prompt-an-embedded-llm)

[告诉我一个故事！大型语言模型驱动的叙事 XAI](https://www.insead.edu/faculty-research/publications/journal-articles/tell-me-a-story-narrative-driven-xai-large-language)

[使用 LLMs 提高数据通信 – Dataquest](https://www.dataquest.io/blog/llms-to-improve-data-communication/)

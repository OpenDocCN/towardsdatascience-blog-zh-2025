# 为什么营销人员转向准地理提升实验？（以及如何规划它们）

> 原文：[`towardsdatascience.com/why-are-marketers-turning-to-quasi-geo-lift-experiments-and-how-to-plan-them/`](https://towardsdatascience.com/why-are-marketers-turning-to-quasi-geo-lift-experiments-and-how-to-plan-them/)

**💡 注意：** 文章末尾的 Geo-Lift R 代码。

<mdspan datatext="el1758588746310" class="mdspan-comment">在我的职业生涯中，我使用了带有合成控制组的准实验设计来衡量商业变化的影响。</mdspan> 在 [Trustpilot](https://business.trustpilot.com/jobs) 上，我们更改了主页上横幅的位置，并回顾性地看到了指向我们主要需求指标下降的信号。通过模拟没有变化时的表现，并将其与实际发生的情况进行比较，我们得到了增量下降的明确读数。在 [Zensai](https://zensai.com/) 上，我们决定对一些网页表单进行处理，并以类似的方式确认变化是否对用户报告的完成度有积极影响。

> 这种方法适用于任何可以将某物分为处理组和控制组的情况：**个人、智能手机、操作系统、表单或着陆页**。你处理一些，其他的帮助你模拟没有处理会发生的情况。然后进行比较。

当你无法在个人层面进行随机化时，**地理**是一个很好的分析单位。城市和地区可以通过付费社交轻松定位，它们的边界有助于限制溢出效应。

有三种实际的设计用于隔离增量影响：

1.  **随机对照试验 (RCT**) 如**转化**或**品牌提升**测试，当它们可以在大型广告平台上运行时很有吸引力。它们自动化随机**用户**分配，并可以在平台环境中提供统计上稳健的答案。然而，除非你利用转化 API，否则它们的范围仅限于平台的自身指标和机制。最近，一些证据 ([1](https://download.ssrn.com/24/05/04/ssrn_id4817018_code346011.pdf?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEOz%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJIMEYCIQDyOtcHoexHZxtY3l6KH3gxSz6lfUdeWdG41EfeOsaT4wIhAOwHwq7t1gJPHmc8PzzXT07q%2F%2Bm304dYbjRpCAaKi4vRKr4FCGUQBBoMMzA4NDc1MzAxMjU3IgzJbu%2FnEKpwLZQEHhcqmwVi0JxVRkJY1uI%2B2zru2KE9yoMhJh13t6xznXu2PYOkr2WmeV7rIggKiBufGdvDFfeZFpdQtQpiMPNVH8mbVmq9FW5e75nZ5CoWLnFxaI%2B%2F5aufBxpIXoX5YpPSF7SvsJfiVIODH6pYLo0tibfjvgAwDv%2BLragU0is%2BPUDezyBBQHMjPaaqn%2BWbTpEOqN17qkpjMjxodnKbUHxSO6Cc1PDNK1YItijSAq4jaa7HdTDsGBhWO6hTTCuQk%2Ff2vbysVotzaeibextDoG3UVqAypbrI79PG747B6X9V7RlXsZeCGqSr7WYea0NCP244mp80PbwxN9LPW974V6sQfdiZpHhqazNfn7lrI9JEjh73jYrHXF5MJji91yUKuiTT737nHhroiZzohmBi%2FAkFmJnYm25xu8F%2F3euruXMzucsPPeNeFDCHJ6PUn7r9D%2F%2FzujfZmGbOZm2ZLq4B3w8Pf%2Fwpm%2FKSKbi0sOH2eI%2BuK4hUehHnjzpWga1iSfK6UCg9HbHyEaesqfMH1WT%2BjZIggRyEW7Legla8bcH%2BkDsthDqr92rsW4WmNMzgBLPBooDEKhneO4Sd%2F9ZNA35apSXdtpEzIbE8DI8PbnPHxBIFRDdmcSFHGksHPTThmHfSUYNXSn5WntvHt8iGDnB3az7G%2B0UHgV4ifU4dHA%2FYWt%2BTlN%2FtJODIaE5BoOxSKkjYQoJyeualP4vnhmw7n8UriUVLfgfNuFpq29C%2FrPzS8yQuOuUtJJhel%2F95CoGwMNF%2FHeLx72LWzuBv7N9qOsGKKFwQz7dtK%2BWsiyDOP6%2FbcmbO4%2BjuYe4uK9Jvos7AxBEofmgZCh6QXQ0ZcVUzsC2VG3BzjuZD6ToxtNHv6lfYxfkyxOKgFKYA4jfM%2BYgjOJOSv5RFMMywnMYGOrABPkNgqOoDZuhtv0etplrrcXMitFGepgltgvJ%2FE6ij6C36NIrvjfclqcC4UcstdO4ynw%2BAHebeG0dlqgLY11FYcEKxty%2FC%2Bge0DVPI38h8RMXYGSM66nr3Yz2njYz7KRY7uyu6qFtFcoPBEPcjUxZrU%2F1UfgN2JFHIUqZfoLTFKs0AFrEBPJWMEro%2FY%2F8YpzQH39qxKOnSiutB%2FO4ZM4nLNkYl880%2FBsmd9qdVq9%2B%2BysA%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250914T194826Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAUPUUPRWESGFYXSFQ%2F20250914%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=5849ed0e4aa9696b0d7560bef17a116d7b2dba1b900fccb94eded26897991747&abstractId=3896024), [2](https://pdf.sciencedirectassets.com/271657/1-s2.0-S0167811625X00057/1-s2.0-S0167811624001149/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEOz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIGnMfYPF%2BjpFYwwr4ICID7oAMzy7lqCB8d3NfjjDjrQxAiBZcPK%2B5b801TkjhGOTMvW9FBZJANzw0m%2Bg%2FiOOZBOsdiqzBQhkEAUaDDA1OTAwMzU0Njg2NSIMpQQrJ%2FZeHRyQYYSpKpAFNm83qTjJOtjKQ7MHs9il5Pcd2ZWW%2BcjKxDJ4SHAnnn7Lb2JqTT%2BCLUBOTeCNk3Fnr9CzK5JmZWUA4EIVrWPBoIWayvsNSI%2BuhBvBwzNUsDG0NKlMveG3knRpy7E2%2FA5zWEzyhSmHrtaXpDpI0kDac5lSh9dTUDPef25%2FCI1OPhE5%2FqlU%2FZBz6O1hNgrbIkFpw%2BTpJPnVq4xjtws8EYPjML0M0HEZxjxF0Ij5zc%2FHJfAfOVjvFm%2B3oIAd7r6HZzgmxZKDWBM91yoQRT%2B0cPA4TN5%2FmtkedT6%2F2lWB88J7VP%2BRqv63pIwWGwCro%2BmZu1sPDG0pyV6OHYRkYZWhSs6mm%2Bf%2F0cAB4Q81UpOnxdy2QMu8IPWns670%2FGd%2BguRKQjL4QXyCee5440HckVT%2BXzPtoaJdqhPfXKxwd%2Fs%2F3rYnYUuW6dqbRuYF%2FARVh90tus7XpF34b82%2FCCLP3f077HJhs80mTHF9CIYzPhjUUnGK885NS8%2Fc2GIBgyvgAx3M79ia9cYp7Z8mkacaItBOKefQbSTXWkvBlo%2FpGe3mEm3E9mbNJBcnBXMlnJYqOq3LEG3a9Jcq8GF98%2FGpoHzG8QsCuYyB3qVb3GxY13Y%2FUDYbzvKa%2FfU06bCTywzwXs5%2BJ4OP4FHvi9MrXV09D8jOpFICGEpuFXsUDuYXFD6gAqlvvg2mltPYdAhm%2Bqn4usenXkMUpeONQrt6jzhx%2BY11x75z4l%2B5VaWbTzFNarUNO4QZ4uy%2FYT0xLNfuarhSorvgxnH9WBRRbVesbrxE29z84RGdKGITuI7GC2Px6PX2zHQpQ%2FSa4axQVu3TtT59r5Y5H4brX9fpvUsYgjiZAVCptTE49o0bBMMTt9AKA2vgJMIm7sIwlaecxgY6sgFCptx5FQOBs1CWDufkOR0wMxAoW%2FRAxzRzW6LeeZQnbChDn1sYNu%2Bz%2B6KEvyM%2Bn0%2BKVen%2BmAWyiIyW9pOPurpyDkq6cp3EhUNwszlL7lrzoocac1VUMnI9MPhM0grDQi9kYpwfO0PPTqQtGJqFqvGMxPuNFFZLiPQZxeKnbUOwfoZ%2BqD0R4g7bGZkJD%2Bh1Mie9aGajmc%2B0l4pl5WD6E2gzL0eaNWfudcC%2FxQvd8Un8F7am&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250914T194920Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTY7L5QXKS5%2F20250914%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=8ec2dba88d8b01a11bb77967b133541e7c7931cad2dc61ad4e05a772e6e3b94f&hash=f6ce455cadeff33f92df972b22da2dd9df60b16bb642ea982e3b0f944a3cfc8f&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=S0167811624001149&tid=spdf-e17bc7ed-b5d3-4918-9f57-65223e010838&sid=89ab90a13a50a84cd41b2b39bbac2f058eeagxrqb&type=client&tsoh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&rh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&ua=100c5857005d505d56&rr=97f2683179b746d5&cc=sk), [3](https://www.pnas.org/doi/epdf/10.1073/pnas.1805363115)) 表明算法特征可能会损害分配的随机性。这导致了**“分叉交付”**：广告 A 被展示给一种类型的受众（例如，更多男性或具有特定兴趣的人），而广告 B 则被展示给另一种受众。任何点击率（CTR）或转化的差异都不能纯粹归因于广告创意。它还与算法如何交付广告有关。在 Google Ads 上，转化提升通常需要由 Google 账户代表启用。在 Meta 上，符合条件的广告主可以在广告管理器中运行自助转化提升测试（受支出和转化前提条件限制）。

1.  **地理随机对照试验（随机地理实验）**使用地理区域作为分析单位，**随机分配**给治疗组和对照组。无需进行个体层面的跟踪，但需要足够多的地理区域来实现统计功效和可靠的结果。由于分配是随机的，因此您不需要像在下一类实验中那样构建合成控制。

1.  **准地理提升实验**使用地理区域作为分析单位，无需进行个体层面的跟踪。与需要随机化和更多地理区域的地理随机对照试验（Geo-RCTs）不同，这种准实验方法提供了三个关键优势，无需进行随机化：它 **a)** **适用于较少的地理区域**（无需像 Geo-RCTs 那样需要许多地理区域），**b)** **允许** **战略市场选择**（根据业务优先级直接控制治疗的应用地点和时间），**c)** **适应** **回顾性分析和分阶段推出**（如果您的活动已经启动或必须因运营原因分阶段推出，您仍然可以测量增量）。合成控制构建以匹配干预前治疗单元的趋势和特征，因此干预开始后的任何差异都可以归因于活动的增量。**但是**：成功的执行需要分析和绩效营销团队之间有强大的协调，以确保适当的实施和解释。

准地理提升实验的好处很大。为了帮助您了解在您的营销科学职能中何时可能使用准实验，让我们考虑以下示例。

* * *

## 准地理提升实验示例

![](img/4ec24b7746759e7e92aae893449f827f.png)

***图 2:*** *准实验示例总结（来源：自行制作）*

> *考虑一家如 Bolt 的打车公司正在波兰评估一个新渠道。该业务已经关注这个新渠道一段时间，现在想要对其进行测试。需要了解这个名为 TweetX 的新渠道可能进行的营销活动对打车的影响，以及检测这种影响所需的最小预算。可用的数据集是一个覆盖 13 个波兰城市，从 2022 年开始的每日打车面板。建议的起始日期是 2023-09-18，激活可以是全国性的或基于城市的。存在实际限制：由于新渠道营销活动仅针对新用户，因此无法将用户纳入个别治疗组和对照组，这排除了用户级别的 A/B 测试。此外，隐私条件阻止了基于跟踪的归因的依赖。波兰其他渠道的历史先验表明，每趟行程的成本在 6 欧元到 12 欧元之间，CPM 在 2 欧元到 4 欧元之间，平均每趟行程的利润为 6 欧元。一个类似渠道 Snapcap 的最近基准月预算最高为 5000 欧元/月。*

这些条件有利于采用直接测量业务成果的准实验地理设计，**不** **依赖** **用户级跟踪** 或依赖 Meta 或 Google 等平台，而是使用常见的可用历史数据，如行程、销售额、转化或潜在客户。

在我们的示例中，为了执行具有合成控制组设计的地理提升测试，我们将使用 Meta 开发的 R 包 [**GeoLift**](https://facebookincubator.github.io/GeoLift/docs/GettingStarted/Walkthrough/)。我将使用的数据是按波兰 13 个城市的每日行程计数结构化的。

![](img/fde1ed9ec2214606d1c0f211ef6a77d8.png)

***图 3：*** *准地理提升实验数据集* *(来源：自行生产)*

### 在使用准地理提升实验时，你的数据最佳实践。

+   在可能的情况下，使用 **每日数据** 而不是每周数据。

+   使用 **最详细的位置数据**（例如，邮编、城市）。

+   在稳定、活动前的历史数据中，测试持续时间至少是 **4-5 倍**（没有重大变化或中断 - 更多内容将在下面的操作化章节中介绍！）

+   至少有 **25 个预处理期**，其中 **至少** **10 个**，但理想情况下 **20+ 地理单元**。

+   理想情况下，收集 **52 周的历史数据** 以捕捉季节性模式和其它因素。

+   测试应至少持续一个购买周期。

+   至少运行研究 **15 天（每日数据）** 或 **4-6 周（每周数据）**。

+   面板数据（协变量）有帮助，但不是必需的。

+   对于每个时间/地点，包括 **日期、地点和关键绩效指标**（无缺失值）。如果它们也符合此规则，可以添加额外的协变量。

* * *

## 规划你的准地理提升实验

![图 4：准地理提升实验规划（来源：自行生产）](img/dcd4042f83144d29ecd5bb17d16b4c38.png)

***图 4：*** *准地理提升实验规划* (来源：自行生产)

设计准地理提升实验不仅仅是进行测试，它关乎创建一个能够将营销行动与业务成果可信地联系起来的系统。

为了以结构化的方式进行，任何新渠道或活动的推出都应分阶段进行。以下是我的 **ABCDE** 框架，可以帮助你完成这项工作：

### （A）评估

+   确定增量如何衡量以及将使用哪种方法。

### （B）预算

+   确定检测到既具有统计可信度又具有商业意义的效应所需的最小支出。

### （C）构建

+   指定哪些城市将被处理，如何形成控制组，活动将运行多长时间，以及需要哪些操作限制。

### （D）交付

+   将统计结果转换为指标，并以读数形式报告结果。

### （E）评估

+   使用结果通过更新 MMM 和 MTA 来指导更广泛的决策。关注校准、压力测试、复制和本地化以进行推广。

* * *

## (A) 通过深入分析来**评估**所涉及的市场三角定位变量。

![](img/1c26dbf2121bad9872dd70be657a572f.png)

***图 5：*** *市场三角定位框架和增量分解*（来源：自行生产）

### 1. 将市场三角定位作为起点。

首先评估你需要分解的市场三角定位的哪一部分。在我们的案例中，那将是增量部分。对于其他项目，类似地深入到 MTA 和 MMM。例如，MTA 揭示了**启发式跳出盒子的技术**，如最后点击、第一次点击、首次或最后接触衰减、（倒置）U 形或 W 形，但也包括**数据驱动**的方法，如**马尔可夫**链。MMM 可以是定制的、第三方、Robyn 或 Meridian，并涉及饱和度、广告库存和预算再分配模拟等额外步骤。别忘了设置你的测量指标。

### 2. 增量性通过地理提升测试来操作化。

地理提升测试是框架的实践表达，因为它以业务管理的相同单位读取结果，例如每天每城市的乘车次数（或销售、转化、线索）。它就像一个经典随机研究一样创建治疗组和对照组，但它是在地理层面上进行的。

![](img/11c055f13d7b4e3e43e29a9f3349ad07.png)

***图 6：*** *地理提升测试与以用户为中心的 RCT 转化提升测试对比*（来源：自行生产）

> 这使得设计可以在多个平台上执行，并且独立于用户级跟踪，同时允许你研究你偏好的业务指标。

### 3. 识别地理提升测试的两个实验家族：RCTs 和准实验。

在平台内 RCTs（如转化提升测试或品牌提升测试）存在的地方（Meta、Google），它们仍然是标准，并且可以被利用。当个体随机化不可行时，地理提升测试随后作为准实验进行。

### 4. 识别依赖于合成控制方法。

对于每个受处理的城市，学习控制城市的加权组合以再现其测试窗口前的轨迹。观察序列与其合成对应物在测试窗口期间的差异被解释为增量效应。此估计器在保持执行可行和可审计的同时，保持了科学严谨性。

### 5. 校准和验证是明确的步骤，而不是事后考虑。

增量性的实验估计用于验证归因信号指向正确的方向，并**校准**MMM 弹性和广告库存（通过校准乘数），因此跨渠道预算再分配基于**因果**真实。

![](img/008c289f6e174eeadac56d67280c2fb1.png)

**图 7**：如何校准你的 MTA 和 MMM。利用准地理提升实验结果，使用校准乘数来校准你的 MTA/MMM（来源：自行生产）

**其他需要记住的校准点：**

+   a) 将每个实验视为一个分布，并使用提升、ROI 或 CPA 的**完整** **区间**，以保留不确定性。**如果你忽略区间，你会丢弃你支付的大部分费用，并冒着做出错误决策的风险。**

+   b) **结果** 仅与您进行测试的时间相关，因此随着您远离测试窗口，应给予较少的权重。**随着您远离窗口，您的信心应该减弱，因为平台和市场会发生变化。**

+   c) **营销** **绩效** **变化** 随时间推移。宏观环境的变化，你的营销效率结果也会随之变化，这是完全可以接受的，并不是红旗。**这对于校准很重要，因为今天你使用的模型中嵌入的非常旧的固定提升可能已经与现实脱节（主要由于拍卖、观众组合、创意质量和宏观因素）。**矛盾的是——你实际上可能从移除校准器中受益。

### 6. 用商业术语衡量影响。

在规划阶段，核心统计量是**平均处理效果（ATT）**，以每天的单位结果（例如，每天每座城市的乘车次数）表示。该估计值转换为测试窗口内的**总增量乘车次数**，然后通过将支出除以总增量乘车次数转换为**增量转化成本（CPIC）**。**最小可检测效应（MDE）**被报告出来，以明确设计的敏感性并区分可操作的结果和不可决的结果。最后，**净利润**通过结合历史骑手利润和增量结果以及 CPIC 来计算。

> 总增量线索可以通过将线索到客户的混合历史转化率相乘来估算，以估计活动预期产生多少新客户。然后，该数字乘以每位客户的平均利润（以美元计）。这样，即使收入在线索阶段之后实现，实验仍然提供了增量财务影响的透明估计和是否扩展渠道的明确决策规则。所有其他指标，如 ATE、总增量线索、增量线索成本和 MDE，将以类似的方式计算。

* * *

## (B) 为你的准实验制定预算。

![](img/3003031a7a05af77225d6ad400735ac0.png)

***图 8：*** *估算准地理提升实验的预算* *(来源：自行生产)*

**预算估算不是猜测**。它是一个设计选择，决定了实验是否产生可操作的结果或不可决的噪音。关键概念是**最小可检测效应（MDE）**：在历史数据方差、受处理和控制城市的数量以及测试窗口长度给定的情况下，测试可以可靠检测到的最小提升。

实际上，方差是从历史行程（或销售、转化或其他行业的线索）中估计的。处理城市的数量和测试长度定义了敏感性。例如，正如你稍后将会看到的，对 3 个城市进行 21 天的治疗，同时保持 10 个作为控制，足以检测到大约 4-5% 的提升。检测较小但具有统计学意义的效应需要更多时间、更多市场或更多支出。

> Geo-Lift 包会模拟这些功效分析，并为任何实验 ID 打印预算-效应-功效模拟图。

**预算与单元经济相一致**。在 Bolt 的案例中，行业先验知识表明每趟行程的成本为 6-12 欧元，每趟行程的利润为 6 欧元。在这些假设下，实现大约 5% 的 MDE 的最低消费为 **3 周的 3,038 欧元**，或者 **每天每处理城市 48.23 欧元**。这符合 5,000 欧元的基准预算，但更重要的是，它明确说明了测试可以和不能检测到的效应大小。

**以这种方式制定预算有两个优点**。首先，它确保实验的设计旨在检测对业务有意义的效应。其次，它为利益相关者提供了清晰的了解：如果结果是无效的，这意味着真实效应更有可能小于阈值，而不是测试执行得不好。无论如何，支出都没有浪费。它购买了因果知识，这有助于未来分配决策的准确性。

* * *

## (C) **构建你的准实验设计**。

![图片](img/ab60551dee5c63147fdb58dfef4ebd84.png)

***图 9：*** *准地理提升实验中的处理和控制地理选择* *(来源：自行生产)*

**设计实验不仅仅是随机选择城市**。这是关于构建一个既保持有效性又适合运营的布局。分析的单位是**城市-天**，结果是感兴趣的商务指标，例如行程、销售额或线索。治疗应用于选定的城市，在一个固定的测试窗口内进行，而其余的城市作为控制组。

> Geo-Lift 将为您的治疗建模和分组理想的城市候选人。

### 控制组不会被简单地保留原样。

它们可以通过**合成控制方法**进行细化。每个处理城市都与一个加权组合的控制城市配对，以复制其测试前的轨迹。当测试前期的拟合准确时，发布后观察结果和合成结果之间的差异提供了增量效应的可靠估计。

### 运营的护栏对于保护信号质量至关重要。

![图片](img/3e0e0d5e3247950ba52669d60e5400a5.png)

***图 10：*** *准地理提升实验规划和设计* *(来源：自行生产)*

在设置中应紧密围栏城市边界以减少通勤者的溢出。还应考虑在设置中严格排除控制城市与处理城市，反之亦然。不应将特定的**目标**和**相似**受众应用于活动。可能**混淆**测试的本地促销要么在处理地理区域内**冻结**，要么在控制组中**镜像**。

在窗口期内，创意、出价和节奏保持不变，由于营销**库存**效应，结果仅在短暂的冷却期后读取。**其他**与**商业**相关的**因素**应予以考虑，例如，在我们的案例中，我们应该提前检查供应驱动能力的容量，以确保额外的需求能够得到满足，而不会扭曲价格或等待时间。

### 构建测试意味着选择正确的平衡。

![图片](img/68cb4768049d4549d06319922721ce60.png)

**图 11：***模拟和排名的准地理提升实验组* *(来源：自行生产)*

> Geo-Lift 包将通过建模和构建所有最佳实验来为你做繁重的工作。

从你的功率分析和市场选择功能代码中，选择正确的实验设置是在以下方面之间的平衡：

+   城市数量

+   持续时间

+   花费

+   所需的增量提升

+   利润

+   统计显著性

+   测试与控制对齐

+   最小可检测的提升

+   其他商业环境

在 21 天期间对三个处理城市进行配置，如 Bolt 示例所示，可以提供足够的功率来检测约 4-5%的提升，同时保持测试窗口足够短，以最大限度地减少**污染**。根据**先前**的**实验**，我们知道这种水平的功率是**足够的**。此外，我们需要保持在每月**€5,000**的预算内，并且预计三周的投资€3,038.16 很好地符合这一限制。

* * *

## (D) 提供实验后的读数。

![图片](img/92c3a43472aad4f34735bcb8f2c5b7b0.png)

**图 12：***准地理提升实验的结果* *(来源：自行生产)*

最后一步是将结果转化为将统计提升转化为对利益相关者有明确商业影响的成果。实验的输出应以分析师和决策者都能使用的术语来表述。

核心是**平均处理效果（ATT）**，以每天的单位结果（如行程、销售额或线索）表示。据此，分析计算测试窗口内的**总增量结果**，并通过将支出除以这些结果来推导**增量转换成本（CPIC）**。**最小可检测效果（MDE）**与结果一起报告，以使测试的敏感性透明，将可操作的提升与不确定的噪声分开。最后，分析通过结合增量转换与单位经济学将结果转化为**净利润**。

对于**基于铅**的业务，同样的逻辑适用，但净利润：为此，可以将总增量潜在客户数乘以潜在客户到客户的综合转化率，然后乘以每客户的平均利润，以近似净财务影响。

> 在向利益相关者解释结果时要谨慎。使用 p 值进行统计分析提供的是证据，而不是绝对的证明，因此措辞很重要。GeoLift [使用](https://facebookincubator.github.io/GeoLift/docs/Methodology/Confidence_Interval_Explanation/) 基于合成控制/增强合成控制的**频率推断**。

对统计显著性的**常见但误导性的解释**可能听起来像这样：

> *“准 Geo-Lift 实验的结果证明，由于该活动，乘车次数增加了 11.1%，有 95%的概率。”*

这种解释存在几个问题：

+   它**将统计显著性视为证据**。

+   它**假设**效应大小是精确的（11.1%），忽略了围绕该估计的不确定性范围。

+   它**误解**了**置信区间**。

+   它**没有**为可能创造业务价值的替代探索留下空间。

+   它可能会**误导决策者**，造成过度自信，并可能导致风险业务选择。

### 在进行准 Geo-Lift 测试时，统计测试实际上测试的是什么？

每个统计测试都依赖于一个**统计模型**，这是一个复杂的假设网络。这个模型不仅包括正在测试的主要假设（例如，新的 TweetX 活动没有影响），还包括关于数据生成方式的长长列表的其他假设。这些包括关于：

+   随机抽样或随机化。

+   数据遵循的概率分布类型。

+   独立性。

+   选择偏差。

+   没有重大测量错误。

+   分析是如何进行的。

> 统计测试**不仅**评估测试假设（如零假设）。它评估的是**整个统计模型**：完整的假设集。**这就是为什么**我们总是试图确保所有其他假设都完全满足，实验设计没有缺陷：这样，如果我们观察到小的 p 值，我们可以合理地将其解读为对零假设的证据，而不是证明或“接受”H1。

### P 值是兼容性的衡量，而不是真理。

P 值的常见定义是错误的。一个更准确、更有用的定义是：

> p 值是衡量观察到的数据与用于分析它们的整个统计模型**兼容性**的定量指标——也就是说，测试假设*以及*所有其他假设（抽样、测量、分析选择等）。如果 p 值为 0.05，如果没有实际效果，我们预计只有大约 5%的时间会看到这种极端的结果。它不是关于可能性，而是关于抽样或实验设置的变异性。它**不是**假设正确或错误的概率，也不是仅机会的衡量。

将其视为一个“惊喜指数”。

+   一个**小的 P 值**（例如，P=0.01）表明，如果整个模型都是真的，数据将是令人惊讶的。这是一个红旗，告诉我们我们的一个或多个假设可能是不正确的。然而，它**并没有告诉我们哪个假设是错误的**。问题可能是零假设，也可能是违反了研究方案、选择偏差或其他未满足的假设。**这就是为什么**我们总是尽力确保所有其他假设都得到充分满足，实验设计没有缺陷：这样，如果我们观察到小的 p 值，我们可以合理地将其解读为对零假设的证据，而不是作为对 H1 的证明或“接受”。

+   一个**大的 P 值**（例如，P=0.40）表明，在模型下，数据并不异常或令人惊讶。它表明数据与模型兼容，但它**并不证明模型或测试假设是正确的**。数据可以与其他许多模型和假设同样兼容。

将 P 值简化为简单的二元选择，“具有统计学意义”（P≤0.05）或“不具有统计学意义”（P>0.05）的常见做法是有害的。它创造了一种虚假的确定性，并忽略了实际证据的数量。

### 置信区间（CI）及其在准 Geo-Lift 测试中的重要性。

置信区间（CI）比从零假设检验得到的简单 P 值更有信息量。它可以理解为在给定的统计模型下，与数据相对**兼容**的效应量范围。

> 95%置信区间具有特定的频率特性：如果你用有效的统计模型无数次地重复进行准 Geo-Lift 研究，**平均而言，95%的计算置信区间将包含真实效应量**。

关键的是，这并不意味着有 95%的概率你的特定区间包含真实效应。一旦计算出来，你的区间要么包含真实值，要么不包含（0%或 100%）。这里的“95%”告诉我们，在许多重复研究中，这种方法捕捉到真实效应的频率，而不是我们对这个单一区间的确定性。如果你想从频率主义的置信区间转向关于提升的直接概率，**贝叶斯**方法是途径。

* * *

如果你想深入了解 P 值、置信区间和假设检验，我推荐以下两篇知名论文：

+   *《十二个 P 值误解：十二个 P 值误解》* 链接[**这里**](https://sixsigmadsi.com/wp-content/uploads/2020/10/A-Dirty-Dozen-Twelve-P-Value-Misconceptions.pdf)**。

+   *《统计测试、P 值、置信区间和功效：误解释指南》* 链接[**这里**](https://link.springer.com/article/10.1007/s10654-016-0149-3)**。

* * *

## (E) 使用更广阔的视角**评估**你的准 Geo-Lift 实验。

![](img/388c5f785e821b270ebd99039e02ef7c.png)

***图 13：*** *实验后评估和下一步行动* *(来源：自行制作)*

> **记住要放大视角，‘看到森林而不是树木’**。

结果必须通过**验证**归因 ROAS 和**校准**营销组合模型与因果乘数来反馈到营销三角化。

他们还应该指导下一步：在**新****地理**中**复制**积极结果，评估增量百分比与可检测最小阈值，并在进一步测试之前**避免**从单一市场**推广**。

**使用安慰剂测试**（空间或时间）进行压力测试也将增强对您发现结果的信心。

![](img/4ef035633b5c40752ffd72f114ec9e47.png)

***图 14：*** *五个随机即时安慰剂测试* *(来源：自产)*

以下是来自一个**即时安慰剂**的结果：安慰剂增加 1.3%并不足以统计上强烈拒绝处理组和对照组之间没有差异 = H0（正如安慰剂所预期）：

![](img/edb77656f1adc261fa3b5d2f9f06851d.png)

***图 15：*** *即时安慰剂测试结果* *(来源：自产)*

**将**渠道**曝光**和**花费**纳入 MMM 中的跨渠道互动有助于捕捉媒体组合中**其他**渠道的**互动**。

如果尽管计划表明应该实现预期的提升，但活动未能实现，那么评估在定量结果中未捕获的因素很重要：**信息**或**创意**执行。通常，不足可能归因于活动（没有）与目标受众产生共鸣，而不是实验设计中的缺陷。

## 这对你有什么好处？

准地理提升让您能够在**无需用户级跟踪或大型随机测试**的情况下证明活动是否真正推动了指针。您选择几个市场，从其余部分构建一个合成控制，并直接在业务单元（乘车、销售、线索）中读取**增量**影响。ABCDE 计划使其变得实用：

+   **评估**营销三角化和您将如何衡量，

+   **预算**到明确的 MDE，

+   **构建**处理/控制和护栏，

+   **交付**ATT → CPIC → 利润，然后

+   **评估**通过校准 MMM/MTA，使用安慰剂进行压力测试，以及通过更广阔的视角观察您的业务环境。

净结果？更快、更便宜、有据可依的答案，您可以采取行动。

* * *

感谢您阅读。如果您喜欢这篇文章或学到了新东西，请随时**联系**并联系我[LinkedIn](https://www.linkedin.com/in/tomas-jancovic/)。

* * *

<details class="wp-block-details is-layout-flow wp-block-details-is-layout-flow"><summary>完整代码：</summary>

```py
library(tidyr)
library(dplyr)
library(GeoLift)

# Assuming long_data is your pre-formatted dataset with columns: date, location, Y
# The data should be loaded into your R environment before running this code.

long_data <- read.csv("/Users/tomasjancovic/Downloads/long_data.csv")

# Market selection (power analysis)
GeoLift_PreTest <- long_data
GeoLift_PreTest$date <- as.Date(GeoLift_PreTest$date)

# using data up to 2023-09-18 (day before launch)
GeoTestData_PreTest <- GeoDataRead(
  data = GeoLift_PreTest[GeoLift_PreTest$date < '2023-09-18', ],
  date_id = "date",
  location_id = "location",
  Y_id = "Y",
  format = "yyyy-mm-dd",
  summary = TRUE
)

# overview plot
GeoPlot(GeoTestData_PreTest, Y_id = "Y", time_id = "time", location_id = "location")

# power analysis & market selection
MarketSelections <- GeoLiftMarketSelection(
  data = GeoTestData_PreTest,
  treatment_periods = c(14, 21, 28, 35, 42),
  N = c(1, 2, 3, 4, 5),
  Y_id = "Y",
  location_id = "location",
  time_id = "time",
  effect_size = seq(0, 0.26, 0.02),
  cpic = 6,
  budget = 5000,
  alpha = 0.05,
  fixed_effects = TRUE,
  side_of_test = "one_sided"
)

print(MarketSelections)
plot(MarketSelections, market_ID = 4, print_summary = TRUE)

# ------------- simulation starts, you would use your observed treatment/control groups data instead

# parameters
treatment_cities <- c("Zabrze", "Szczecin", "Czestochowa")
lift_magnitude <- 0.11
treatment_start_date <- as.Date('2023-09-18')
treatment_duration <- 21
treatment_end_date <- treatment_start_date + (treatment_duration - 1)

# extending the time series
extend_time_series <- function(data, extend_days) {
  extended_data <- data.frame()

  for (city in unique(data$location)) {
    city_data <- data %>% filter(location == city) %>% arrange(date)

    baseline_value <- mean(tail(city_data$Y, 30))

    recent_data <- tail(city_data, 60) %>%
      mutate(dow = as.numeric(format(date, "%u")))

    dow_effects <- recent_data %>%
      group_by(dow) %>%
      summarise(dow_multiplier = mean(Y) / mean(recent_data$Y), .groups = 'drop')

    last_date <- max(city_data$date)
    extended_dates <- seq(from = last_date + 1, by = "day", length.out = extend_days)

    extended_values <- sapply(extended_dates, function(date) {
      dow <- as.numeric(format(date, "%u"))
      multiplier <- dow_effects$dow_multiplier[dow_effects$dow == dow]
      if (length(multiplier) == 0) multiplier <- 1

      value <- baseline_value * multiplier + rnorm(1, 0, sd(city_data$Y) * 0.1)
      max(0, round(value))
    })

    extended_data <- rbind(extended_data, data.frame(
      date = extended_dates,
      location = city,
      Y = extended_values
    ))
  }

  return(extended_data)
}

# extending to treatment_end_date
original_end_date <- max(long_data$date)
days_to_extend <- as.numeric(treatment_end_date - original_end_date)

set.seed(123)
extended_data <- extend_time_series(long_data, days_to_extend)

# Combining original + extended
full_data <- rbind(
  long_data %>% select(date, location, Y),
  extended_data
) %>% arrange(date, location)

# applying treatment effect
simulated_data <- full_data %>%
  mutate(
    Y_original = Y,
    Y = if_else(
      location %in% treatment_cities &
        date >= treatment_start_date &
        date <= treatment_end_date,
      Y * (1 + lift_magnitude),
      Y
    )
  )

# Verifying treatment (prints just the table)
verification <- simulated_data %>%
  filter(location %in% treatment_cities,
         date >= treatment_start_date,
         date <= treatment_end_date) %>%
  group_by(location) %>%
  summarize(actual_lift = (mean(Y) / mean(Y_original)) - 1, .groups = 'drop')

print(verification)

# building GeoLift input (simulated)
GeoTestData_Full <- GeoDataRead(
  data = simulated_data %>% select(date, location, Y),
  date_id = "date",
  location_id = "location",
  Y_id = "Y",
  format = "yyyy-mm-dd",
  summary = TRUE
)

# Computing time indices
date_sequence <- seq(from = min(full_data$date), to = max(full_data$date), by = "day")
treatment_start_time <- which(date_sequence == treatment_start_date)
treatment_end_time <- which(date_sequence == treatment_end_date)

# Running GeoLift
GeoLift_Results <- GeoLift(
  Y_id = "Y",
  data = GeoTestData_Full,
  locations = treatment_cities,
  treatment_start_time = treatment_start_time,
  treatment_end_time = treatment_end_time,
  model = "None",
  fixed_effects = TRUE
)

# ---------------- simulation ends!

# plots
summary(GeoLift_Results)
plot(GeoLift_Results)
plot(GeoLift_Results, type = "ATT")

# placebos
set.seed(42)

# window length (days) of the real treatment
window_len <- treatment_end_time - treatment_start_time + 1

# the furthest you can shift back while keeping the full window inside pre-period
max_shift <- treatment_start_time - window_len
n_placebos <- 5

random_shifts <- sample(1:max(1, max_shift), size = min(n_placebos, max_shift), replace = FALSE)

placebo_random_shift <- vector("list", length(random_shifts))
names(placebo_random_shift) <- paste0("Shift_", random_shifts)

for (i in seq_along(random_shifts)) {
  s <- random_shifts[i]
  placebo_random_shift[[i]] <- GeoLift(
    Y_id = "Y",
    data = GeoTestData_Full,
    locations = treatment_cities,
    treatment_start_time = treatment_start_time - s,
    treatment_end_time   = treatment_end_time   - s,
    model = "None",
    fixed_effects = TRUE
  )
}

# --- Print summaries for each random-shift placebo ---
for (i in seq_along(placebo_random_shift)) {
  s <- random_shifts[i]
  cat("\n=== Summary for Random Shift", s, "days ===\n")
  print(summary(placebo_random_shift[[i]]))
}

# Plot ATT for each random-shift placebo
for (i in seq_along(placebo_random_shift)) {
  s <- random_shifts[i]
  placebo_end_date <- treatment_end_date - s
  cat("\n=== ATT Plot for Random Shift", s, "days ===\n")
  print(plot(placebo_random_shift[[i]], type = "ATT", treatment_end_date = placebo_end_date))
}
```</details>

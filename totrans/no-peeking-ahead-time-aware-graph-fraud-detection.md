# 不提前查看：时间感知图欺诈检测

> 原文：[`towardsdatascience.com/no-peeking-ahead-time-aware-graph-fraud-detection/`](https://towardsdatascience.com/no-peeking-ahead-time-aware-graph-fraud-detection/)

在我上一篇文章[[1](https://medium.com/@erikapatg/applications-and-opportunities-of-graphs-in-insurance-0078564271ab)]中，我提出了一些围绕构建结构化图的想法，主要关注通过图结构对数据进行描述性或无监督的探索。然而，当我们使用图特征来改进我们的模型时，必须考虑到数据的**时间特性**。如果我们想避免不希望的效果，我们需要小心不要将未来的信息**泄露到我们的训练过程中**。这意味着我们的图（以及从中派生出的特征）必须以时间感知、增量方式构建。

> 数据泄露是一个如此矛盾的问题，以至于 Sayash Kapoor 和 Arvind Narayanan 在 2023 年的一项研究发现，截至那时，它已经影响了 17 个科学领域的 294 篇研究论文。他们将数据泄露的类型**从教科书错误到开放的研究问题**进行了分类。

问题在于，在原型设计阶段，结果往往看起来非常有希望，而实际上并非如此。大多数时候，人们直到模型在生产中部署时才意识到这一点，**浪费了整个团队的时间和资源**。然后，性能通常低于预期，而人们却不知道为什么。这个问题可能成为**阿基里斯的脚踵**，破坏所有商业人工智能倡议。

…

## 基于机器学习的泄露

> 数据泄露发生在训练数据包含在推理期间不可用的输出信息时。这导致在开发期间评估指标过于乐观，产生了误导性的期望。然而，当在具有适当数据流的真实时间系统中部署时，模型预测变得不可信，因为它学习了无法访问的信息。

**道德上**，我们必须努力产生真正反映我们模型能力的成果，而不是**耸人听闻或误导性的发现**。当模型从原型设计过渡到生产阶段时，它不应未能正确泛化；如果它确实如此，其实际价值就会受到损害。**泛化能力差的模型在推理或部署过程中可能会出现重大问题**，从而损害其有用性。

这在像欺诈检测这样的敏感环境中尤其危险，它通常涉及不平衡的数据场景（欺诈案例少于非欺诈案例）。在这些情况下，**数据泄露造成的危害更为明显，因为模型可能会过度拟合与少数类相关的泄露数据**，为少数标签产生看似良好的结果，而这是最难预测的。这可能导致欺诈检测的遗漏，从而产生严重的实际后果。

数据泄露的例子可以分为教科书错误和开放研究问题[2]，如下所示：

**教科书错误：**

+   使用整个数据集而不是仅使用训练集来填补缺失值，导致有关测试数据的信息泄露到训练中。

+   在训练集和测试集中出现重复或非常相似的实例，例如从略微不同的角度拍摄的同一种物体的图像。

+   训练集和测试集之间缺乏清晰的分离，或者根本没有测试集，导致模型在评估之前就能访问测试信息。

+   使用结果变量的代理来间接揭示目标变量。

+   在多个相关记录属于单个实体的场景中进行随机数据拆分，例如来自同一客户的多个索赔状态事件。

+   在整个数据集上而不是仅在训练集上执行合成数据增强。

**研究开放性问题：**

+   当未来数据无意中影响训练时，就会发生时间泄露。在这种情况下，严格的分离是具有挑战性的，因为时间戳可能是嘈杂或不完整的。

+   在没有谱系或审计跟踪的情况下更新数据库记录，例如，在没有存储历史记录的情况下更改欺诈状态，可能导致模型无意中在未来的或更改的数据上训练。

+   复杂的实际情况数据集成和管道问题，这些问题通过配置错误或缺乏控制引入了泄露。

这些案例是机器学习研究中报告的更广泛分类法的一部分，突出了数据泄露作为可靠建模中的一个关键且常常被低估的风险[3]。即使是在简单的表格数据中，这些问题也可能出现，并且如果每个特征都没有单独检查，它们**可能在处理许多特征时保持隐藏**。

现在，让我们考虑**当我们把节点和边包括在方程中时会发生什么**...

…

## 图基础泄露

在基于图模型的案例中，泄露可能比传统表格设置更隐蔽。当特征是从连接组件或拓扑结构派生时，使用未来的节点或边可能会无声地改变图的结构。例如：

+   例如，**图神经网络（GNNs）不仅从单个节点学习上下文，还从它们的邻居学习上下文**，如果在训练过程中在图结构中传播敏感或未来信息，可能会无意中引入泄露。

+   当图结构被覆盖或更新而没有保留过去的事件时，意味着**模型失去了进行准确时间分析所需的有价值上下文**，并且它可能再次在错误的时间访问信息或失去对可能泄露或与图形起源数据相关问题的追踪性。

+   在整个图上计算图聚合（如度、三角形或 PageRank）而不考虑时间维度（时间无关聚合）使用所有边：过去、现在和未来。这导致数据泄露，因为特征包括来自未来边的信息，这些信息在预测时间是不可用的。

> 当在训练中包含来自未来时间点的特征、边或节点关系，以违反事件的时间顺序时，会发生图时间泄露。这导致包含应未知的时间步数据的边或训练特征。

…

## **如何解决这个问题？**

我们可以通过为边分配**时间戳或时间间隔**来构建一个捕获整个历史的单个图。为了分析到特定时间点（t）的图，我们通过过滤任何图以仅包括在该截止点之前或发生的所有事件来“回顾过去”。这种方法对于防止数据泄露是理想的，因为它确保仅使用过去和现在的信息进行建模。此外，它提供了定义不同时间窗口进行安全和准确时序分析的灵活性。

在这篇文章中，我们构建了一个**保险索赔的时间图**，其中节点代表单个索赔，当两个索赔共享一个实体（例如，电话号码、车牌号、维修店等）时创建时间链接以确保正确的事件顺序。然后计算基于图的特征来喂养欺诈预测模型，小心避免使用未来信息（**不要偷看**）。

这个想法很简单：**如果两个断言共享一个共同实体，并且一个发生在另一个之前**，我们在这种连接变得可见的时刻将它们连接起来（见图 1）。正如我们在上一节中解释的，我们建模数据的方式至关重要，不仅是为了捕捉我们真正寻找的东西，而且是为了使高级方法（如图神经网络 GNNs）的使用成为可能。

![图片](img/a0ab67141ef5fc26a143967ab68e29f5.png)

**图 1：** 随着时间的推移，断言和实体（如电话号码）被添加到图中。当一个新断言（在时间 t 的 Claim_id2）与一个较早的断言（在时间 t-1 的 Claim_id1）共享之前观察到的实体时，从较早的断言到较晚的断言创建一个有向时间边（蓝色箭头）。这种构建揭示了关系何时变得可见，并确保图中的因果、时间尊重的连通性。图片由作者提供。

在我们的图模型中，我们保存实体首次出现的时间戳，捕捉它在数据中出现的时刻。然而，在许多现实世界场景中，考虑实体首次和最后一次出现之间的时间间隔（例如，通过另一个变量如车牌号或电子邮件生成）也是很有用的。这个间隔可以提供更丰富的时序背景，反映节点和边的生命周期或活跃期，这对于动态时序图分析和高级模型训练非常有价值。

## 代码

代码存储在本存储库中：[链接到存储库](https://github.com/erikapat/graphs_neo4j/tree/feature/first_steps/temporal_graph)

要运行实验，请设置一个 Python ≥3.11 环境，并安装所需的[库](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/requirements.txt)（例如，**torch**，**torch-geometric**，**networkx**等）。建议使用虚拟环境（通过**venv**或**conda**）来保持依赖项的隔离。

## 代码管道

图 2 的图表显示了使用 GraphSAGE 进行欺诈检测的端到端工作流程。**步骤 1**加载（模拟的）原始索赔数据。**步骤 2**构建一个带时间戳的有向图（实体→索赔和旧索赔→新索赔）。**步骤 3**执行时间切片以创建训练、验证和测试集，然后索引节点、构建特征，并最终训练和验证模型。

![图片](img/d60f5aac8e9bcb0826fa3d0315163f84.png)

图 2.时间欺诈建模的端到端管道：**（I）**加载数据→**（II）**构建并保存带时间戳的图和**（III）**准备子图（时间切片→节点索引→特征构建→PyG `Data`）以进行训练和推理。图由作者提供。

### 步骤 1：模拟欺诈数据集

我们首先[模拟一个保险索赔数据集](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/1_simulation.py)。数据集[数据集](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/data/sy_dataset_1.csv)中的每一行代表一个索赔，并包括如下变量：

+   **实体**：`insurer_license_plate`，`insurer_phone_number`，`insurer_email`，`insurer_address`，`repair_shop`，`bank_account`，`claim_location`，`third_party_license_plate`

+   **核心信息**：`claim_id`，`claim_date`，`type_of_claim`，`insurer_id`，`insurer_name`

+   **目标**：`fraud`（二元变量，指示索赔是否为欺诈）

这些实体属性作为索赔之间的潜在链接，使我们能够通过共享值（例如，使用相同维修店或电话号码的两个索赔）推断出连接。通过将这些隐含关系建模为图中的边，我们可以构建强大的拓扑表示，捕捉可疑的行为模式，并使下游任务（如特征工程或基于图的学习）成为可能。

![图片](img/305885c3411bc0990718ef4a3b64e80e.png)

表 1.模拟保险索赔数据的概述，显示了每个记录的关键实体字段和欺诈标签。表由作者提供。

![图片](img/975b794759ab3136be363c8750c9024e.png)

图 3.模拟数据集中欺诈和非欺诈索赔的分布（左）和欺诈率随索赔量的每日演变（右）。欺诈基础率约为 12.45%。图由作者提供。

### 步骤 2：图建模

我们使用 NetworkX 库来构建我们的[图模型](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/3_graph_creation.py)。对于**小型示例，NetworkX 足够且有效**。对于更高级的图处理，可以使用 Memgraph 或 Neo4j 等工具。要使用 NetworkX 建模，我们创建表示实体及其关系的节点和边，从而在 Python 中实现网络分析和可视化。

因此，我们有：

+   每个陈述一个节点，节点键等于`claim_id`，属性为`node_type`和`claim_date`

+   每个实体值（电话、车牌、银行账户、商店等）一个节点。**节点键**：`"{column_name}:{value}"` 和属性 `node_type = <column_name>`（例如，`"insurer_phone_number"`，`"bank_account"`，`"repair_shop"`）`label = <value>`（仅是原始值，不带前缀）

图包括这两种类型的**边**：

+   `claim_id(t-1)`→ `claim_id(t)` ：当两个陈述共享一个实体（`edge_type='claim-claim'`）

+   `entity_value` →`claim_id`：直接链接到共享实体（`edge_type='entity-claim'`）

这些边用以下方式标注：

+   `edge_type`：区分关系（`claim`→`claim` 与 `entity`→`claim`）

+   `entity_type`：值所在的列（如`bank_account`）

+   `shared_value`：实际值（如电话号码或车牌号）

+   `timestamp`：边被添加的时间（基于当前陈述的日期）

为了解释我们的模拟，我们实现了一个[脚本，用于生成解释](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/2_why.py)，说明为什么一个陈述被标记为欺诈。在图 4 中，陈述 20000695 被认为有风险，主要是因为它与维修店 SHOP_856 有关，该维修店作为一个活跃的中心，在相似日期周围有多个陈述与之相连，这是欺诈“爆发”中常见的一种模式。此外，该陈述与几个其他陈述共享车牌号和地址，与其他可疑案件建立了密集的连接。

![图片](img/17992110b8441851c0f1ce7b229db56d.png)

图 4. 对陈述 20000695 的视觉解释：左侧面板显示一个实体-陈述网络，突出陈述与维修店、位置和车牌号等关键实体之间的连接；右侧面板显示陈述-陈述网络，揭示该陈述如何通过共享实体与其他陈述聚类。下侧面板总结了支持欺诈标签的风险因素。[Streamlit 代码](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/6_streamlit.py)。图片由作者提供。

此代码将图保存为 pickle 文件：`temporal_graph_with_edge_attrs.gpickle. `

### 第 3 步：图准备与训练

> 表示学习将复杂、高维数据（如文本、图像或传感器读数）转换为简化、结构化的格式（通常称为嵌入），这些格式捕获了有意义的模式和关系。这些学习到的表示提高了模型性能、可解释性和在不同任务之间迁移学习的能力。

[我们训练一个神经网络](https://github.com/erikapat/graphs_neo4j/blob/feature/first_steps/temporal_graph/5_training.past_value.py)，将每个输入映射到ℝᵈ向量中，该向量编码了重要信息。在我们的管道中，**GraphSAGE**在索赔图上执行**表示学习**：它从节点的邻居（共享电话、商店、盘子等）中聚合信息，并将其与节点的自身属性混合，以生成**节点嵌入**。然后，这些嵌入被输入到一个小型分类器头部，以预测欺诈。

**3.1 时间切片**

在步骤 2 中创建的单个完整图中，我们提取了用于训练、验证和测试的**三个时间切片子图**。对于每个分割，我们选择一个截止日期，并仅保留（1）`claim_date ≤ cutoff`的**索赔节点**，以及（2）`timestamp ≤ cutoff`的**边**。这为该分割生成一个**时间一致的子图**：未来信息不会泄露到过去，与模型在生产环境中仅使用历史数据运行的方式相匹配。

**3.2 节点索引**

给切片图中每个节点分配一个整数索引`0…N-1`。这只是一个**ID 映射**（类似于标记化）。我们将使用这些索引来对齐张量中的特征、标签和边。

**3.3 构建节点特征**

为每个节点创建一个特征行：

+   **类型一热编码**（索赔、电话、电子邮件、……）。

+   在切片图中计算的**度统计**：在切片图内的归一化入度、出度和无向度。

+   **先前邻居的欺诈行为（仅限索赔）**：考虑当前索赔时间之前的邻居中，被标记为欺诈的**较老连接索赔**（直接索赔→索赔前辈）的比例。

    我们还为索赔设置了标签`y`（1/0），为实体设置了 0，并在`claim_mask`中标记索赔，以便仅对索赔计算损失/度量。

**3.4 构建 PyG 数据**

使用节点索引将边`(u→v)`转换为 2×E 整数张量`edge_index`，并**添加自环**，以便每个节点在每一层也保留其自身的特征。将所有内容打包到 PyG `Data(x, edge_index, y, claim_mask)`对象中。边是定向的，因此消息传递尊重时间（较早→较晚）。

**3.5 GraphSage**：

我们在 PyTorch Geometric 中实现了 GraphSAGE 架构，使用`SAGEConv`层。因此，我们运行了两个 GraphSAGE 卷积层（平均聚合），ReLU，dropout，然后是一个线性头部来预测欺诈与非欺诈。我们进行**全批处理**（无邻居采样）。损失函数被加权以处理类别不平衡，并且仅通过`claim_mask`在**索赔节点**上计算。在每个 epoch 后，我们在**验证集**上进行评估，并选择最大化 F1 的决策**阈值**；我们通过 val-F1（早期停止）保留最佳模型。

![图片](img/c8d84ce09d564b5dff1c02bf211cd952.png)

图 5. 图 SAGE 模型架构的 PyTorch 实现，用于在图数据上学习节点表示和预测。图片由作者提供。

**3.6 推理结果。**

使用**验证选择的**阈值在**测试集**上评估最佳模型。报告准确率、精确率、召回率、F1 分数和混淆矩阵。生成**提升表/图**（按分数十分位集中欺诈的情况），导出**t-SNE 图**以可视化嵌入结构。

![图片](img/1299bcb5fb22b365997f1c1ccad72c66.png)

**图 6**：模型结果。**左**：测试集上的十分位提升；**右**：声明嵌入的 t-SNE（欺诈与非欺诈）。图片由作者提供。

提升图评估模型对欺诈的排名效果：柱状图显示按分数十分位提升，而线条显示累积欺诈捕获。在最高 10-20%的索赔（十分位 1-2）中，欺诈率约为平均水平的 2-3 倍，这表明审查最高 20-30%的索赔将捕获大量欺诈。t-SNE 图显示了几个欺诈集中的聚类，表明模型学习到了有意义的关联模式，而与非欺诈点的重叠突出了剩余的模糊性和特征或模型调整的机会。

…

## **结论**

使用仅将旧声明与新声明（过去与未来）连接起来的图，而不“泄露”未来的欺诈信息，模型成功地将欺诈案例集中在得分最高的组中，在最高 10-20%的得分中实现了约 2-3 倍的检测率。这种设置足够可靠，可以部署。

作为测试，可以尝试一个图中连接是双向或无向（双向连接）的版本，并比较其与单向版本的**虚假改进**。如果双向版本得到显著更好的结果，那很可能是由于“时间泄露”，意味着未来的信息不恰当地影响了模型。这是证明为什么在真实用例中不应使用双向连接的一种方法。

为了避免使文章太长，我们将关于有泄露和无泄露的实验放在另一篇文章中讨论。在这篇文章中，我们专注于开发一个符合生产就绪的模型。

通过更丰富的特征、校准和模型的小幅调整，仍有改进的空间，但我们的重点是解释一种漏安全的时序图方法，该方法解决了数据泄露问题。

## **参考文献**

[1] Gomes-Gonçalves, E. (2025, January 23). Applications and Opportunities of Graphs in Insurance. *Medium*. Retrieved September 11, 2025, from [`medium.com/@erikapatg/applications-and-opportunities-of-graphs-in-insurance-0078564271ab`](https://medium.com/@erikapatg/applications-and-opportunities-of-graphs-in-insurance-0078564271ab)

[2] Kapoor, S., & Narayanan, A. (2023). *Leakage and the reproducibility crisis in machinelearning-based science. Patterns. 2023; 4 (9): 100804*. [Link](https://pdf.sciencedirectassets.com/776857/1-s2.0-S2666389922X0010X/1-s2.0-S2666389923001599/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEA4aCXVzLWVhc3QtMSJIMEYCIQCZBfrG4QrnhKJ%2Bn0n4lXjLmcjiQs7PoKeSb7C4lfpl%2FQIhANZjGQTIaURPRzMNpe9o%2BnX4nPcomYvwJCtgyhuZOKMoKrMFCHcQBRoMMDU5MDAzNTQ2ODY1Igz%2BHgNEXZXB%2FbZIZcEqkAXFhkKRGDxiSpHa%2BkphZvpX6iQUqo730MgusTWc8Pe2%2BEZfj9HNvp86gmObqFnw4y5JitTjMg0GL%2BXMikPNMwlWF%2BuDrjUUyVTvIqYbRY9HPtnEif4fAeTskU71SuTgA%2Bb91Rj6o0CE1sU8NdVULngUo9WcxV0alR0stU1Nf41zeb9tUiEReRg7q22wK0xeNQA8tPdsMuTtNNQ2W8pwFpFTtd09GNKqAQfNm0lmBQDeit9tkUkEZ%2FX6io4rR%2BeTL0x%2FhEtfH94dxIyFI%2BCeEZBNQz%2BD8uMxOO%2Bi6tXQWL2uErNnfl05lMIi54a415fJoQuOZFi9mhKqB7VQ4QaInUFl7i03wWBdkqPciWvP72RnEBc%2FpDTpXIAAzxohE%2FDLpRIAGhs%2FIX%2BieJIOMuwScibe8AcEwhxOF4EMws4nbkuykfj5JwMxb3tanWLnCrFd12GHfKg%2Bc6bUBxC4477c2VOIefQjZ%2Fxv6yc3%2Fm2OZetakhAtnD2rXxjU9qogCeFuXWAMYynSxgxDtpwuW5tGnm%2BkRtwqpajg8mMuvcC5lw5qpkXhKaYjP8HdwO%2B%2B4gvuLWXE3cQU2v00%2BlJ6kbkOrFmr8CxKJGs8744997GokVqYqNcN5DjFTwFWITBCxX2R4g%2FZajP9u5WXazbp6Z6x2D3EbZ97LfPMbXcdOyy%2B%2BVCmJcLC9gOC9hZeDWPF%2B0UXnfqHQRIv9m%2Bkb5XHyQKqbQSFm1%2FAQ2guzKTIkrnzCaRJeTAfigDjDwekO3RurM21NOdqRd760IQX6hxImstxaiOXeAVEUWmWrFBq6IDuZKty1jgedgTpvTqoFvLRaLSQuWAy1TZigbHHZ3Nl6LDbMxEWDAHUTkc8H2oqk3Tq%2BRI20zD32OvFBjqwAVxsgWoIpwk3NUogXRCkSYWJsdqUEWvfDgQmkreLfhtGY94xDxzdM5zXQKj2gth9GIECPvdGljT1FaZN5hJP%2Fd3F9VoP7%2F9MIeBtBUQRjNrL%2FirA1IFzInN2Tju3n1qWRZAxnggbGPN9bsAEKepY7%2BqIqPPEKJAjzuJySs5sTy1dqzH%2BQIN%2FYiwnDG4YEz4L7QFOHYdUeXtp%2FRcxMEGeLciOGmJR6ybSEr%2B9Umfl63U%2F&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250905T143853Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTY5LC2L7YA%2F20250905%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1862e25765da631946ce29cc907833c2d760e33689aed23cc1cbb5b96c5a4f9c&hash=a63208c15d523f43702ed5a09d4bf6d972a0cc05ddb9d6b0540f8e0915ee33dc&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=S2666389923001599&tid=spdf-66969b3e-1a80-4618-9b74-683b3dff2f8b&sid=a93309ab3c2b444bfd49f1a2f39c098134a8gxrqb&type=client&tsoh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&rh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&ua=061458570759515c54&rr=97a679129f1c03eb&cc=es).

[3] Guignard, F., Ginsbourger, D., Levy Häner, L., & Herrera, J. M. (2024). Some combinatorics of data leakage induced by clusters. *Stochastic Environmental Research and Risk Assessment*, *38*(7), 2815–2828.

[4] 黄某，等. (2024). UTG：面向时序图快照和事件模型统一视图. *arXiv 预印本 arXiv:2407.12269*. [`arxiv.org/abs/2407.12269`](https://arxiv.org/abs/2407.12269)

[5] Labonne, M. (2022). GraphSAGE：扩展图神经网络. *数据科学之路*. 从[`towardsdatascience.com/introduction-to-graphsage-in-python-a9e7f9ecf9d7/`](https://towardsdatascience.com/introduction-to-graphsage-in-python-a9e7f9ecf9d7/)获取。

[6] 图 SAGE 简介. (2025). Weights & Biases. 从[`wandb.ai/graph-neural-networks/GraphSAGE/reports/An-Introduction-to-GraphSAGE–Vmlldzo1MTEwNzQ1`](https://wandb.ai/graph-neural-networks/GraphSAGE/reports/An-Introduction-to-GraphSAGE--Vmlldzo1MTEwNzQ1)获取。

# 从零开始创建 SMOTE 过采样

> 原文：[`towardsdatascience.com/creating-smote-oversampling-from-scratch-64af1712a3be/`](https://towardsdatascience.com/creating-smote-oversampling-from-scratch-64af1712a3be/)

合成少数类过采样技术（SMOTE）通常用于处理数据集中的类别不平衡问题。假设有两个类别，其中一个类别有远多于另一个类别的样本（多数类）比另一个（少数类）。在这种情况下，SMOTE 将在少数类中生成更多的合成样本，以便与多数类相匹配。

在现实世界中，我们不会在分类问题中拥有平衡的数据集。以预测患者是否患有镰状细胞性贫血的分类器为例。如果患者的血红蛋白水平异常（6-11 g/dL），那么这是镰状细胞性贫血的强烈预测因素。如果患者的血红蛋白水平正常（12 mg/dL），那么这个预测因素本身并不表明患者是否患有镰状细胞性贫血。

然而，在美国大约有 10 万名患者被诊断出患有镰状细胞性贫血。目前美国有 3.349 亿公民。如果我们有一个包含每个美国公民的数据集，并标记或未标记患者是否患有镰状细胞性贫血，那么患有该疾病的人只占 0.02%。我们有一个主要类别不平衡。我们的模型无法提取出有意义的特点来预测这种异常。

此外，如果我们的模型预测所有患者都没有镰状细胞性贫血，那么它的准确率将达到 99.98%。这并不能帮助我们解决健康问题，而且也限制了使用准确率作为评估模型性能的唯一指标。[在医疗数据集中，类别不平衡很常见，从而阻碍了罕见疾病/事件的检测。](https://pmc.ncbi.nlm.nih.gov/articles/PMC5987310/) 大多数疾病分类方法隐含地假设类别发生的频率相等，这有助于最大化整体分类准确率。

[虽然可以使用其他指标代替准确率（精确率和召回率）](https://www.linkedin.com/pulse/balancing-act-navigating-precision-vs-recall-ai-emily-ux75c/)，我们还想对少数类（患有罕见疾病的患者）进行过采样，以确保每个类别标签的数据点数量相似。

## SMOTE 是如何工作的？

然而，SMOTE 不会创建少数样本的精确副本。相反，它结合了两种算法：K 最近邻和线性插值。

下面是利用这两种算法的 SMOTE 的**伪代码**。

+   SMOTE 在少数样本中随机选择一个点

+   SMOTE 在该少数样本中找到其 k 个最近邻

+   SMOTE 随机选择这些 k 个邻居之一

+   SMOTE 通过随机选择的点和其邻居创建一个线性方程，然后沿着该线性方程生成一个合成样本。

下面是 SMOTE 工作原理的图表。

![作者自定义生成的视觉图像](img/ed29a3ab2e7382421bf25665f9904ea7.png)

作者自定义生成的视觉图像。

你可以看到，合成样本（橙色）是在感兴趣的数据点和选定的邻居之间形成的线性方程上的一个随机点。合成样本坐标有以下约束。

+   Xs 是 Xk 和 Xi 之间的一个值。在这个例子中，我们假设 Xs、Xk 和 Xi 都是整数。

+   Ys 是将 Xs 插入线性方程后的结果。然后 Xs 乘以邻居斜率，再加上 y 截距（在这种情况下，Yk）。

下面是如何使用 [Imbalance-Learn](https://imbalanced-learn.org/stable/) SMOTE 包的示例。

> [**使用 Python 的 SMOTE 进行不平衡分类**](https://www.analyticsvidhya.com/blog/2020/10/overcoming-class-imbalance-using-smote-techniques/)

但我们如何从头开始创建它呢？

## 初始化数据集

因此，让我们有一个包含 110 个点的数据集。每个点都有一个 x 和 y 坐标，这两个值都在 -5 到 5 的范围内。这些 x 和 y 坐标也是整数。

+   其中 10 个点的标签为 1

+   其中 100 个点的标签为 0

下面是此数据集的图表。

![Matplotlib 数据集的图表。作者自定义生成的图表。](img/0e36a6116f0ca162afbb1a9ac52b7669.png)

Matplotlib 数据集的图表。作者自定义生成的图表。

如果我们在这个数据集上训练一个分类器，它如果只预测所有点的标签为 0，将得到 90% 的准确率。因此，我们想要使用 SMOTE 对少数类（标签 1）进行过采样，以便它可以区分两个标签。

我们想要添加 90 个带有标签 1 的合成数据点，以便我们有一个包含 200 个点的数据集，每个标签有 100 个点。

## 步骤

### 在少数样本中随机选择一个点

假设代码的第一步是过滤掉少数样本中的所有数据点，并随机选择其中一个点。

下面是少数样本中所有数据点的图表。

![Matplotlib 数据集的图表。作者自定义生成的图表。](img/07269f7bee9be6f45aacb21af55ebabd.png)

Matplotlib 数据集的图表。作者自定义生成的图表。

因此，让我们取 (1, 4) 的数据点。

![Matplotlib 数据集的图表。作者自定义生成的图表。](img/fc093db69f0955ce61ae0a850de465be.png)

Matplotlib 数据集的图表。作者自定义生成的图表。

### 寻找 k 个最近邻

下一步是找到该数据点的 k 个最近邻。目前，我们假设 k=1。

因此，数据点 (1,4) 的最近邻是 (-2,2)。

![Matplotlib 数据集的图表。作者自定义生成的图表。](img/5311d61e2960e875e87e423c76f005bc.png)

Matplotlib 数据集的图表。作者自定义生成的图表。

### 随机选择那些 k 个邻居中的一个

然后，我们随机选择该数据点的 k 个邻居中的一个。由于我们假设 k=1，我们将使用 (-2, 2)。

### 创建线性方程

从该数据点和其邻居，我们将创建一个线性方程。这个方程是 y = (2x + 10) / 3

![Matplotlib 数据集的图表。作者自定义生成的图表。](img/4e2fa2eb1235aec33de127f382c517cc.png)

数据集的 Matplotlib 图表。由作者自定义生成的图表。

### 生成合成样本

根据给定的方程，随机生成该线上的合成样本。所以如果我们设置 x = 0，y = (2 *0 + 10)/3 = 10/3 = 3.33

我们合成的样本是点（0，3.33）。

![数据集的 Matplotlib 图表。由作者自定义生成的图表。](img/4b629e0fd90ab3f2d887ae920711f8a5.png)

数据集的 Matplotlib 图表。由作者自定义生成的图表。

### 重复

因此，我们创建了 1 个合成数据点。这还剩下 89 个更多的合成数据点。

因此，我们使用不同的数据点重复上述步骤，直到我们得到 90 个合成数据点。

## SMOTE 的代码

```py
import numpy as np
from sklearn.neighbors import NearestNeighbors

def custom_smote(samples, n, k):
    '''
    n = total number of synthetic samples to generate
    k = number of heighbos
    '''
    synthetic_shape = (n, samples.shape[1]) 
    synthetic = np.empty(shape=synthetic_shape)
    synthetic_index = 0

    nbrs = NearestNeighbors(n_neighbors=k,metric='euclidean',algorithm='kd_tree').fit(samples)

    for synthetic_index in range(synthetic.shape[0]):
       max_samples_index = samples.shape[0]
       A_idx = np.random.randint(low=0, high=max_samples_index)
       A_point = samples[A_idx]
       distances,knn_indices = nbrs.kneighbors(X=[A_point], n_neighbors=(k+1))
       neighbor_array = knn_indices[0]

       if A_idx in neighbor_array:
          condition = np.where(neighbor_array == A_idx)
          neighbor_array = np.delete(neighbor_array, condition)

       len_neighbor_array = len(neighbor_array)
       if len_neighbor_array > 0:
          B_idx = np.random.randint(low=0, high=len_neighbor_array)
       else:
          B_idx = 0

       B_point = samples[neighbor_array[B_idx]]
       m = (B_point[1] - A_point[1])/(B_point[0] - A_point[0])

       high_point = A_point[0] if A_point[0] > B_point[0] else B_point[0]
       low_point = A_point[0] if A_point[0] < B_point[0] else B_point[0]

       if m == m:
          random_x = np.random.uniform(low=low_point, high=high_point)
          random_y = m * (random_x-A_point[0]) + A_point[1]
       else:
          random_x = B_point[0]
          random_y = B_point[1]

       synthetic[synthetic_index] = (random_x, random_y)

    return synthetic 
```

## SMOTE 的结果

使用 1 个邻居，我们将生成以下带有标签=1 的 90 个合成数据点

![数据集的 Matplotlib 图表。由作者自定义生成的图表。](img/b0578cd2d7458c0696b4449c45d8d692.png)

数据集的 Matplotlib 图表。由作者自定义生成的图表。

如果我们将我们的合成样本附加到现有数据集中，我们可以看到它们如何映射。

![数据集的 Matplotlib 图表。由作者自定义生成的图表。](img/fd4abcadc880130b42268988569f2f89.png)

数据集的 Matplotlib 图表。由作者自定义生成的图表。

我们没有看到合成样本之间有很多差异。它们对于标签=1 很好地分组，而不是像标签=0 那样分散。这是因为我们选择从少数类数据点的“一个最近邻”中随机选择一个邻居。

如果我们增加随机选择的邻居数量，我们将为少数类提供更多样化的样本。

![](img/61a7acece92d53e021c58f070bd65a7a.png)![](img/af0fa63b29d619b780c5f408c0b4de05.png)![](img/2859722bab7d7e56b14a387c9c4a07d3.png)![随着我们增加要选择的邻居数量，样本分布得更加广泛。数据集的 Matplotlib 图表。由作者自定义生成的图表。](img/3cd2a0bf5cca599dca801d94e22baa47.png)

你可以看到，当我们增加要选择的邻居数量时，样本分布得更加广泛。数据集的 Matplotlib 图表。由作者自定义生成的图表。

## 生成图表的代码

```py
from collections import Counter
import matplotlib.pyplot as plt
import os    

def summarize_data(regular, graph_title, filename):
    plt.figure(figsize=(8,6))
    x = regular[:, 0]
    y = regular[:, 1]
    plt.scatter(x, y)
    plt.title(graph_title)
    plt.savefig(filename)

def summarize_data_with_legend(X, y, graph_title, filename):
    plt.figure(figsize=(8,6))
    counter = Counter(y)
    for label, _ in counter.items():
        row_idx = np.where(y == label)[0]
        plt.scatter(X[row_idx, 0], X[row_idx, 1], label=str(label))
    plt.legend()
    plt.title(graph_title)
    plt.savefig(filename)

k=9

small_sample_label_1 = np.random.randint(-5, 5, (10,2))
small_sample_label_0 = np.random.randint(-5, 5, (100,2))
print(small_sample_label_1 )
file_path = os.path.dirname(__file__)
image_output_path = file_path + '/small_sample.jpg'

# Create plot of 10 label 1 points and 100 label 0 points
summarize_data(small_sample_label_1, "Original Sample With Label=1", file_path + '/original_sample.jpg')
X = np.concatenate((small_sample_label_0, small_sample_label_1), axis=0)
y = np.concatenate((np.zeros(100), np.ones(10)), axis=0)
summarize_data_with_legend(X, y,  "Original Sample with labels", file_path + '/original_with_labels.jpg')

# Create 90 synthetic samples wit label 1, and plot them in single plot
small_synthetic_label_1 = custom_smote(small_sample_label_1 , 90, k)
summarize_data(small_synthetic_label_1 , "SMOTE of Sample With Label=1, k=%i"%k, file_path + '/synthetic_sample_k_%i.jpg'%k)

# Join 90 synthetic samples with rest of dataset, and plot
X = np.concatenate((small_sample_label_0, small_sample_label_1, small_synthetic_label_1), axis=0)
y = np.concatenate((np.zeros(100), np.ones(100)), axis=0)
summarize_data_with_legend(X, y,  "SMOTE Sample with labels, k=%i"%k, file_path + '/SMOTE_with_labels_k_%i.jpg'%k)
```

## SMOTE 的变体

现在你有了从头实现 SMOTE 的代码，还有其他方法可以调整它以改善采样的方差（除了增加要选择的邻居数量）。

+   使用径向核而不是线性方程来生成合成样本。

+   使用除了 K 最近邻算法之外的其他算法来选择比较点。例如，使用 K-Means 算法来找到少数样本的聚类。从那里，获取这些聚类的质心。使用质心作为与选定的数据点（而不是 K 最近邻算法中的实际数据点）进行比较的点。

+   选择多个 K 最近邻。而不是使用线性方程来生成合成样本，使用所选数据点的线性超平面及其超过一个的 K 最近邻。

下面的这篇文章来自 KDNuggets，介绍了 7 种其他的 SMOTE 变体。它们都在 Python 库中，但描述应该能给您提供一个思路，了解如何在自己的自定义 SMOTE 中融入这些变化。

[`www.kdnuggets.com/2023/01/7-smote-variations-oversampling.html`](https://www.kdnuggets.com/2023/01/7-smote-variations-oversampling.html)

## 结论

我们讨论了过采样技术及其在现实世界中的应用。我们描述了如何从头开始实现这一技术，并提供了 Python 逻辑。然后我们讨论了其他形式的 SMOTE，这些形式可以增加少数样本的方差，以及如何实现这些变化。

* * *

感谢阅读！如果您想阅读更多我的作品，请查看我的[目录](https://hd2zm.medium.com/table-of-contents-read-this-first-a124146f566c)。

如果您不是 Medium 付费会员，但只想订阅阅读像这样的教程和文章，请[点击此处](https://hd2zm.medium.com/membership)加入会员。通过此链接加入会员意味着我因向您推荐 Medium 而获得报酬。

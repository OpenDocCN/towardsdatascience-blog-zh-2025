# 零膨胀数据：回归模型比较

> 原文：[`towardsdatascience.com/zero-inflated-data-comparison-of-regression-models/`](https://towardsdatascience.com/zero-inflated-data-comparison-of-regression-models/)

<mdspan datatext="el1757014699606" class="mdspan-comment">我最近在处理一个回归问题。我知道**我想要设计的预测模型的目标**是可计数的（即 0、1、2、……）。因此，我立即想到了选择一个具有相关**离散分布**的广义线性模型（GLM），如**泊松分布**或**负二项分布**。但一切并没有像预期的那样顺利。我误将仓促行事当作了迅速行动。

## 零膨胀数据

首先，让我们看看适合这篇文章的数据集。我选择了[**NextGen National Household Travel Survey**](http://nhts.ornl.gov) [1]的结果。感兴趣的变量名为“BIKETRANSIT”，表示“过去 30 天内骑自行车使用的天数”，因此对于每日用户来说，这是一个介于 0 到 30 之间的整数。以下是该变量的直方图。

![](img/978e97fb2a55c96d4aef144fe50ffaaa.png)

自行车日数的直方图

我们可以清楚地看到**计数数据是零膨胀的**。大多数受访者在过去 30 天内没有骑过一天自行车。我还注意到一些有趣的模式：与相邻数字相比，报告在恰好 5、10、15、20、25 或 30 天使用自行车的人数似乎更多。这可能是因为受访者在不确定确切计数时倾向于选择整数。无论原因如何，在这篇文章中，我们将主要关注**通过比较针对零膨胀计数数据设计的模型**来探讨零膨胀问题。

几个调查领域已被选为**独立变量**来解释自行车日数（例如，年龄、性别、工人类别、教育水平、家庭规模和地区特征）。我故意排除了计数其他活动（如使用出租车或共享单车）天数的特点，因为其中一些与感兴趣的结果高度相关。我希望模型保持现实：基于同一时期的出租车、汽车或公共交通使用情况来预测 30 天内的自行车使用情况不会提供有意义的见解。

## 泊松回归限制

在介绍零膨胀模型之前，我想**说明泊松回归的局限性**，这是我最初考虑用于此数据集的。我尚未在该部分查看负二项分布。泊松回归假设在独立变量 X 和参数β的条件下，因变量 Y 遵循泊松分布。

![](img/15ac241d6dc510e5d559e2af71350207.png)

泊松回归分布模型

那么，让我们来看看 Y∣X,β的一些经验分布。由于我包含了众多特征，很难找到具有完全相同 X 值的大量观测值。为了解决这个问题，我使用了一个聚类算法——[scikit-learn 中的 AgglomerativeClustering](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html) [2]——将具有相似特征轮廓的观测值分组。

首先，我预处理了数据，使其可以输入回归模型和聚类算法。我不想花太多时间解释所有预处理步骤，因为这篇文章并不专注于这一点。**完整的预处理代码可在** [**repo**](https://github.com/kapytaine/zero_inflated_data) **[8]** 上找到。简而言之，我使用独热编码对分类特征进行了编码。我还对其他特征应用了几个预处理步骤：填补缺失值、剪裁异常值，并在适当的地方应用转换函数。最后，我在转换后的数据集上执行了聚类。

```py
from sklearn.cluster import AgglomerativeClustering
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline(
    [
        ("scaler", StandardScaler()), # I normalized the data as some numerical features, like age, have range of value greater than the one hot encoded features and I know clustering works based on some distance
        ("cluster", AgglomerativeClustering(n_clusters=100)) # I chose 100 clusters to have many observations in the biggest groups
    ]
)
cluster_id = pipe.fit_predict(X_train_preprocessed) # here X_train_preprocessed is numerical dataframe, after encoding the categorical features
```

然后，我使用每个聚类组的观测随机变量的均值作为无偏估计量来估计泊松分布的参数Ⲗ。

![](img/bc5d58a130ea0607ecb9c86c9d08c4eb.png)

Ⲗ估计量

我随后绘制了**经验直方图**以及拟合泊松分布的**概率质量函数**，针对几个观测值组。为了评估拟合质量，我计算了**交叉熵**和**熵**，并注意到根据[吉布斯不等式](https://en.wikipedia.org/wiki/Gibbs%27_inequality) [3]，熵是交叉熵的下界。一个好的模型应该产生接近熵的交叉熵值（尽管略大）。

在这次分析中，我专注于三个最大的组，因为在大样本量下参数估计更可靠。在这里尤其重要，因为数据由于零膨胀而偏斜，需要收集大量观测值。在这些组中，有两个包含自行车用户，而一个组（228 个受访者）报告根本不使用自行车。对于这个最后组，没有拟合泊松分布，因为泊松参数必须严格大于零。最后，我在图表中使用了垂直对数刻度来考虑零膨胀。

![](img/eab8f50dc9e0c8d6f8cf4e27be351eb3.png)

我发现通过观察熵和交叉熵来评估拟合分布的质量比较困难。然而**我可以看到直方图和概率质量函数差异很大**。**这就是我随后考虑使用零膨胀泊松（ZIP）分布的原因**。

## 零膨胀数据适应模型

为零膨胀数据设计的模型旨在**捕捉零的高概率和其他事件的相对低概率**。我探索了**两种主要模型家族**：

+   “**零膨胀模型** [...] 使用双组分混合模型来模拟零值。 [...] 变量为零的概率由主要分布和混合权重共同决定”。 “零膨胀模型只能增加 P(x = 0) 的概率” [5]。为了表示，我使用了以下设置（与维基百科和其他来源略有不同）。令 X1 为一个服从伯努利分布的隐藏变量。在我的表示法中，成功的概率是 p（而维基百科使用 1-π）。令 X2 为另一个服从允许零值有非零概率的分布的隐藏变量。对于我的用例，我假设 X2 是离散的。观察变量定义为 X=X1*X2，从而得到以下概率质量函数：

    ![图片](img/18f297ab3df162ca27af92010d71a632.png)我们可以注意到 X1 和 X2 部分是隐藏的。当 X=0 时，我们无法知道 X1 和 X2 的值，但一旦 X>0，X1 和 X2 这两个变量都是已知的。

+   [**门槛模型**](https://en.wikipedia.org/wiki/Hurdle_model) 使用两部分来模拟可观察的“随机变量 [...]”，其中第一部分是达到值 0 的概率，第二部分模拟非零值的概率 [5]。与零膨胀模型不同，第二部分必须遵循一个零概率恰好为零的分布。使用与之前相同的表示法，X1 模拟观察值是零还是非零（通常通过伯努利分布）。X2 遵循一个不对零分配概率质量的分布。因此，概率质量函数为：

    ![图片](img/d617ea5b2f7526ca03c7a2646e25f22d.png)

## 零膨胀泊松模型

让我们看看[**零膨胀泊松模型**](https://en.wikipedia.org/wiki/Zero-inflated_model#Zero-inflated_Poisson) [4]。ZIP 概率质量函数为：

![图片](img/b006468b67a7427dffd009376cdb3326.png)

ZIP 概率质量函数

现在可以通过添加 ZIP-拟合的概率质量函数来扩展先前的直方图和泊松拟合的概率质量函数。为此，需要两个参数 p 和 λ 的估计量。我使用了矩估计法来推导这些估计量：前两个矩提供了一组包含两个未知数的两个方程，然后可以求解。

![图片](img/9ea1c0faa929bb63f9d162bf72763e20.png)

矩估计法获取 ZIP 参数估计量

因此，参数估计量为：

![图片](img/95680e0ab71ccdf4d6eef34b9c277c1b.png)

ZIP 参数估计量

最后，我绘制了相同的两个图，其中包含了拟合的 ZIP 分布概率质量函数以及交叉熵度量。

![图片](img/a426e0a3b5dd141e7ec739aa981d252d.png)

**视觉检查**和**交叉熵**值都表明 ZIP 模型比泊松模型更好地拟合观察数据。这为选择 ZIP 回归而不是泊松回归提供了一个客观和可量化的理由。

## 模型比较

现在我们来比较几个模型。我将数据分为训练集和测试集，但并不立即清楚哪种评估指标最为合适。例如，我应该依赖泊松偏差，尽管数据是零膨胀的？还是均方误差，它对异常值有严重的惩罚？最终，我选择使用多个指标来更好地捕捉模型性能：**平均绝对误差**、**泊松偏差**和**相关系数**。我评估的模型包括：

+   一个预测训练集平均值的**朴素模型**，

+   **线性回归** (*lr*),

+   **泊松回归** (*pr*),

+   **零膨胀泊松回归** (*zip*),

+   一个**链式逻辑-泊松回归**（障碍模型，*lr_pr*），

+   一个**链式逻辑-零截断泊松回归**（障碍模型，*lr_tpr*）。

## ZIP 模型

让我们看看 ZIP 回归的实现。首先，观察数据的**负对数似然**，记为 y，是：

![](img/ce661b4ea561ec3f47081de94dfd83f0.png)

负对数似然

观察数据的边缘似然，P(Y=y)，可以在不使用联合分布的积分公式 P(Y=y, X1=x1)的情况下解析地表示。因此，**可以直接优化，无需使用**[**期望最大化算法**](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm) [6]。两个分布参数 p 和Ⲗ是特征 X 和模型参数β的函数，这些参数将被学习。我选择将**p 定义为 X 和β的点积的 sigmoid 函数**，**Ⲗ定义为 X 和β的点积的指数函数**。为了使模型更加灵活，我使用不同的参数集β：一个用于 p，另一个用于λ。

![](img/ef5135e395ec850faaca01f4b0cb340c.png)

ZIP 参数表达式

此外，我还为参数β添加了一个**先验，以正则化模型**，这对于由于零膨胀而观察较少的泊松模型特别有用。我假设了一个正态先验，因此在损失函数中添加了 L2 正则化项。我假设了两个不同的先验，一个用于伯努利模型的β，一个用于泊松模型的β，因此有两个超参数α，分别记为模型中的 alpha_b 和 alpha_p 属性。我通过超参数优化来优化这些值。

我创建了一个从 scikit-learn 的`BaseEstimator`继承的类。损失函数的 Python 实现如下（在类内部实现，因此有`self`参数）：

```py
def _loss(self, beta: np.ndarray, X: np.ndarray, y: np.ndarray) -> float:
    n_feat = X.shape[1]

    # split beta into two parts: one for bernoulli p and one for poisson lambda
    beta_p = beta[:n_feat]
    beta_lam = beta[n_feat:]

    # get bernoulli p and poisson lambda
    p = sigmoid.val(beta_p, X)
    lam = exp.val(beta_lam, X)

    # initialize negative log likelihood
    out = 0

    # y == 0
    y_e0_mask = np.where(y == 0)[0]
    out += np.sum(-np.log((1 - p) + p * np.exp(-lam))[y_e0_mask])

    # y > 0
    y_gt0_mask = np.where(y > 0)[0]
    out += np.sum(-np.log(p)[y_gt0_mask])
    out += np.sum(-xlogy(y, lam)[y_gt0_mask])
    out += np.sum(lam[y_gt0_mask])

    # prior
    mask_b = np.ones_like(beta)
    mask_b[n_feat:] = 0
    mask_p = np.ones_like(beta)
    mask_p[:n_feat] = 0
    if self.fit_intercept:
        mask_b[n_feat - 1] = 0
        mask_p[2 * n_feat - 1] = 0
    out += 0.5 * self.alpha_b * np.sum((beta * mask_b) ** 2)
    out += 0.5 * self.alpha_p * np.sum((beta * mask_p) ** 2)

    return out
```

为了优化损失目标函数，我还计算了**损失的雅可比矩阵**。

![](img/725a9bce5a8e74f34c11b937706de0a7.png)

负对数似然的雅可比矩阵

Python 实现如下：

```py
def _jac(self, beta: np.ndarray, X: np.ndarray, y: np.ndarray) -> np.ndarray:
    n_feat = X.shape[1]

    # split beta into two parts: one for bernoulli p and one for poisson lambda
    beta_p = beta[:n_feat]
    beta_lam = beta[n_feat:]

    # get bernoulli p and poisson lambda
    p = sigmoid.val(beta_p, X)
    lam = exp.val(beta_lam, X)

    # y == 0 & beta_p
    jac_e0_p = np.expand_dims(
        np.where(
            y == 0,
            (1 - np.exp(-lam)) / ((1 - p) + p * np.exp(-lam)),
            np.zeros_like(y),
        ),
        axis=1,
    ) * sigmoid.jac(beta_p, X)
    # y == 0 & beta_lam
    jac_e0_lam = np.expand_dims(
        np.where(
            y == 0,
            p * np.exp(-lam) / ((1 - p) + p * np.exp(-lam)),
            np.zeros_like(y),
        ),
        axis=1,
    ) * exp.jac(beta_lam, X)

    # y > 0 & beta_p
    jac_gt0_p = np.expand_dims(
        np.where(y > 0, -1 / p, np.zeros_like(y)), axis=1
    ) * sigmoid.jac(beta_p, X)
    # y > 0 & beta_lam
    jac_gt0_lam = np.expand_dims(
        np.where(y > 0, 1 - y / lam, np.zeros_like(y)), axis=1
    ) * exp.jac(beta_lam, X)

    # initialize jac
    out = np.concatenate((jac_e0_p + jac_gt0_p, jac_e0_lam + jac_gt0_lam), axis=1)

    # jac for prior
    mask_b = np.ones_like(beta)
    mask_b[n_feat:] = 0
    mask_p = np.ones_like(beta)
    mask_p[:n_feat] = 0
    if self.fit_intercept:
        mask_b[n_feat - 1] = 0
        mask_p[2 * n_feat - 1] = 0

    return (
        np.sum(out, axis=0)
        + self.alpha_b * beta * mask_b
        + self.alpha_p * beta * mask_p
    )
```

不幸的是，损失函数不是凸的，局部最小值不一定是全局最小值。我选择了 scipy 中 Broyden-Fletcher-Goldfarb-Shanno 的轻量级实现，因为它比我已经测试过的梯度下降方法更快。

```py
res = minimize(
    self._loss,
    np.zeros(2 * n_feat),
    args=(X, y),
    jac=self._jac,
    method="L-BFGS-B",
)
```

整个班级的代码都包含在这个[文件](https://github.com/kapytaine/zero_inflated_data/blob/master/src/model/MixtureBernPoisRegression.py)中，来自共享仓库。

在执行超参数优化调整阶段以获得最佳正则化超参数之后，我终于在测试集上计算了所选指标。除了指标外，还显示了拟合时间。

![图片](img/fddc30f74b309234e5f6ed27b0453f17.png)

基准结果

**零膨胀模型**（ZIP 和门槛）**比朴素模型、线性回归和标准泊松回归实现了更好的指标**。鉴于观察到的 Y 的经验直方图比泊松分布更接近 ZIP 分布，我最初预计性能差距会更大。然而，这种改进是以更长的拟合时间为代价的，尤其是对于 ZIP 模型。对于这个用例，门槛模型似乎提供了最佳折衷方案，在保持训练时间相对较低的同时提供强大的性能。

相对较小的改进可能的一个原因是数据并不严格遵循 ZIP 分布。为了调查这一点，我使用相同的模型在特别为遵循 ZIP 分布而生成的人工数据集上运行了另一个基准测试。这个数据集被设计成具有与原始数据集大致相同的观测数和特征数，但目标变量按照设计遵循 ZIP 分布。

![图片](img/1cf2a48de704142083c781b5886e9109.png)

假设 ZIP 分布数据集的基准结果

**当目标真正遵循 ZIP 分布时，ZIP 模型优于所有其他考虑的模型**。还值得注意的是，在这个合成设置中，特征不再是稀疏的（按设计），这可能有助于解释拟合时间的减少。

## 结论

**在选择统计模型之前**，仔细分析数据集比仅仅依赖对其特征的先验假设至关重要。通过检查经验分布（例如通过直方图）通常可以揭示指导选择适当概率模型的见解。

这对于零膨胀数据尤为重要，标准模型可能难以处理。一个具有零膨胀泊松（ZIP）分布的合成示例展示了**正确的模型如何与替代方案相比提供更好的拟合**，即使这些替代方案并不完全错误。

**对于零膨胀数据集**，如**零膨胀泊松**或**门槛模型**之类的模型特别有用。虽然两者都能有效地捕捉额外的零，但门槛模型有时在训练速度上提供可比的性能。

## 进一步阅读

在研究这个主题并撰写文章时，我发现这篇[medium 文章](https://medium.com/@amit25173/zero-inflated-models-for-rare-event-prediction-9dd2ec6af614) [7]非常推荐。

## 参考文献

+   美国联邦公路管理局. (2022). 2022 年 NextGen 全国家庭出行调查核心

    数据，美国交通部，华盛顿特区。在线可用：

    [`nhts.ornl.gov`](http://nhts.ornl.gov)

+   聚类分析 [`scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html`](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html)

+   吉布斯不等式 [`en.wikipedia.org/wiki/Gibbs%27_inequality`](https://en.wikipedia.org/wiki/Gibbs%27_inequality)

+   零膨胀泊松 [`en.wikipedia.org/wiki/Zero-inflated_model#Zero-inflated_Poisson`](https://en.wikipedia.org/wiki/Zero-inflated_model#Zero-inflated_Poisson)

+   阈值模型 [`en.wikipedia.org/wiki/Hurdle_model`](https://en.wikipedia.org/wiki/Hurdle_model)

+   期望最大化算法 [`en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm`](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm)

+   零膨胀模型用于罕见事件预测 [`medium.com/@amit25173/zero-inflated-models-for-rare-event-prediction-9dd2ec6af614`](https://medium.com/@amit25173/zero-inflated-models-for-rare-event-prediction-9dd2ec6af614)

    重新生成结果的代码库 [`github.com/kapytaine/zero_inflated_data`](https://github.com/kapytaine/zero_inflated_data)

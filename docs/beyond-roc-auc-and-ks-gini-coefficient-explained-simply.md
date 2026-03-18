# 吉尼系数：从洛伦兹曲线到模型评估

> 原文：[`towardsdatascience.com/beyond-roc-auc-and-ks-gini-coefficient-explained-simply/`](https://towardsdatascience.com/beyond-roc-auc-and-ks-gini-coefficient-explained-simply/)

<mdspan datatext="el1759191862708" class="mdspan-comment">我们已经在之前的博客中讨论了分类指标，如 ROC-AUC 和柯尔莫哥洛夫-斯米尔诺夫（KS）统计量。</mdspan>

在这篇博客中，我们将探讨另一个重要的分类指标，即**吉尼系数**。

* * *

## 我们为什么有多个分类指标？

每个分类指标都从不同的角度告诉我们模型的表现。我们知道 ROC-AUC 给我们提供了模型的总体排序能力，而 KS 统计量则显示了两组之间最大差距出现在哪里。

**当谈到吉尼系数时，它告诉我们我们的模型在将正值排序高于负值方面比随机猜测要好多少。**

* * *

首先，让我们看看吉尼系数是如何计算的。

对于这个，我们再次使用[德国信用数据集](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)。

让我们使用之前用来理解柯尔莫哥洛夫-斯米尔诺夫（KS）统计量计算的数据样本。

![图片](img/55f161aba2ee1868138b68a5f83ba6ff.png)

图片由作者提供

这个样本数据是通过在德国信用数据集上应用逻辑回归获得的。

由于模型输出概率，我们从这些概率中选取了 10 个点来展示吉尼系数的计算。

## 计算

### 第 1 步：按预测概率对数据进行排序。

样本数据已经按照预测概率升序排序。

### 第 2 步：计算累计人口和累计正值。

**累计人口：** 到目前为止考虑的记录的累计数量。

**累计人口（%）：** 到目前为止覆盖的总人口百分比。

**累计正值：** 到目前为止我们看到的实际正值（类别 2）的数量。

****累计正值（%）：** 到目前为止捕获的正值百分比。

![图片](img/3193aae319edd13bf5d2a8e27cce5bf3.png)

图片由作者提供

### 第 3 步：绘制 X 和 Y 值

X = 累计人口（%）

Y = 累计正值（%）

这里，我们将使用 Python 来绘制这些 X 和 Y 值。

**代码：**

```py
import matplotlib.pyplot as plt

# X and Y values
X = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
Y = [0, 0, 0, 0, 0, 0.25, 0.25, 0.50, 0.75, 1.0]

# Plot Lorenz curve
plt.plot(X, Y, marker='o', label="Lorenz Curve")

# Plot line of equality
plt.plot([0, 1], [0, 1], linestyle='--', color='red', label="Line of Equality")

# Labels
plt.title("Lorenz Curve (Sample Data)")
plt.xlabel("Cumulative Population (%)")
plt.ylabel("Cumulative Positives (%)")
plt.grid(True, linestyle='--', alpha=0.5)
plt.legend()
plt.show() 
```

**图表：**

![图片](img/87c5eb9901ab268889bd25a76c128139.png)

图片由作者提供

当我们绘制累计人口（%）和累计正值（%）时得到的曲线称为**洛伦兹曲线**。

### 第 4 步：计算洛伦兹曲线下的面积。

当我们讨论 ROC-AUC 时，我们使用梯形公式来找到曲线下的面积。

两个点之间的每个区域都被视为梯形，其面积被计算，然后将所有面积相加得到最终值。

这里应用相同的方法来计算洛伦兹曲线下的面积。

**洛伦兹曲线下的面积**

梯形面积：

$$

\text{面积} = \frac{1}{2} \times (y_1 + y_2) \times (x_2 – x_1)

$$

\[

\begin{aligned}

A_1 &= \tfrac{1}{2}(0 + 0)(0.1 – 0.0) = 0.0000 \\[6pt]

A_2 &= \tfrac{1}{2}(0 + 0)(0.2 – 0.1) = 0.0000 \\[6pt]

A_3 &= \tfrac{1}{2}(0 + 0)(0.3 – 0.2) = 0.0000 \\[6pt]

A_4 &= \tfrac{1}{2}(0 + 0)(0.4 – 0.3) = 0.0000 \\[6pt]

A_5 &= \tfrac{1}{2}(0 + 0)(0.5 – 0.4) = 0.0000 \\[6pt]

A_6 &= \tfrac{1}{2}(0 + 0.25)(0.6 – 0.5) = 0.0125 \\[6pt]

A_7 &= \tfrac{1}{2}(0.25 + 0.25)(0.7 – 0.6) = 0.0250 \\[6pt]

A_8 &= \tfrac{1}{2}(0.25 + 0.50)(0.8 – 0.7) = 0.0375 \\[6pt]

A_9 &= \tfrac{1}{2}(0.50 + 0.75)(0.9 – 0.8) = 0.0625 \\[6pt]

A_{10} &= \tfrac{1}{2}(0.75 + 1.00)(1.0 – 0.9) = 0.0875 \\[8pt]

\text{洛伦兹曲线下的总面积：}

A &= 0.0000 + 0.0000 + 0.0000 + 0.0000 + 0.0000 \\[6pt]

&\quad + 0.0125 + 0.0250 + 0.0375 + 0.0625 + 0.0875 \\[6pt]

&= 0.2250

\end{aligned}

\]

我们计算了洛伦兹曲线下的面积，为 0.225。

在这里，我们在 X 轴上绘制了累积人口，在 Y 轴上绘制了累积正样本。

洛伦兹曲线帮助我们理解，当我们从最低到最高的预测概率移动时，正样本（类别 2）如何在人口中分布。

当曲线在一段时间内保持平坦，然后在接近结束时急剧上升时，这意味着大多数正样本都集中在得分最高的观察值中。这正是好的模型所期望做到的。

在我们的样本数据集中，有四个正样本（类别 2）和六个负样本（类别 1）。曲线的形状显示了模型如何有效地分离这两个类别。

将正样本推向更高的预测概率的模型将创建一个更弯曲的洛伦兹图，这表明更强的分离能力和更好的排序能力。

如果模型是完美的，所有正样本都只会出现在最高的预测概率区域。

在那种情况下，洛伦兹曲线会在大多数人口中保持为零，然后在接近结束时突然上升到 100%。这个陡峭的跳跃显示了完美的分离。

对于完美模型，曲线看起来是这样的。

![](img/a00104d716ccfc1c5857027790a68e8f.png)

图片由作者提供

完美模型的洛伦兹曲线下的面积。

\[

\begin{aligned}

\text{完美面积} &= \tfrac{1}{2} \times \text{底} \times \text{高} \\[6pt]

&= \tfrac{1}{2} \times (1.0 – 0.6) \times 1.0 \\[6pt]

&= \tfrac{1}{2} \times 0.4 \\[6pt]

&= 0.2

\end{aligned}

\]

我们还有另一种方法来计算完美模型的曲线下的面积。

\[

\text{设} \pi \text{为数据集中正样本的比例。}

\]

\[

\text{完美面积} = \frac{1}{2} (1 – (1 – \pi)) \times 1

\] \[

= \frac{1}{2} \pi

\] \[

= \frac{1}{2} \times 0.4

\] \[

= 0.2

\]

对于我们的数据集：

在这里，我们有 10 条记录中的 4 个正样本，所以：π = 4/10 = 0.4。

我们计算了样本数据集和具有相同正负样本数量的完美模型的洛伦兹曲线下的面积。

现在，如果我们不对数据集进行排序就通过它，正面结果会均匀分布。这意味着我们收集正面的速率与我们通过人口的速率相同。

这是随机模型，它总是给出曲线下的面积为 0.5。

![](img/a0e154a625154bf83dcdfbb46d17376b.png)

作者图片

### 第 5 步：计算基尼系数

\[

A_{\text{model}} = 0.225

\]

\[

A_{\text{random}} = 0.5

\] \[

A_{\text{perfect}} = 0.2

\] \[

\text{Gini} = \frac{A_{\text{random}} – A_{\text{model}}}{A_{\text{random}} – A_{\text{perfect}}}

\] \[

= \frac{0.5 – 0.225}{0.5 – 0.2}

\] \[

= \frac{0.275}{0.3}

\] \[

\approx 0.92

\]

我们得到了基尼系数为 0.92，这意味着几乎所有的正面结果都集中在排序列表的顶部。这表明模型在将正面结果与负面结果分开方面做得非常好，接近完美。

* * *

正如我们所见，基尼系数是如何计算的，让我们看看在计算过程中我们实际上做了什么。

我们考虑了一个由逻辑回归输出概率组成的 10 个点的样本。

我们按升序排列了概率。

Next, we calculated Cumulative Population (%) and Cumulative Positives (%) and then plotted them.

我们得到了一个称为洛伦兹曲线的曲线，并计算了其下的面积，为 0.225。

现在让我们理解 0.225 是什么意思？

我们的样本由 4 个正面（类别 2）和 6 个负面（类别 1）组成。

输出的概率是针对类别 2 的，这意味着概率越高，客户属于类别 2 的可能性就越大。

在我们的样本数据中，有四个正面（类别 2）和六个负面（类别 1）。洛伦兹曲线显示了随着我们从最低预测概率移动到最高预测概率，这些正面的分布情况。

在一个完美的模型中，所有四个正面结果只会出现在最高预测分数处，这意味着曲线在大多数人口中保持为零，然后在接近结束时跳到 100%。

在这种情况下，正面结果将完全包含在最后 40%的人口中，洛伦兹曲线下的面积将为 0.2。

对于我们的模型，洛伦兹曲线下的面积为 0.225，非常接近完美值。

这意味着模型在排序观测值方面做得很好：大多数正面结果都集中在最高分数中，模型有效地将两个类别分开。

洛伦兹曲线下的面积越小，模型捕捉正面的效率就越高，这表明预测性能更强。

接下来，我们计算了基尼系数，为 0.92。

\[

\text{Gini} = \frac{A_{\text{model}} – A_{\text{random}}}{A_{\text{perfect}} – A_{\text{random}}}

\]

分子告诉我们我们的模型比随机猜测好多少。

分母告诉我们相对于随机情况的最大可能改进。

这个比率将这两个结合起来，因此基尼系数总是在 0（随机）和 1（完美）之间。

基尼系数用于衡量模型在分离正负类别方面有多接近完美。

但我们可能会质疑为什么我们要计算基尼系数，以及为什么我们没有在 0.225 后停止。

0.225 是我们模型下洛伦兹曲线下的面积。如果不与 0.2（完美模型的面积）进行比较，它并不能告诉我们模型离完美有多近。

因此，我们计算基尼系数以标准化它，使其介于 0 和 1 之间，这使得比较模型变得容易。

* * *

在经济学中，对角线通常被称为平等线，洛伦兹曲线通常根据其与该线的接近程度来解释。

如果曲线非常接近，这意味着收入在人口中的分布更加均匀，这被认为是一件好事。

但在我们的情况下，我们不是在衡量收入不平等。我们试图了解模型在区分违约者和非违约者方面的表现如何。

在这里，我们实际上希望洛伦兹曲线远离平等线。

它弯曲得越远，模型在区分两个类别方面的能力就越强，其预测性能也会越强。

在经济学中，基尼系数为 0.92 通常被视为不良现象，因为它意味着存在大量不平等，大部分收入集中在少数人手中。

但在机器学习中，其含义完全不同。基尼系数为 0.92 被认为是非常好的，因为它表明模型在分离两个类别方面做得非常出色。

这意味着大多数违约者和非违约者都被正确地排名，模型实现了大约 92% 的最佳分离度。

* * *

银行也使用基尼系数与 ROC-AUC 和 KS 统计量一起评估信用风险模型。这些指标共同给出了模型性能的完整图景。

* * *

现在，让我们计算我们的样本数据的 ROC-AUC。

```py
import pandas as pd
from sklearn.metrics import roc_auc_score

# Sample data
data = {
    "Actual": [2, 2, 2, 1, 2, 1, 1, 1, 1, 1],
    "Pred_Prob_Class2": [0.92, 0.63, 0.51, 0.39, 0.29, 0.20, 0.13, 0.10, 0.05, 0.01]
}

df = pd.DataFrame(data)

# Convert Actual: class 2 -> 1 (positive), class 1 -> 0 (negative)
y_true = (df["Actual"] == 2).astype(int)
y_score = df["Pred_Prob_Class2"]

# Calculate ROC-AUC
roc_auc = roc_auc_score(y_true, y_score)
roc_auc
```

我们得到了 AUC = 0.9583

现在，Gini = (2 * AUC) – 1 = (2 * 0.9583) – 1 = 0.92

这是基尼系数与 ROC-AUC 之间的关系。

* * *

现在，让我们在完整的数据集上计算基尼系数。

**代码：**

```py
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score

# 1\. Load and preprocess dataset
file_path = "C:/german.data"
data = pd.read_csv(file_path, sep=" ", header=None)

# Rename columns
columns = [f"col_{i}" for i in range(1, 21)] + ["target"]
data.columns = columns

# Features and target
X = pd.get_dummies(data.drop(columns=["target"]), drop_first=True)
y = data["target"]

# Convert target: make it binary (1 = good, 0 = bad)
y = (y == 2).astype(int)

# Train-test split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# 2\. Train logistic regression
model = LogisticRegression(max_iter=10000)
model.fit(X_train, y_train)

# Predicted probabilities
y_pred_proba = model.predict_proba(X_test)[:, 1]

# 3\. Calculate ROC-AUC and Gini
auc = roc_auc_score(y_test, y_pred_proba)
gini = 2 * auc - 1

print("ROC-AUC:", auc)
print("Gini:", gini)

# 4\. Lorenz Curve Calculation
# Combine predictions and true labels into a dataframe
df_lorenz = pd.DataFrame({"y_true": y_test, "y_pred": y_pred_proba})

# Sort by predicted probability in ascending order (required for Lorenz curve)
df_lorenz = df_lorenz.sort_values("y_pred")

# Calculate cumulative population and cumulative positives
df_lorenz["cum_pop"] = np.arange(1, len(df_lorenz) + 1) / len(df_lorenz)
df_lorenz["cum_positives"] = df_lorenz["y_true"].cumsum() / df_lorenz["y_true"].sum()

# Add (0,0) starting point
cum_pop = np.insert(df_lorenz["cum_pop"].values, 0, 0)
cum_positives = np.insert(df_lorenz["cum_positives"].values, 0, 0)

# 5\. Plot Lorenz Curve
plt.figure(figsize=(7,7))
plt.plot(cum_pop, cum_positives, label="Model Lorenz Curve", color='blue', marker='o')

# Random model (diagonal)
plt.plot([0, 1], [0, 1], linestyle='--', color='gray', label="Random Model")

plt.title("Lorenz Curve - Logistic Regression Model")
plt.xlabel("Cumulative Population (%)")
plt.ylabel("Cumulative Positives (%)")
plt.legend()
plt.grid(True, linestyle='--', alpha=0.6)
plt.show()
```

![图片](img/b104e89588ccc29267143e60ee3b938a.png)

图片由作者提供

我们得到了基尼系数 Gini = 0.60，这意味着模型在将正样本排在负样本之上方面做得很好，并且达到了一个完美模型大约 60% 的分离度。

### 解释（机器学习/信用评分背景）：

基尼系数低于 0.5 通常表明模型较弱，并不比随机猜测好多少。

介于 0.5 和 0.6 之间的分数表明模型是可以接受的，并且具有一定的预测能力。

介于 0.6 和 0.7 之间的范围被认为是好的，表明模型可以可靠地区分这两个类别。

基尼系数高于 0.8 被认为是优秀的，在现实场景中很少实现。

* * *

### 数据集

本博客使用的数据集是[德国信用数据集](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)，该数据集在 UCI 机器学习库上公开可用。它是在[Creative Commons Attribution 4.0 国际（CC BY 4.0）许可](https://creativecommons.org/licenses/by/4.0/legalcode)下提供的。这意味着它可以自由使用和分享，只要适当注明出处。

* * *

希望您觉得这篇文章有用且清晰。

如果您还没有阅读我之前关于[ROC-AUC](https://towardsdatascience.com/roc-auc-explained-a-beginners-guide-to-evaluating-classification-models/)和[Kolmogorov Smirnov 统计量](https://towardsdatascience.com/kolmogorov-smirnov-statistic-explained-measuring-model-power-in-credit-risk-modeling/)的博客，您可以在下面查看。

如果您想阅读我的更多作品，您也可以在[Medium](https://medium.com/@dasarinikhil076)和[LinkedIn](https://www.linkedin.com/in/nikhildasari-dataanalyst/)上找到。

**注意：** 我在之前对这个部分的解释中犯了一个小错误。我已经纠正了它，所以概念更加清晰和准确。

感谢您的阅读！

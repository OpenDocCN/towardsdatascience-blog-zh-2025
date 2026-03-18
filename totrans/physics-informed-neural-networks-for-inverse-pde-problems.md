# 物理信息神经网络用于逆偏微分方程问题

> 原文：[`towardsdatascience.com/physics-informed-neural-networks-for-inverse-pde-problems/`](https://towardsdatascience.com/physics-informed-neural-networks-for-inverse-pde-problems/)

使用物理信息神经网络（PINN）感觉就像给一个常规神经网络提供作弊单。没有作弊单，我们可以仅使用神经网络估计物理系统的解。位置 *(x)* 和时间 *(t)* 作为输入，温度 *(u)* 作为输出。如果有足够的数据，这个解将是有效的。然而，它没有利用我们关于系统的物理知识。我们预计温度将遵循热量方程的动力学，我们也希望将其纳入我们的神经网络中。

PINNs 提供了一种结合系统已知物理和神经网络估计的方法。这是通过利用自动微分和基于物理的损失函数巧妙实现的。因此，我们可以用更少的数据获得更好的结果。

### 日程

+   提供对热量方程的解释

+   使用温度数据模拟数据

+   使用 DeepXDE 编写求解热扩散率***κ***和热源***q(x,t)***的代码

+   解释偏微分方程理论中正问题与逆问题的区别

这是我们将要处理的数据。让我们假设我们使用了传感器收集了 1 米长棒在 5 秒内的温度。

![自动微分和基于物理的损失函数](img/168060a9ec03feafa2e95c68e5119768.png)

温度（u）相对于位置（x）和时间（t）的图形。

作者插画

![偏微分方程理论中正问题与逆问题的区别](img/693295437b7d204673d4cff88bf48721.png)

热量方程

作者插画

> 简而言之，PINNs 通过使用底层系统的数据和我们的物理方程，提供了一种新的方法来近似物理方程（常微分方程、偏微分方程、随机微分方程）的解。

### 解释热量方程

![作者插画](img/0933dee56c2ba4c81056567c38ec4466.png)

热量方程中的逆问题术语。

作者插画

左侧的偏导数表示温度随时间的变化。这是一个关于位置（x）和时间的函数。在右侧，q(x,t)表示进入系统的热量。这是我们用本生灯加热棒的过程。中间项描述了热量根据周围点的变化。热量从热点流向冷点，寻求与周围点达到平衡。二阶空间导数（1D 中的∂²u/∂x²，或高维中的∇²u）捕捉热量扩散。这是热量从热区流向冷区的自然趋势。

这个项乘以热扩散率(κ)，它取决于材料的属性。我们预计像金属这样的导电材料会更快地升温。当∇²u 为正时，该点的温度低于其邻居的平均温度，因此热量倾向于流向该点。当∇²u 为负时，该点比其周围更热，热量倾向于流出。当∇²u 为零时，该点与其直接邻域处于热平衡状态。

在下面的图像中，函数的顶部可能代表一个非常热的点。注意拉普拉斯值是负的，这表明热量将从这个热点流向周围的较冷点。拉普拉斯是围绕一点的曲率的度量。在热方程中，这是温度分布的曲率。

![](img/1546e3d0e11b1f9bfee17a3a357a1edd.png)

高斯峰和 Gaussian 峰的拉普拉斯。

作者插图

### 生成数据

我必须承认，我实际上并没有烧一根棒来测量其随时间变化的温度变化。我使用热方程来模拟数据。这是我们用来模拟数据的代码。所有这些都可以在我的[GitHub](https://github.com/marco-hening-tallarico/Notebooks/blob/main/Copy_of_hot_rod_4.ipynb)上找到。

```py
#--- Generating Data ---
L = 1.0  # Rod Length (m)
Nx = 51  # Number of spatial points
dx = L / (Nx - 1)  # Spatial step
T_total = 5.0  # Total time (s)
Nt = 5000  # Number of time steps
dt = T_total / Nt  # Time step
kappa = 0.01  # Thermal diffusivity (m²/s)
q = 1.0  # Constant heat source term (C/s)

u = np.zeros(Nx)
x_coords = np.linspace(0, L, Nx)
temperature_data_raw = []

header = ["Time (s)"] + [f"x={x:.2f}m" for x in x_coords]
temperature_data_raw.append(header)
temperature_data_raw.append([0.0] + u.tolist())

for n in range(1, Nt + 1):
    u_new = np.copy(u)
    for i in range(1, Nx - 1):
        u_new[i] = u[i] + dt * (kappa * (u[i+1] - 2*u[i] + u[i-1]) / (dx**2) + q)
    u_new[0] = 0.0
    u_new[Nx-1] = 0.0
    u = u_new
    if n % 50 == 0 or n == Nt:
        temperature_data_raw.append([n * dt] + u.tolist()) 
```

为了生成这些数据，我们使用了*κ = 0.01 和 q = 1*，但只将*x, t*和*u*用于估计*κ*和*q*。换句话说，我们假装不知道*κ*和*q*，并试图仅用*x, t*和*u*来估计它们。这个表面是三维的，但它代表了随时间变化的**一维棒**的温度。

![](img/20cce7244c1b1b1874f6cbf4d1d35e29.png)

作者插图

![](img/022b68314977e8f1390f7c0947c1182e.png)

数据框

作者插图

### 安排和分割数据

在这里，我们只是将我们的数据重新排列成列，用于位置*(x)*，时间*(t)*和温度*(u_val)*，然后将它们分别分离成 X 和 Y，最后将它们分成训练集和测试集。

```py
# --- Prepare (x, t, u) triplet data ---
data_triplets = []
for _, row in df.iterrows():
t = row["Time (s)"]
for col in df.columns[1:]:
x = float(col.split('=')[1][:-1])
u_val = row[col]
data_triplets.append([x, t, u_val])
data_array = np.array(data_triplets)

X_data = data_array[:, 0:2] # X position (x), time (t)
y_data = data_array[:, 2:3] # Y temperature (u)
```

![](img/ccbf2fb983d85bc48c4b23fee3d9fc6c.png)

X 数据位置(x)，时间(t)，Y 数据温度(u)

作者插图

我们保持测试大小(20%)

```py
# --- Train/test split ---
from sklearn.model_selection import train_test_split
x_train, x_test, u_train, u_test = train_test_split(X_data, y_data, test_size=0.2, random_state=42)
Train Test Split 
```

因为我们的 PINN 接收位置(x)和时间(t)作为输入，并使用自动微分和链式法则，它可以计算以下偏导数。

\[

\frac{\partial u}{\partial t}, \quad

\frac{\partial u}{\partial x}, \quad

\frac{\partial² u}{\partial x²}, \quad

\nabla² u

\]

因此，找到常数变成了测试*κ*和*q(x, t)*的不同值的问题，最小化由损失函数给出的残差。

### 包和种子以及连接后端

如果您还没有安装，别忘了安装**DeepXDE**。

```py
!pip install --upgrade deepxde
```

这些都是我们将要使用的库。为了使其正常工作，请确保您使用**Tensorflow 2**作为 DeepXDE 的后端。

```py
# --- Imports and Connecting Backends ---

import os
os.environ["DDE_BACKEND"] = "tensorflow"  # Set to TF2 backend
import deepxde as dde
print("Backend:", dde.backend.__name__)  # Should now say: deepxde.backend.tensorflow
import tensorflow as tf
print("TensorFlow version:", tf.__version__)
print("Is eager execution enabled?", tf.executing_eagerly())
import deepxde as dde
print("DeepXDE version:", dde.__version__)
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
from deepxde.backend import tf
import random
import torch
```

为了确保可重复性，我们将我们的**种子**设置为**42**。您可以在多个库上使用此代码。

```py
# --- Setting Seeds ---
SEED = 42

random.seed(SEED)
np.random.seed(SEED)
os.environ['PYTHONHASHSEED'] = str(SEED)
tf.random.set_seed(SEED)
torch.manual_seed(SEED)
torch.cuda.manual_seed(SEED)
torch.cuda.manual_seed_all(SEED)
torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False
```

## 编码 PINN

### 环境

因为热方程模型在空间和时间上模拟温度，我们需要考虑空间域和时间域。

+   空间(0, 1)用于 1 米长的杆

+   时间(0,5)用于 5 秒的观察

+   Geomtime 结合了这些维度

```py
# --- Geometry and domain ---
geom = dde.geometry.Interval(0, 1)
timedomain = dde.geometry.TimeDomain(0, 5.0)
geomtime = dde.geometry.GeometryXTime(geom, timedomain)
```

选择您从数据中想要推断的 PDE 中的值。在这里，我们选择*kappa (κ)*和热源*q*。

```py
# --- Trainable variables ---
raw_kappa = tf.Variable(0.0)
raw_q = tf.Variable(0.0)
```

### 物理损失

我们的物理损失很简单：热方程一侧的所有元素。当这为零时，我们的方程成立。物理损失将被最小化，因此我们对*κ*和*q*的估计将最好地符合物理规律。如果我们有一个物理方程*A = B*，我们只需将所有元素移到一侧，并将我们的残差定义为*A – B = 0*。*A – B*越接近零，我们的 PINN 就越能捕捉到*A = B*的动态。

```py
def pde(x, u):
    du_t = dde.grad.jacobian(u, x, j=1)
    du_xx = dde.grad.hessian(u, x, i=0, j=0)
    kappa = tf.nn.softplus(raw_kappa)
    q = raw_q
    return du_t - kappa * du_xx - q
```

\[

\text{残差}(x, t) = \frac{\partial u}{\partial t} – \kappa \frac{\partial² u}{\partial x²} – q

\]

### PINNs

存在于残差中的导数是通过在**反向传播**期间通过计算图应用链式法则来计算的。这些导数允许 PINN 评估 PDE 的残差。

可选地，我们还可以添加数据损失，也称为标准神经网络的损失，它最小化预测值和已知值之间的差异。

```py
# --- Adding Data Loss ---
def custom_loss(y_true, y_pred):
    base_loss = tf.reduce_mean(tf.square(y_true - y_pred))
    reg = 10.0 * (tf.square(tf.nn.softplus(raw_kappa) - 0.01) + tf.square(raw_q - 1.0))
    return base_loss + reg #Loss from Data + Loss from PDE
```

下面，我们创建一个**TimePDE**数据对象，这是 DeepXDE 中用于求解时间相关 PDEs 的一种数据集类型。它为训练 PINN 准备了几何形状、物理、边界条件和初始条件。

```py
# --- DeepXDE Data object ---
data = dde.data.TimePDE(
    geomtime,
    pde,     #loss function
    [dde.PointSetBC(x_train, u_train)],  # Observed values as pseudo-BC
    num_domain=10000,
    num_boundary=0,
    num_initial=0,
    anchors=x_test,
)
```

使用[2] + [64]*3 + [1]的架构。我们从这个两个输入*(x, t)*，64 个神经元，3 个隐藏层，和 1 个输出*(u)*中获得。

> [2] + [64]*3 + [1] = [2, 64, 64, 64, 1]

使用**双曲正切**激活函数来捕捉 PDE 解中的线性和非线性行为。使用**权重初始化器**“Glorot normal”来防止训练中的梯度消失或爆炸。

```py
# --- Neural Network ---
net = dde.maps.FNN([2] + [64]*3 + [1], "tanh", "Glorot normal")
model = dde.Model(data, net)
```

我们可以使用不同的优化器。对我来说，**L-BFGS-B**效果更好。

```py
# --- Train with Adam ---
model.compile("adam", lr=1e-4, loss=custom_loss,
              external_trainable_variables=[raw_kappa, raw_q])
losshistory, train_state = model.train(iterations=100000)

# --- Optional L-BFGS-B fine-tuning ---
model.compile("L-BFGS-B", loss=custom_loss,
              external_trainable_variables=[raw_kappa, raw_q])
```

训练可能需要一些时间…

### 模型损失

随着时间推移监控模型损失是一个很好的方法来观察过拟合。因为我们只使用了物理损失，所以我们看不到组件 2，否则它将是数据损失。由于所有代码都上传到了我的[GitHub](https://github.com/marco-hening-tallarico/Notebooks)，您可以随意运行它并查看改变**学习率**如何影响模型损失的方差。

```py
# --- Plot loss ---
dde.utils.plot_loss_history(losshistory)
plt.yscale("log")
plt.title("Training Loss (log scale)")
plt.xlabel("Iteration")
plt.ylabel("Loss")
plt.grid(True)
plt.show()

# --- Detailed loss plotting ---
losses = np.array(losshistory.loss_train)  # shape: (iterations, num_components)
iterations = np.arange(1, len(losses) + 1)

plt.figure(figsize=(10, 6))
plt.plot(iterations, losses[:, 0], label="Train Total Loss")

# If there are multiple components (e.g., PDE + BC + data), plot them
if losses.shape[1] > 1:
    for i in range(1, losses.shape[1]):
        plt.plot(iterations, losses[:, i], label=f"Train Loss Component {i}")

# Plot validation loss if available
if losshistory.loss_test:
    val_losses = np.array(losshistory.loss_test)
    plt.plot(iterations, val_losses[:, 0], '--', label="Validation Loss", color="black")

    # Optionally: plot validation loss components
    if val_losses.shape[1] > 1:
        for i in range(1, val_losses.shape[1]):
            plt.plot(iterations, val_losses[:, i], '--', label=f"Validation Loss Component {i}", alpha=0.6)

plt.xlabel("Iteration")
plt.ylabel("Loss")
plt.yscale("log")
plt.title("Training and Validation Loss Over Time")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

![图片](img/30d183adc7b986ca21243cc3f8a90cf6.png)

作者插图

![图片](img/5c8021df5aa8b758164050a55e38aa34.png)

作者插图

### 结果

我们可以非常精确地推断这些常数。部分成功归因于我们只关注物理损失而没有结合我们的数据损失。这是 PINNs 中的一个选项。这里的准确性也归因于数据生成过程中**噪声**的缺失。

```py
# --- Results ---
learned_kappa = tf.nn.softplus(raw_kappa).numpy()
learned_q = raw_q.numpy()
print("\n--- Results ---")
print(f"True kappa: 0.01, Learned kappa: {learned_kappa:.6f}")
print(f"True q: 1.0, Learned q: {learned_q:.6f}")
```

![图片](img/4e29d0983511ded3140c2a8f6ff48276.png)

## 正向和逆向问题：

在这篇文章中，我们解决了 PDE 的**逆向问题**。这涉及到求解两个红色常数。

![](img/c7903b9ae3e73ad5df55562f05b398d5.png)

热方程中的正向和逆向问题。

作者插画

**正向问题**被描述如下：给定偏微分方程、基础参数、边界条件和强迫条件，我们希望计算系统的状态。在这种情况下，温度 *(u)*。这个问题涉及到预测系统。正向问题通常**是良定的**；解存在且唯一。这些解对输入是连续依赖的

**逆向问题**被描述如下：给定系统的状态（温度），推断出解释观察数据最佳的基础参数、边界条件或强迫项。在这里，我们估计未知参数。逆向问题通常是**病态的**，缺乏唯一性或稳定性。

> **正向**：当你知道原因时预测结果。
> 
> **逆向**：从观察到的结果中找出原因（或最佳输入）。

出乎意料的是，逆向问题通常首先解决。了解参数极大地帮助于解决正向问题。如果我们能够找出*kappa（κ）*和*q(x, t)*，求解温度*u(x,t)*将会容易得多。

## 结论

PINNs 提供了一种解决物理方程中逆向和正向问题的创新方法。与神经网络相比，它们的优点是能够使我们用更少的数据更好地解决这些问题，因为它们将物理知识融入神经网络中。这也带来了改进泛化的额外好处。PINNs 特别擅长解决逆向问题。

我接下来的文章：

Tallarico, M. H. (2025, September 3). 随机微分方程和温度——NASA 气候数据（第二部分）：Python 中的 Ornstein–Uhlenbeck 过程。数据科学之路。[链接](https://towardsdatascience.com/stochastic-differential-equations-and-temperature-nasa-climate-data-pt-2/)。[谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:Y0pCki6q_DkC)。

我之前的文章：

Tallarico, M. H. (2025, July 1). 如何访问 NASA 的气候数据——以及它是如何助力对抗气候变化的（第一部分）：从建筑设计到粮食安全。数据科学之路。[链接](https://towardsdatascience.com/how-to-access-nasas-climate-data-and-how-its-powering-the-fight-against-climate-change-pt-1/)。[谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:Tyk-4Ss8FVUC)

## 参考文献

+   Raissi, M., Perdikaris, P., & Karniadakis, G. E. (2019). 物理信息神经网络：解决涉及非线性偏微分方程的前向和逆问题的深度学习框架. *计算物理杂志，378*，686–707\. [`doi.org/10.1016/j.jcp.2018.10.045`](https://doi.org/10.1016/j.jcp.2018.10.045)

+   Raissi, M. (2018). 深度隐藏物理模型：非线性偏微分方程的深度学习. *机器学习研究杂志，19*(25)，1–24\. [`arxiv.org/abs/1801.06637`](https://arxiv.org/abs/1801.06637)

+   陆，孟，毛，& 卡尼亚迪斯. G. E. (2021). *DeepXDE：一个用于求解微分方程的深度学习库*. **SIAM 评论，63**(1)，208–228\. [`doi.org/10.1137/19M1274067`](https://doi.org/10.1137/19M1274067)

+   DeepXDE 开发者. (n.d.). *DeepXDE：一个用于求解微分方程的深度学习库* [计算机软件文档]. 收取 *2025 年 7 月 25 日*，来自 [`deepxde.readthedocs.io/en/latest/`](https://deepxde.readthedocs.io/en/latest/)

+   任，周，刘，& 刘. (2025). 物理信息神经网络：方法演变、理论基础和跨学科前沿综述，*应用科学，15*(14)，文章 8092\. [`doi.org/10.3390/app15148092`](https://doi.org/10.3390/app15148092) [MDPI](https://www.mdpi.com/2076-3417/15/14/8092?utm_source=chatgpt.com)

+   托雷斯，施菲尔，& 尼佩特. M. (2025). *自适应物理信息神经网络：综述*. arXiv. [`arxiv.org/abs/2503.18181`](https://arxiv.org/abs/2503.18181) [arXiv+1OpenReview+1](https://arxiv.org/abs/2503.18181?utm_source=chatgpt.com)

* * *

[网站](https://marcoheningtallarico.com/) | [领英](https://www.linkedin.com/in/marco-hening-tallarico/) | [GitHub](https://github.com/marco-hening-tallarico?tab=repositories)

![](img/a8fda6e571926b12e7a40a9644caddc3.png)

作者

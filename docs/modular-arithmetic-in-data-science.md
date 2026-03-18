# 数据科学中的模运算

> 原文：[`towardsdatascience.com/modular-arithmetic-in-data-science/`](https://towardsdatascience.com/modular-arithmetic-in-data-science/)

*<mdspan datatext="el1755445157197" class="mdspan-comment">模运算</mdspan>* 是一种数学系统，其中数字在达到一个称为**模数**的值后循环**回到起点**。这个系统通常被称为“时钟算术”，因为它与模拟 12 小时时钟表示时间的方式相似。本文提供了模运算的概念概述，并探讨了数据科学中的实际应用案例。

## 概念概述

### 基础知识

模运算定义了一个基于称为模数的特定整数的整数运算系统。表达式 *x mod d* 等价于 *x* 除以 *d* 所得到的余数。如果 *r* ≡ *x mod d*，则说 *r* 与 *x mod d* **同余**。换句话说，这意味着 *r* 和 *x* 之间的差是 *d* 的倍数，或者说 *x – r* 能被 *d* 整除。在模运算中，使用符号‘≡’（三条水平线）而不是‘=’，以强调我们处理的是同余而不是通常意义上的相等。

例如，在模 7 的情况下，数字 10 与 3 同余，因为 10 除以 7 的余数是 3。因此，我们可以写成 3 ≡ 10 *mod* 7。在 12 小时钟的情况下，凌晨 2 点与下午 2 点（即 14 *mod* 12）同余。在 Python 等编程语言中，百分号（‘%’）用作模运算符（例如，`10 % 7` 将计算为 3）。

这里有一个视频，更详细地解释了这些概念：

### 解线性同余

一个**线性同余**可以是一个形式为 *n* ⋅ *y* ≡ *x* (mod *d*) 的模表达式，其中系数 *n*、目标 *x* 和模数 *d* 是已知的整数，未知整数 *y* 的一次方（即，它没有被平方、立方等）。表达式 2017 ⋅ *y* ≡ 2025 (mod 10000) 是一个线性同余的例子；它表明当 2017 乘以某个整数 *y* 时，该乘积除以 10000 的余数是 2025。要解表达式 *n* ⋅ *y* ≡ *x* (mod *d*) 中的 *y*，请遵循以下步骤：

1.  找到系数 *n* 和模数 *d* 的最大公约数（GCD），也写作 GCD(*n*, *d*), 它是既是 *n* 也是 *d* 的最大正整数。可以使用[扩展欧几里得算法](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm)来高效地计算 GCD；这也会得到 *n^(-1)* 的一个候选值，即系数 *n* 的**模逆**。

1.  确定是否存在解。如果目标 *x* 不能被 GCD(*n*, *d*) 整除，则方程无解。这是因为同余只有在 GCD 能整除目标时才有解。

1.  如有必要，通过将系数 *n*、目标 *x* 和模数 *d* 除以最大公约数 GCD(*n*, *d*) 来简化模表达式，将问题简化为更简单的等价形式；我们分别称这些简化量为 *n[0]*、*x[0]* 和 *d[0]*。这确保了 **n[0]** 和 **d[0]** 是 *互质的*（即，1 是它们的唯一公约数），这对于找到模逆是必要的。

1.  计算模逆 *n[0]^(-1)*，即 ***n[0]*** mod ***d[0]*** 的模逆（再次使用扩展欧几里得算法）。

1.  找到一个未知值 *y* 的解。为此，将模逆 *n[0]^(-1)* 乘以简化后的目标 *x[0]*，以获得 *y* mod **d[0]** 的一个有效解。

1.  最后，基于步骤 5 的结果，生成所有可能的解。由于原始方程通过 GCD(*n*, *d*) 被简化，因此有 GCD(*n*, *d*) 个不同的解。这些解均匀地分布在简化的模数 **d[0]** 之间，并且所有解都与原始模数 *d* 相对应。

以下是对上述过程的 Python 实现：

```py
def extended_euclidean_algorithm(a, b):
    """
    Computes the greatest common divisor of positive integers a and b,
    along with coefficients x and y such that: a*x + b*y = gcd(a, b)
    """
    if b == 0:
        return (a, 1, 0)
    else:
        gcd, x_prev, y_prev = extended_euclidean_algorithm(b, a % b)
        x = y_prev
        y = x_prev - (a // b) * y_prev
        return (gcd, x, y)

def solve_linear_congruence(coefficient, target, modulus):
    """
    Solves the linear congruence: coefficient * y ≡ target (mod modulus)
    Returns all integer solutions for y with respect to the modulus.
    """
    # Step 1: Compute the gcd
    gcd, _, _ = extended_euclidean_algorithm(coefficient, modulus)

    # Step 2: Check if a solution exists
    if target % gcd != 0:
        print("No solution exists: target is not divisible by gcd.")
        return None

    # Step 3: Reduce the equation by gcd
    reduced_coefficient = coefficient // gcd
    reduced_target = target // gcd
    reduced_modulus = modulus // gcd

    # Step 4: Find the modular inverse of reduced_coefficient with respect to the reduced_modulus
    _, inverse_reduced, _ = extended_euclidean_algorithm(reduced_coefficient, reduced_modulus)
    inverse_reduced = inverse_reduced % reduced_modulus

    # Step 5: Compute one solution
    base_solution = (inverse_reduced * reduced_target) % reduced_modulus

    # Step 6: Generate all solutions modulo the original modulus
    all_solutions = [(base_solution + i * reduced_modulus) % modulus for i in range(gcd)]

    return all_solutions
```

这里有一些示例测试：

```py
solutions = solve_linear_congruence(coefficient=2009, target=2025, modulus=10000)
print(f"Solutions for y: {solutions}")

solutions = solve_linear_congruence(coefficient=20, target=16, modulus=28)
print(f"Solutions for y: {solutions}")
```

结果：

```py
Solutions for y: [225]
Solutions for y: [5, 12, 19, 26]
```

这段视频更详细地解释了如何解决线性同余：

## 数据科学用例

### 用例 1：特征工程

在数据科学中，模运算有许多有趣的用例。一个直观的用例是在特征工程的背景下，用于编码像一天中的小时这样的循环特征。由于时间每 24 小时就会循环一次，将小时视为线性值可能会错误地表示关系（例如，晚上 11 点和凌晨 1 点在数值上相距很远，但在时间上很接近）。通过应用模编码（例如，使用小时模 24 的正弦和余弦变换），我们可以保留时间的循环性质，使机器学习（ML）模型能够识别在特定时间段（如夜间）发生的模式。以下 Python 代码展示了如何实现这种编码：

```py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Example: List of incident hours (in 24-hour format)
incident_hours = [22, 23, 0, 1, 2]  # 10 PM to 2 AM

# Convert to a DataFrame
df = pd.DataFrame({'hour': incident_hours})

# Encode using sine and cosine transformations
df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)
```

生成的数据框 `df`：

```py
 hour  hour_sin  hour_cos
0    22 -0.500000  0.866025
1    23 -0.258819  0.965926
2     0  0.000000  1.000000
3     1  0.258819  0.965926
4     2  0.500000  0.866025
```

注意到正弦的使用如何区分 12 点之前和之后的小时（例如，将晚上 11 点和凌晨 1 点分别编码为 -0.258819 和 0.258819），而余弦的使用则没有这种区分（例如，晚上 11 点和凌晨 1 点都被映射到值 0.965926）。编码的最佳选择将取决于要部署机器学习模型的业务环境。最终，这项技术增强了特征工程，适用于异常检测、预测和分类等任务，在这些任务中，时间邻近性很重要。

在以下章节中，我们将考虑两个涉及求解形式为 *n* ⋅ *y* ≡ *x* (mod *d*) 的模表达式中 *y* 的较大数据科学用例。

### 用例 2：分布式数据库系统中的重新分片

在分布式数据库中，数据通常使用哈希函数跨多个节点进行分区（或 *分片*）。当分片数量发生变化时——比如说，从 *d* 变为 *d’*——我们需要有效地重新分片数据，而无需从头开始重新哈希一切。

假设每个数据项都按照以下方式分配到片段：

`shard = hash(key) mod d`

当将项目重新分配到新的 *d’* 片段集时，我们可能希望以保持平衡和最小化数据移动的方式将旧片段索引映射到新索引。这可能导致在表达式 *n* ⋅ *y* ≡ *x* (mod *d*) 中求解 *y*，其中：

+   *x* 是原始片段索引，

+   *d* 是旧的片段数量，

+   *n* 是缩放因子（或变换系数），

+   *y* 是我们正在求解的新片段索引

在此上下文中使用模数算术确保了旧片段布局和新片段布局之间的一致映射，最小化了重新分配，保持了数据局部性，并在重新分片期间实现了确定性和可逆的转换。

下面是这个场景的 Python 实现：

```py
def extended_euclidean_algorithm(a, b):
    """
    Computes gcd(a, b) and coefficients x, y such that: a*x + b*y = gcd(a, b)
    Used to find modular inverses.
    """
    if b == 0:
        return (a, 1, 0)
    else:
        gcd, x_prev, y_prev = extended_euclidean_algorithm(b, a % b)
        x = y_prev
        y = x_prev - (a // b) * y_prev
        return (gcd, x, y)

def modular_inverse(a, m):
    """
    Returns the modular inverse of a modulo m, if it exists.
    """
    gcd, x, _ = extended_euclidean_algorithm(a, m)
    if gcd != 1:
        return None  # Inverse doesn't exist if a and m are not coprime
    return x % m

def reshard(old_shard_index, old_num_shards, new_num_shards):
    """
    Maps an old shard index to a new one using modular arithmetic.

    Solves: n * y ≡ x (mod d)
    Where:
        x = old_shard_index
        d = old_num_shards
        n = new_num_shards
        y = new shard index (to solve for)
    """
    x = old_shard_index
    d = old_num_shards
    n = new_num_shards

    # Step 1: Check if modular inverse of n modulo d exists
    inverse_n = modular_inverse(n, d)
    if inverse_n is None:
        print(f"No modular inverse exists for n = {n} mod d = {d}. Cannot reshard deterministically.")
        return None

    # Step 2: Solve for y using modular inverse
    y = (inverse_n * x) % d
    return y
```

示例测试：

```py
import hashlib

def custom_hash(key, num_shards):
    hash_bytes = hashlib.sha256(key.encode('utf-8')).digest()
    hash_int = int.from_bytes(hash_bytes, byteorder='big')
    return hash_int % num_shards

# Example usage
old_num_shards = 10
new_num_shards = 7

# Simulate resharding for a few keys
keys = ['user_123', 'item_456', 'session_789']
for key in keys:
    old_shard = custom_hash(key, old_num_shards)
    new_shard = reshard(old_shard, old_num_shards, new_num_shards)
    print(f"Key: {key} | Old Shard: {old_shard} | New Shard: {new_shard}")
```

注意，我们使用的是一种自定义的哈希函数，它对 `key` 和 `num_shards` 是确定性的，以确保可重复性。

结果：

```py
Key: user_123 | Old Shard: 9 | New Shard: 7
Key: item_456 | Old Shard: 7 | New Shard: 1
Key: session_789 | Old Shard: 2 | New Shard: 6
```

### 用例 3：联邦学习中的差分隐私

在联邦学习中，ML 模型在去中心化的设备上训练，同时保护用户隐私。差分隐私通过向梯度更新添加噪声来掩盖设备间的个体贡献。通常，这种噪声是从离散分布中抽取的，并且必须进行模数缩减以适应有界范围。

假设一个客户端发送一个更新 *x*，服务器应用形式为 *n* ⋅ (*y* + *k*) ≡ *x* (mod *d*) 的转换，其中：

+   *x* 是发送到服务器的噪声梯度更新，

+   *y* 是原始（或真实）的梯度更新，

+   *k* 是噪声项（从整数范围中随机抽取），

+   *n* 是编码因子，

+   *d* 是模数（例如，所有操作发生的有限域或量化范围的大小）

由于此设置的隐私保护特性，服务器只能恢复 *y* + *k*，即有噪声的更新，但不能恢复真正的更新 *y* 本身。

下面是现在熟悉的 Python 设置：

```py
def extended_euclidean_algorithm(a, b):
    if b == 0:
        return a, 1, 0
    else:
        gcd, x_prev, y_prev = extended_euclidean_algorithm(b, a % b)
        x = y_prev
        y = x_prev - (a // b) * y_prev
        return gcd, x, y

def modular_inverse(a, m):
    gcd, x, _ = extended_euclidean_algorithm(a, m)
    if gcd != 1:
        return None
    return x % m
```

示例测试模拟一些客户端：

```py
import random

# Parameters
d = 97  # modulus (finite field)
noise_scale = 20  # controls magnitude of noise

# Simulated clients
clients = [
    {"id": 1, "y": 12, "n": 17},
    {"id": 2, "y": 23, "n": 29},
    {"id": 3, "y": 34, "n": 41},
]

# Step 1: Clients add noise and mask their gradients
random.seed(10)
for client in clients:
    noise = random.randint(-noise_scale, noise_scale)
    client["noise"] = noise
    noisy_y = client["y"] + noise
    client["x"] = (client["n"] * noisy_y) % d

# Step 2: Server receives x, knows n, and recovers noisy gradients
for client in clients:
    inv_n = modular_inverse(client["n"], d)
    client["y_noisy"] = (client["x"] * inv_n) % d

# Output
print("Client-side masking with noise:")
for client in clients:
    print(f"Client {client['id']}:")
    print(f"  True gradient y       = {client['y']}")
    print(f"  Added noise           = {client['noise']}")
    print(f"  Masked value x        = {client['x']}")
    print(f"  Recovered y + noise   = {client['y_noisy']}")
    print()
```

结果：

```py
Client-side masking with noise:
Client 1:
  True gradient y       = 12
  Added noise           = 16
  Masked value x        = 88
  Recovered y + noise   = 28

Client 2:
  True gradient y       = 23
  Added noise           = -18
  Masked value x        = 48
  Recovered y + noise   = 5

Client 3:
  True gradient y       = 34
  Added noise           = 7
  Masked value x        = 32
  Recovered y + noise   = 41
```

注意，服务器只能推导出噪声梯度，而不是原始梯度。

## 包装

模数算术，以其优雅的循环结构，不仅提供了一种聪明的计时方式，而且支撑着现代数据科学中一些最关键的机制。通过探索模数变换和线性同余，我们已经看到这个数学框架如何成为解决现实问题的强大工具。在特征工程、分布式数据库中的重新分片以及通过差分隐私在联邦学习中保护用户隐私等多样化的用例中，模数算术提供了构建健壮、可扩展系统所需的抽象和精度。随着数据科学的不断发展，这些模数技术的相关性可能会增加，这表明有时，创新的钥匙在于余数。

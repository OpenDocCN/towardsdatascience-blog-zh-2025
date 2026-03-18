# GPU 上的 NumPy API？

> 原文：[`towardsdatascience.com/numpy-api-on-a-gpu/`](https://towardsdatascience.com/numpy-api-on-a-gpu/)

## 这是否是 Python 数值计算的**未来**？

去年年底，NVIDIA 就 Python 基于数值计算的未来做出了重大公告。如果你错过了它，我并不感到惊讶。毕竟，当时和现在，每家 AI 公司的每个公告似乎都至关重要。

那个公告介绍了**cuNumeric**库**，**它是基于**Legate**框架的通用 NumPy 库的即插即用替代品。

### Nvidia 是谁？

大多数人可能都知道 Nvidia，因为其超快的芯片为全球的计算机和数据中心提供动力。你可能也熟悉 Nvidia 的魅力四射、喜欢皮夹克的 CEO，黄仁勋，他似乎最近出现在每个 AI 会议的舞台上。

许多人可能不知道，Nvidia 还设计和创建创新的设备架构及其相关软件。其最珍贵的产物之一是**统一计算设备架构**(CUDA)。CUDA 是 NVIDIA 的专有并行计算平台和编程模型。自 2007 年推出以来，它已发展成为一个包含驱动程序、运行时、编译器、数学库、调试和性能分析工具以及容器镜像的全面生态系统。结果是，一个精心调校的软硬件循环，使 NVIDIA GPU 成为现代高性能和 AI 工作负载的中心。

### Legate 是什么？

Legate 是一个由 NVIDIA 领导的开放源代码运行时层，它允许你在多核 CPU、单 GPU 或多 GPU 节点，甚至多节点集群上运行熟悉的 Python 数据科学库（NumPy、cuNumeric、Pandas 风格的 API、稀疏线性代数内核等），而无需更改你的 Python 代码。它将高级数组操作转换为细粒度任务的图，并将该图交给 C++ **Legion**运行时，该运行时负责调度任务、分区数据，并在 CPU、GPU 和网络链路之间移动瓦片。

简而言之，Legate 允许熟悉的单节点 Python 库透明地扩展到多 GPU、多节点机器。

### cuNumeric 是什么？

cuNumeric 是 NumPy 的即插即用替代品，其数组操作由 Legate 的任务引擎执行，并在一个或多个 NVIDIA GPU 上加速（如果没有 GPU，则在所有 CPU 核心上）。实际上，你只需安装它，并更改一行导入语句，就可以开始使用它替代常规 NumPy 代码。例如 …

```py
# old
import numpy as np
...
...

# new
import cupynumeric as np     # everything else stays the same
...
...
```

… 并使用 legate 命令在终端上运行你的脚本。

在幕后，cuNumeric 将你发出的每个 NumPy 调用（例如，np.sin、np.linalg.svd、花哨的索引、广播、归约等）转换为 Legate 任务。这些任务将，

1.  **将**数组**划分为适合 GPU 内存大小的瓦片**。

1.  **在**最佳可用设备（GPU 或 CPU）上**调度**每个瓦片。

1.  当工作负载跨越多个 GPU 或节点时，**Overlap** 计算与通信。

1.  当你的数据集超过 GPU RAM 时，**Spill** 瓦片会自动到 NVMe/SSD。

因为 cuNumeric 的 API 几乎与 NumPy 完全一致，现有的科学或数据科学代码可以在不重写的情况下从笔记本电脑扩展到多 GPU 集群。

### 性能优势

所以，这一切看起来都很棒，对吧？但这只有在它带来实际性能提升的情况下才有意义，而 Nvidia 正在做出一些强有力的声明，称这是事实。作为数据科学家、机器学习工程师和数据工程师，我们通常使用 NumPy 很多，我们可以欣赏这一点，这是我们编写和维护的系统的一个关键方面。

现在，我没有集群的 GPU 或超级计算机来测试这个，但我的台式机确实有一个 Nvidia GeForce RTX 4070 GPU，我们将使用它来测试 Nvidia 的一些声明。

```py
(base) tom@tpr-desktop:~$ nvidia-smi
Sun Jun 15 15:26:36 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 565.75                 Driver Version: 566.24         CUDA Version: 12.7     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4070 Ti     On  |   00000000:01:00.0  On |                  N/A |
| 32%   29C    P8              9W /  285W |    1345MiB /  12282MiB |      2%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

我将在我的电脑上安装 cuNumeric 和 NumPy 来进行对比测试。这将帮助我们评估 Nvidia 的声明是否准确，并了解两个库之间的性能差异。

## 设置开发环境。

如同往常，我喜欢设置一个独立的环境来运行我的测试。这样，我在该环境中所做的任何操作都不会影响我的其他项目。在撰写本文时，cuNumeric 不可在 Windows 上安装，所以我将使用 WSL2 Ubuntu for Windows。

我将使用 Miniconda 来设置我的环境，但请随意使用您感到舒适的任何工具。

```py
$ conda create cunumeric-env python=3.10 -c conda-forge
$ conda activate cunumeric-env
$ conda install -c conda-forge -c legate cupynumeric
$ conda install -c conda-forge ucx cuda-cudart cuda-version=12
```

### 代码示例 1 — 简单矩阵乘法

矩阵乘法是许多 AI 系统背后的数学运算的基础，因此尝试这个操作是有意义的。

> *请注意，在我所有的示例中，我将连续运行 NumPy 和 cuNumeric 代码片段五次，并平均每次运行的时间。* *我还执行了一个“在计时运行之前对 GPU 进行预热步骤，以考虑诸如即时（JIT）编译等开销。*

```py
import time
import gc
import argparse
import sys

def benchmark_numpy(n, runs):
    """Runs the matrix multiplication benchmark using standard NumPy on the CPU."""
    import numpy as np

    print(f"--- NumPy (CPU) Benchmark ---")
    print(f"Multiplying two {n}×{n} matrices ({runs} runs)\n")

    # 1\. Generate data ONCE before the timing loop.
    print(f"Generating two {n}x{n} random matrices on CPU...")
    A = np.random.rand(n, n).astype(np.float32)
    B = np.random.rand(n, n).astype(np.float32)

    # 2\. Perform one untimed warm-up run.
    print("Performing warm-up run...")
    _ = np.matmul(A, B)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(runs):
        start = time.time()
        # The operation being timed. The @ operator is a convenient
        # shorthand for np.matmul.
        C = A @ B
        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.4f}s")
        del C # Clean up the result matrix
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\nNumPy average: {avg:.4f}s\n")
    return avg

def benchmark_cunumeric(n, runs):
    """Runs the matrix multiplication benchmark using cuNumeric on the GPU."""
    import cupynumeric as cn
    import numpy as np # Import numpy for the canonical sync

    print(f"--- cuNumeric (GPU) Benchmark ---")
    print(f"Multiplying two {n}×{n} matrices ({runs} runs)\n")

    # 1\. Generate data ONCE on the GPU before the timing loop.
    print(f"Generating two {n}x{n} random matrices on GPU...")
    A = cn.random.rand(n, n).astype(np.float32)
    B = cn.random.rand(n, n).astype(np.float32)

    # 2\. Perform a crucial untimed warm-up run for JIT compilation.
    print("Performing warm-up run...")
    C_warmup = cn.matmul(A, B)
    # The best practice for synchronization: force a copy back to the CPU.
    _ = np.array(C_warmup)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(runs):
        start = time.time()

        # Launch the operation on the GPU
        C = A @ B

        # Synchronize by converting the result to a host-side NumPy array.
        np.array(C)

        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.4f}s")
        del C
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\ncuNumeric average: {avg:.4f}s\n")
    return avg

if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Benchmark matrix multiplication on NumPy (CPU) vs. cuNumeric (GPU)."
    )
    parser.add_argument(
        "-n", "--n", type=int, default=3000, help="Matrix size (n x n)"
    )
    parser.add_argument(
        "-r", "--runs", type=int, default=5, help="Number of timing runs"
    )
    parser.add_argument(
        "--cunumeric", action="store_true", help="Run the cuNumeric (GPU) version"
    )

    args, unknown = parser.parse_known_args()

    # The dispatcher logic
    if args.cunumeric or "--cunumeric" in unknown:
        benchmark_cunumeric(args.n, args.runs)
    else:
        benchmark_numpy(args.n, args.runs)
```

运行 NumPy 的部分使用的是常规的 **python example1.py** 命令行语法。对于使用 Legate 运行，语法更复杂。它的作用是禁用 Legate 的自动配置，然后在 Legate 下使用一个 CPU、一个 GPU 和零 OpenMP 线程启动 example1.py 脚本，使用 cuNumeric 后端。

这里是输出。

```py
(cunumeric-env) tom@tpr-desktop:~$ python example1.py
--- NumPy (CPU) Benchmark ---
Multiplying two 3000×3000 matrices (5 runs)

Generating two 3000x3000 random matrices on CPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.0976s
Run 2: time = 0.0987s
Run 3: time = 0.0957s
Run 4: time = 0.1063s
Run 5: time = 0.0989s

NumPy average: 0.0994s

(cunumeric-env) tom@tpr-desktop:~$ LEGATE_AUTO_CONFIG=0 legate --cpus 1 --gpus 1 --omps 0 example1.py --cunu
meric
[0 - 7f2e8fcc8480]    0.000000 {5}{module_config}: Module numa can not detect resources.
[0 - 7f2e8fcc8480]    0.000000 {4}{topology}: can't open /sys/devices/system/node/
[0 - 7f2e8fcc8480]    0.000049 {4}{threads}: reservation ('GPU ctxsync 0x55cd5fd34530') cannot be satisfied
--- cuNumeric (GPU) Benchmark ---
Multiplying two 3000×3000 matrices (5 runs)

Generating two 3000x3000 random matrices on GPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.0113s
Run 2: time = 0.0089s
Run 3: time = 0.0086s
Run 4: time = 0.0090s
Run 5: time = 0.0087s

cuNumeric average: 0.0093s
```

嗯，这是一个令人印象深刻的开始。cuNumeric 在速度上比 NumPy 快 10 倍。

Legate 输出的警告可以忽略。这些是信息性的，表明 Legate 找不到关于机器的 CPU/内存布局（NUMA）或足够的 CPU 核心来管理 GPU 的详细信息。

### 代码示例 2 — 逻辑回归

逻辑回归是数据科学中的一个基础工具，因为它提供了一种简单、可解释的方式来模拟和预测二元结果（是/否、通过/失败、点击/不点击）。在这个例子中，我们将测量在合成数据上训练一个简单二元分类器所需的时间。对于五次运行中的每一次，它首先生成**N**个具有**D**个特征（X）的样本，以及相应的随机 0/1 标签向量（Y）。它将权重向量`w`初始化为零，然后执行**500**次批量梯度下降：计算线性预测**z = X.dot(w**)，应用 sigmoid 函数**p = 1/(1+exp(–z))**，计算梯度**grad = X.T.dot(p – y) / N**，并使用**w -= 0.1 * grad**更新权重。脚本记录每次运行的耗时，清理内存，并最终打印平均训练时间。

```py
import time
import gc
import argparse
import sys

# --- Reusable Training Function ---
# By putting the training loop in its own function, we avoid code duplication.
# The `np` argument allows us to pass in either the numpy or cupynumeric module.
def train_logistic_regression(np, X, y, iters, alpha):
    """Performs a set number of gradient descent iterations."""
    # Ensure w starts on the correct device (CPU or GPU)
    w = np.zeros(X.shape[1])

    for _ in range(iters):
        z = X.dot(w)
        p = 1.0 / (1.0 + np.exp(-z))
        grad = X.T.dot(p - y) / X.shape[0]
        w -= alpha * grad

    return w

def benchmark_numpy(n_samples, n_features, iters, alpha):
    """Runs the logistic regression benchmark using standard NumPy on the CPU."""
    import numpy as np

    print(f"--- NumPy (CPU) Benchmark ---")
    print(f"Training on {n_samples} samples, {n_features} features for {iters} iterations\n")

    # 1\. Generate data ONCE before the timing loop.
    print("Generating random dataset on CPU...")
    X = np.random.rand(n_samples, n_features)
    y = (np.random.rand(n_samples) > 0.5).astype(np.float64)

    # 2\. Perform one untimed warm-up run.
    print("Performing warm-up run...")
    _ = train_logistic_regression(np, X, y, iters, alpha)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(args.runs):
        start = time.time()
        # The operation being timed
        _ = train_logistic_regression(np, X, y, iters, alpha)
        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.3f}s")
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\nNumPy average: {avg:.3f}s\n")
    return avg

def benchmark_cunumeric(n_samples, n_features, iters, alpha):
    """Runs the logistic regression benchmark using cuNumeric on the GPU."""
    import cupynumeric as cn
    import numpy as np # Also import numpy for the canonical synchronization

    print(f"--- cuNumeric (GPU) Benchmark ---")
    print(f"Training on {n_samples} samples, {n_features} features for {iters} iterations\n")

    # 1\. Generate data ONCE on the GPU before the timing loop.
    print("Generating random dataset on GPU...")
    X = cn.random.rand(n_samples, n_features)
    y = (cn.random.rand(n_samples) > 0.5).astype(np.float64)

    # 2\. Perform a crucial untimed warm-up run for JIT compilation.
    print("Performing warm-up run...")
    w_warmup = train_logistic_regression(cn, X, y, iters, alpha)
    # The best practice for synchronization: force a copy back to the CPU.
    _ = np.array(w_warmup)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(args.runs):
        start = time.time()

        # Launch the operation on the GPU
        w = train_logistic_regression(cn, X, y, iters, alpha)

        # Synchronize by converting the final result back to a NumPy array.
        np.array(w)

        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.3f}s")
        del w
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\ncuNumeric average: {avg:.3f}s\n")
    return avg

if __name__ == "__main__":
    # A more robust argument parsing setup
    parser = argparse.ArgumentParser(
        description="Benchmark logistic regression on NumPy (CPU) vs. cuNumeric (GPU)."
    )
    # Hyperparameters for the model
    parser.add_argument(
        "-n", "--n_samples", type=int, default=2_000_000, help="Number of data samples"
    )
    parser.add_argument(
        "-d", "--n_features", type=int, default=10, help="Number of features"
    )
    parser.add_argument(
        "-i", "--iters", type=int, default=500, help="Number of gradient descent iterations"
    )
    parser.add_argument(
        "-a", "--alpha", type=float, default=0.1, help="Learning rate"
    )
    # Benchmark control
    parser.add_argument(
        "-r", "--runs", type=int, default=5, help="Number of timing runs"
    )
    parser.add_argument(
        "--cunumeric", action="store_true", help="Run the cuNumeric (GPU) version"
    )

    args, unknown = parser.parse_known_args()

    # Dispatcher logic
    if args.cunumeric or "--cunumeric" in unknown:
        benchmark_cunumeric(args.n_samples, args.n_features, args.iters, args.alpha)
    else:
        benchmark_numpy(args.n_samples, args.n_features, args.iters, args.alpha)
```

以及输出结果。

```py
(cunumeric-env) tom@tpr-desktop:~$ python example2.py
--- NumPy (CPU) Benchmark ---
Training on 2000000 samples, 10 features for 500 iterations

Generating random dataset on CPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 12.292s
Run 2: time = 11.830s
Run 3: time = 11.903s
Run 4: time = 12.843s
Run 5: time = 11.964s

NumPy average: 12.166s

(cunumeric-env) tom@tpr-desktop:~$ LEGATE_AUTO_CONFIG=0 legate --cpus 1 --gpus 1 --omps 0 example2.py --cunu
meric
[0 - 7f04b535c480]    0.000000 {5}{module_config}: Module numa can not detect resources.
[0 - 7f04b535c480]    0.000000 {4}{topology}: can't open /sys/devices/system/node/
[0 - 7f04b535c480]    0.001149 {4}{threads}: reservation ('GPU ctxsync 0x55fb037cf140') cannot be satisfied
--- cuNumeric (GPU) Benchmark ---
Training on 2000000 samples, 10 features for 500 iterations

Generating random dataset on GPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 1.964s
Run 2: time = 1.957s
Run 3: time = 1.968s
Run 4: time = 1.955s
Run 5: time = 1.960s

cuNumeric average: 1.961s
```

与我们的第一个示例相比，并不那么令人印象深刻，但在一个已经很快的 NumPy 程序上实现 5 倍到 6 倍的速度提升也不容小觑。

### 代码示例 3—求解线性方程

此脚本测试了解决一个密集的 3000×3000 线性代数方程系统所需的时间。这是线性代数中的一个基本操作，用于解决类型为**Ax = b**的方程，其中 A 是一个巨大的数字网格（在本例中是一个 3000×3000 的矩阵），而 b 是一个数字列表（一个向量）。

目标是找到使方程成立的未知数字列表 x。这是一个计算密集型任务，是许多科学模拟、工程问题、金融模型甚至一些 AI 算法的核心。

```py
import time
import gc
import argparse
import sys # Import sys to check arguments

# Note: The library imports (numpy and cupynumeric) are now done *inside*
# their respective functions to keep them separate and avoid import errors.

def benchmark_numpy(n, runs):
    """Runs the linear solve benchmark using standard NumPy on the CPU."""
    import numpy as np

    print(f"--- NumPy (CPU) Benchmark ---")
    print(f"Solving {n}×{n} A x = b ({runs} runs)\n")

    # 1\. Generate data ONCE before the timing loop.
    print("Generating random system on CPU...")
    A = np.random.randn(n, n).astype(np.float32)
    b = np.random.randn(n).astype(np.float32)

    # 2\. Perform one untimed warm-up run. This is good practice even for
    # the CPU to ensure caches are warm and any one-time setup is done.
    print("Performing warm-up run...")
    _ = np.linalg.solve(A, b)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(runs):
        start = time.time()
        # The operation being timed
        x = np.linalg.solve(A, b)
        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.6f}s")
        # Clean up the result to be safe with memory
        del x
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\nNumPy average: {avg:.6f}s\n")
    return avg

def benchmark_cunumeric(n, runs):
    """Runs the linear solve benchmark using cuNumeric on the GPU."""
    import cupynumeric as cn
    import numpy as np # Also import numpy for the canonical synchronization

    print(f"--- cuNumeric (GPU) Benchmark ---")
    print(f"Solving {n}×{n} A x = b ({runs} runs)\n")

    # 1\. Generate data ONCE on the GPU before the timing loop.
    # This ensures we are not timing the data transfer in our main loop.
    print("Generating random system on GPU...")
    A = cn.random.randn(n, n).astype(np.float32)
    b = cn.random.randn(n).astype(np.float32)

    # 2\. Perform a crucial untimed warm-up run. This handles JIT
    # compilation and other one-time GPU setup costs.
    print("Performing warm-up run...")
    x_warmup = cn.linalg.solve(A, b)
    # The best practice for synchronization: force a copy back to the CPU.
    _ = np.array(x_warmup)
    print("Warm-up complete.\n")

    # 3\. Perform the timed runs.
    times = []
    for i in range(runs):
        start = time.time()

        # Launch the operation on the GPU
        x = cn.linalg.solve(A, b)

        # Synchronize by converting the result to a host-side NumPy array.
        # This is guaranteed to block until the GPU has finished.
        np.array(x)

        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.6f}s")
        # Clean up the GPU array result
        del x
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\ncuNumeric average: {avg:.6f}s\n")
    return avg

if __name__ == "__main__":
    # A more robust argument parsing setup
    parser = argparse.ArgumentParser(
        description="Benchmark linear solve on NumPy (CPU) vs. cuNumeric (GPU)."
    )
    parser.add_argument(
        "-n", "--n", type=int, default=3000, help="Matrix size (n x n)"
    )
    parser.add_argument(
        "-r", "--runs", type=int, default=5, help="Number of timing runs"
    )

    # Use parse_known_args() to handle potential extra arguments from Legate
    args, unknown = parser.parse_known_args()

    # The dispatcher logic: check if "--cunumeric" is in the command line
    # This is a simple and effective way to switch between modes.
    if "--cunumeric" in sys.argv or "--cunumeric" in unknown:
        benchmark_cunumeric(args.n, args.runs)
    else:
        benchmark_numpy(args.n, args.runs)
```

输出结果。

```py
(cunumeric-env) tom@tpr-desktop:~$ python example4.py
--- NumPy (CPU) Benchmark ---
Solving 3000×3000 A x = b (5 runs)

Generating random system on CPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.133075s
Run 2: time = 0.126129s
Run 3: time = 0.135849s
Run 4: time = 0.137383s
Run 5: time = 0.138805s

NumPy average: 0.134248s

(cunumeric-env) tom@tpr-desktop:~$ LEGATE_AUTO_CONFIG=0 legate --cpus 1 --gpus 1 --omps 0 example4.py --cunumeric
[0 - 7f29f42ce480]    0.000000 {5}{module_config}: Module numa can not detect resources.
[0 - 7f29f42ce480]    0.000000 {4}{topology}: can't open /sys/devices/system/node/
[0 - 7f29f42ce480]    0.000053 {4}{threads}: reservation ('GPU ctxsync 0x562e88c28700') cannot be satisfied
--- cuNumeric (GPU) Benchmark ---
Solving 3000×3000 A x = b (5 runs)

Generating random system on GPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.009685s
Run 2: time = 0.010043s
Run 3: time = 0.009966s
Run 4: time = 0.009739s
Run 5: time = 0.009383s

cuNumeric average: 0.009763s
```

这是一个惊人的结果。Nvidia cuNumeric 的运行速度比 NumPy 快 100 倍。

### 代码示例 4—排序

排序是计算中发生的一切的基础部分，现代计算机如此之快，以至于大多数开发者甚至都不考虑它。但让我们看看使用 cuNumeric 可以对这个普遍的操作带来多大的差异。我们将对一个大型（3000 万）的一维数字数组进行排序。

```py
# benchmark_sort.py
import time
import sys
import gc

# Array size
n = 30_000_000 # 30 million elements

def benchmark_numpy():
    import numpy as np
    print(f"Sorting an array of {n} elements with NumPy (5 runs)\n")

    times = []
    for i in range(5):
        data = np.random.randn(n).astype(np.float32)
        start = time.time()
        _ = np.sort(data)
        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.6f}s")
        del data
        gc.collect()

    avg = sum(times) / len(times)
    print(f"\nNumPy average: {avg:.6f}s\n")

def benchmark_cunumeric():
    import cupynumeric as np
    print(f"Sorting an array of {n} elements with cuNumeric (5 runs)\n")

    times = []
    for i in range(5):
        data = np.random.randn(n).astype(np.float32)
        start = time.time()
        _ = np.sort(data)
        # Force GPU sync
        _ = np.linalg.norm(np.zeros(()))
        end = time.time()

        duration = end - start
        times.append(duration)
        print(f"Run {i+1}: time = {duration:.6f}s")
        del data
        gc.collect()
        _ = np.linalg.norm(np.zeros(()))

    avg = sum(times) / len(times)
    print(f"\ncuNumeric average: {avg:.6f}s\n")

if __name__ == "__main__":
    if "--cunumeric" in sys.argv:
        benchmark_cunumeric()
    else:
        benchmark_numpy()
```

输出结果。

```py
(cunumeric-env) tom@tpr-desktop:~$ python example5.py
--- NumPy (CPU) Benchmark ---
Sorting an array of 30000000 elements (5 runs)

Creating random array on CPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.588777s
Run 2: time = 0.586813s
Run 3: time = 0.586745s
Run 4: time = 0.586525s
Run 5: time = 0.583783s

NumPy average: 0.586529s
-----------------------------

(cunumeric-env) tom@tpr-desktop:~$ LEGATE_AUTO_CONFIG=0 legate --cpus 1 --gpus 1 --omps 0 example5.py --cunumeric
[0 - 7fd9e4615480]    0.000000 {5}{module_config}: Module numa can not detect resources.
[0 - 7fd9e4615480]    0.000000 {4}{topology}: can't open /sys/devices/system/node/
[0 - 7fd9e4615480]    0.000082 {4}{threads}: reservation ('GPU ctxsync 0x564489232fd0') cannot be satisfied
--- cuNumeric (GPU) Benchmark ---
Sorting an array of 30000000 elements (5 runs)

Creating random array on GPU...
Performing warm-up run...
Warm-up complete.

Run 1: time = 0.010857s
Run 2: time = 0.007927s
Run 3: time = 0.007921s
Run 4: time = 0.008240s
Run 5: time = 0.007810s

cuNumeric average: 0.008551s
-------------------------------
```

cuNumeric 和 Legate 再次展现了令人难以置信的性能。

## 摘要

本文介绍了**cuNumeric**，这是一个由 NVIDIA 设计的用于替代 NumPy 的高性能库。关键点是，数据科学家可以通过最小的努力加速他们在 NVIDIA GPU 上现有的 Python 代码，通常只需更改一行导入语句，并使用**‘legate’**命令运行脚本。

该技术由两个主要组件驱动：

1.  **Legate:** 这是一个来自 NVIDIA 的开源运行时层，它自动将高级 Python 操作转换为任务。它智能地管理将这些任务分配到单个或多个 GPU 上，处理数据分区、内存管理（甚至在需要时将数据溢出到磁盘），并优化通信。

1.  **cuNumeric:** 这是一个面向用户的库，它映射了 NumPy API。当你调用 np.matmul()这样的函数时，cuNumeric 会将其转换为 Legate 引擎在 GPU 上执行的任务。

我通过在我的台式电脑（配备 NVIDIA RTX 4070 Ti GPU）上运行四个基准测试，验证了 Nvidia 的性能声明，比较了 CPU 上的标准 NumPy 与 GPU 上的 cuNumeric。

结果表明 cuNumeric 在性能上取得了显著提升：

+   **矩阵乘法：** 比 NumPy **~10x** 快。

+   **逻辑回归训练：** **~6x** 更快。

+   **解线性方程：** 巨大的 **100x+** 加速。

+   **排序大型数组：** 另一个巨大的改进，运行速度大约 **70x** 更快。

总之，我证明了 cuNumeric 成功地实现了其承诺，使得 GPU 的强大计算能力对更广泛的 Python 数据科学社区变得可访问，而无需陡峭的学习曲线或完全重写代码。

如需更多信息及相关资源的链接，请查看 Nvidia 关于 cuNumeric 的原版公告[此处](https://developer.nvidia.com/blog/nvidia-announces-availability-for-cunumeric-public-alpha/)。

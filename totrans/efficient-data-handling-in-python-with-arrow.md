# 使用 Arrow 在 Python 中高效处理数据

> 原文：[`towardsdatascience.com/efficient-data-handling-in-python-with-arrow/`](https://towardsdatascience.com/efficient-data-handling-in-python-with-arrow/)

## 1\. 简介

我们都习惯于使用 CSV、JSON 文件…使用传统库和大型数据集时，这些操作可能会非常慢，导致性能瓶颈（我经历过）。在大数据量中，高效地处理数据对于我们的数据科学/分析工作流程至关重要，这正是 Apache Arrow 发挥作用的地方。

为什么？主要原因在于数据在内存中的存储方式。例如，虽然 JSON 和 CSV 是基于文本的格式，但 Arrow 是一种列式内存数据格式（这允许不同数据处理工具之间快速的数据交换）。因此，Arrow 被设计为通过启用零拷贝读取、减少内存使用和支持高效压缩来优化性能。

此外，Apache Arrow 是开源的，并针对分析进行了优化。它旨在加速大数据处理，同时保持与各种数据工具（如 Pandas、Spark 和 Dask）的互操作性。通过以列式格式存储数据，Arrow 实现了更快的读写操作和高效的内存使用，使其非常适合分析工作负载。

听起来很棒，对吧？最好的是，这将是我要提供的关于 Arrow 的所有介绍。理论已经足够，我们想看到它在实际中的应用。因此，在这篇文章中，我们将探讨如何在 Python 中使用 Arrow 以及如何充分利用它。

## 2\. Python 中的 Arrow

要开始，您需要安装必要的库：pandas 和 pyarrow。

```py
pip install pyarrow pandas
```

然后，像往常一样，在您的 Python 脚本中导入它们：

```py
import pyarrow as pa
import pandas as pd
```

目前还没有什么新东西，只是完成后续步骤的必要步骤。让我们先执行一些简单的操作。

### 2.1\. 创建和存储表

我们能做的最简单的事情是将表的数据硬编码。让我们创建一个包含足球数据的两列表：

```py
teams = pa.array(['Barcelona', 'Real Madrid', 'Rayo Vallecano', 'Athletic Club', 'Real Betis'], type=pa.string())
goals = pa.array([30, 23, 9, 24, 12], type=pa.int8())

team_goals_table = pa.table([teams, goals], names=['Team', 'Goals'])
```

格式是 *pyarrow.table*，但如果我们想，可以轻松将其转换为 pandas：

```py
df = team_goals_table.to_pandas()
```

然后使用以下方式将其恢复为 arrow：

```py
team_goals_table = pa.Table.from_pandas(df)
```

最后，我们将表存储到文件中。我们可以使用不同的格式，如 feather、parquet… 我将使用最后一个，因为它速度快且内存优化：

```py
import pyarrow.parquet as pq
pq.write_table(team_goals_table, 'data.parquet')
```

读取一个 parquet 文件只需使用 `pq.read_table('data.parquet')`。

### 2.2\. 计算函数

Arrow 有自己的计算模块用于常规操作。让我们先从逐元素比较两个数组开始：

```py
import pyarrow.compute as pc
>>> a = pa.array([1, 2, 3, 4, 5, 6])
>>> b = pa.array([2, 2, 4, 4, 6, 6])
>>> pc.equal(a,b)
[
  false,
  true,
  false,
  true,
  false,
  true
]
```

这很简单，我们可以使用以下方式对数组中的所有元素求和：

```py
>>> pc.sum(a)
<pyarrow.Int64Scalar: 21>
```

从这里我们可以很容易地猜测我们可以计算计数、地板、指数、平均值、最大值、乘法…无需一一赘述。那么，让我们转向表格操作。

我们首先展示如何对其进行排序：

```py
>>> table = pa.table({'i': ['a','b','a'], 'x': [1,2,3], 'y': [4,5,6]})
>>> pc.sort_indices(table, sort_keys=[('y', descending)])
<pyarrow.lib.UInt64Array object at 0x1291643a0>
[
  2,
  1,
  0
]
```

就像在 pandas 中一样，我们可以对值进行分组并聚合数据。例如，我们可以按“i”分组，并在“x”上计算总和，在“y”上计算平均值：

```py
>>> table.group_by('i').aggregate([('x', 'sum'), ('y', 'mean')])
pyarrow.Table
i: string
x_sum: int64
y_mean: double
----
i: [["a","b"]]
x_sum: [[4,2]]
y_mean: [[5,5]]
```

或者我们可以连接两个表：

```py
>>> t1 = pa.table({'i': ['a','b','c'], 'x': [1,2,3]})
>>> t2 = pa.table({'i': ['a','b','c'], 'y': [4,5,6]})
>>> t1.join(t2, keys="i")
pyarrow.Table
i: string
x: int64
y: int64
----
i: [["a","b","c"]]
x: [[1,2,3]]
y: [[4,5,6]]
```

默认情况下，它是一个左外连接，但我们可以通过使用 ***join_type*** 参数来改变它。

还有更多有用的操作，但为了避免使内容过长，我们只看一个额外的操作：向表中追加一个新列。

```py
>>> t1.append_column("z", pa.array([22, 44, 99]))
pyarrow.Table
i: string
x: int64
z: int64
----
i: [["a","b","c"]]
x: [[1,2,3]]
z: [[22,44,99]]
```

在结束本节之前，我们必须看看如何过滤表或数组：

```py
>>> t1.filter((pc.field('x') > 0) & (pc.field('x') < 3))
pyarrow.Table
i: string
x: int64
----
i: [["a","b"]]
x: [[1,2]]
```

很简单，对吧？尤其是如果你已经使用 pandas 和 numpy 好几年了！

## 3. 与文件一起工作

我们已经看到了如何读取和写入 Parquet 文件。但让我们检查一些其他流行的文件类型，以便我们有几个选项可用。

### 3.1. Apache ORC

非常非正式，Apache ORC 可以理解为在文件类型领域中的 Arrow 对应物（尽管它的起源与 Arrow 没有关系）。更准确地说，它是一个开源的列式存储格式。

读取和写入的方式如下：

```py
from pyarrow import orc
# Write table
orc.write_table(t1, 't1.orc')
# Read table
t1 = orc.read_table('t1.orc')
```

作为旁注，我们可以在写入时通过使用“compression”参数来压缩文件。

### 3.2. CSV

没有什么秘密，pyarrow 有 CSV 模块：

```py
from pyarrow import csv
# Write CSV
csv.write_csv(t1, "t1.csv")
# Read CSV
t1 = csv.read_csv("t1.csv")

# Write CSV compressed and without header
options = csv.WriteOptions(include_header=False)
with pa.CompressedOutputStream("t1.csv.gz", "gzip") as out:
    csv.write_csv(t1, out, options)

# Read compressed CSV and add custom header
t1 = csv.read_csv("t1.csv.gz", read_options=csv.ReadOptions(
    column_names=["i", "x"], skip_rows=1
)]
```

## 3.2. JSON

Pyarrow 允许读取 JSON，但不允许写入。这很简单，让我们看看一个例子，假设我们的 JSON 数据在“data.json”中：

```py
from pyarrow import json
# Read json
fn = "data.json"
table = json.read_json(fn)

# We can now convert it to pandas if we want to
df = table.to_pandas()
```

Feather 是一种可移植的文件格式，用于存储 Arrow 表或数据帧（来自 Python 或 R 等语言），它内部使用 [Arrow IPC 格式](https://arrow.apache.org/docs/python/ipc.html#ipc)。因此，与 Apache ORC 不同，这个确实是在 Arrow 项目早期创建的。

```py
from pyarrow import feather
# Write feather from pandas DF
feather.write_feather(df, "t1.feather")
# Write feather from table, and compressed
feather.write_feather(t1, "t1.feather.lz4", compression="lz4")

# Read feather into table
t1 = feather.read_table("t1.feather")
# Read feather into df
df = feather.read_feather("t1.feather")
```

## 4. 高级功能

我们刚刚提到了与 Arrow 一起工作的最基本的功能以及大多数人需要的功能。然而，它的神奇之处并没有结束，它正是从这里开始的。

由于这将非常具有领域特定性，对任何人（也不被认为是入门级）都没有用处，我将只提及其中的一些功能，而不使用任何代码：

+   我们可以通过 **Buffer** 类型（建立在 C++ Buffer 对象之上）来处理内存管理。使用我们的数据创建一个缓冲区不会分配任何内存；它是对从数据字节对象导出的内存的零拷贝视图。继续这种内存管理，**MemoryPool** 的一个实例跟踪所有的分配和释放（类似于 C 中的 *malloc* 和 *free*）。这使我们能够跟踪分配的内存量。

+   类似地，有不同方式以批处理方式处理输入/输出流。

+   PyArrow 附带一个抽象文件系统接口，以及针对各种存储类型的具体实现。因此，例如，我们可以使用 S3FileSystem 从 S3 存储桶中写入和读取 parquet 文件。Google Cloud 和 Hadoop 分布式文件系统（HDFS）也被接受。

## 5. 结论和要点

Apache Arrow 是 Python 中高效数据处理的有力工具。它的列式存储格式、零拷贝读取以及与流行数据处理库的互操作性使其非常适合数据科学工作流程。通过将 Arrow 集成到您的管道中，您可以显著提高性能并优化内存使用。

## 6. 资源

+   [Apache Arrow 文档](https://arrow.apache.org/docs/)

+   [PyArrow GitHub 仓库](https://github.com/apache/arrow)

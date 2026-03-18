# 利用 NumPy 数组类型提示做更多：注释和验证形状与 `dtype`

> 原文：[`towardsdatascience.com/do-more-with-numpy-array-type-hints-annotate-validate-shape-dtype/`](https://towardsdatascience.com/do-more-with-numpy-array-type-hints-annotate-validate-shape-dtype/)

<mdspan datatext="el1748025684049" class="mdspan-comment">NumPy</mdspan> 数组对象可以采取许多具体形式。它可能是一个一维（1D）的布尔数组，或者是一个三维（3D）的 8 位无符号整数数组。正如内置函数 `isinstance()` 所显示的，每个数组都是 `np.ndarray` 的实例，无论形状或数组中存储的元素类型（即 `dtype`）如何。同样，许多类型注解的接口仍然只指定 `np.ndarray`：

```py
import numpy as np

def process(
    x: np.ndarray,
    y: np.ndarray,
    ) -> np.ndarray: ...
```

这样的类型注解是不够的：大多数接口对传入数组的形状或 `dtype` 有强烈的期望。如果期望一个 1D 数组而传入了一个 3D 数组，或者期望一个浮点数数组而传入了一个日期数组，大多数代码都会失败。

充分利用泛型 `np.ndarray`，现在可以完全指定数组形状和 `dtype` 特征：

```py
def process(
    x: np.ndarray[tuple[int], np.dtype[np.bool_]],
    y: np.ndarray[tuple[int, int, int], np.dtype[np.uint8]],
    ) -> np.ndarray[tuple[int], np.dtype[np.float64]]: ...
```

在这样的细节下，静态分析工具的最新版本，如 `mypy` 和 `pyright`，可以在代码运行之前找到问题。此外，针对 NumPy 专门的运行时验证器，如 [StaticFrame](https://github.com/static-frame/static-frame) 的 `sf.CallGuard`，可以重用相同的注解进行运行时验证。

## Python 中的泛型类型

通过指定每个接口所包含的类型，可以将泛型内置容器如 `list` 和 `dict` 具体化。一个函数可以声明它接受一个 `str` 类型的 `list`，用 `list[str]` 表示；或者可以指定一个 `str` 到 `bool` 的 `dict`，用 `dict[str, bool]` 表示。

## 泛型 `np.ndarray`

`np.ndarray` 是一个由单一元素类型（或 `dtype`）组成的 N 维数组。`np.ndarray` 泛型接受两个类型参数：第一个通过一个 `tuple` 定义形状，第二个通过泛型 `np.dtype` 定义元素类型。虽然 `np.ndarray` 已经使用了两个类型参数一段时间了，但第一个参数（形状）的定义直到 NumPy 2.1 才被完全指定。

### 形状类型参数

当使用 `np.empty` 或 `np.full` 等接口创建数组时，形状参数以一个 `tuple` 的形式给出。元组的长度定义了数组的维度；每个位置的大小定义了该维度的尺寸。因此，形状 `(10,)` 是一个包含 10 个元素的 1D 数组；形状 `(10, 100, 1000)` 是一个大小为 10x100x1000 的三维数组。

当在 `np.ndarray` 泛型中使用 `tuple` 来定义形状时，目前通常只能使用维数的数量进行类型检查。因此，`tuple[int]` 可以指定一维数组；`tuple[int, int, int]` 可以指定三维数组；`tuple[int, ...]`，指定零个或多个整数的元组，表示一个 N 维数组。未来可能能够对每个维度的特定大小进行类型检查（使用 `Literal`），但这目前还没有广泛支持。

### `dtype` 类型参数

NumPy 的 `dtype` 对象定义了元素类型，对于某些类型，还包括其他特性，例如大小（对于 Unicode 和字符串类型）或单位（对于 `np.datetime64` 类型）。`dtype` 本身是泛型的，它接受一个 NumPy “泛型”类型作为类型参数。最窄的类型指定了特定的元素特性，例如 `np.uint8`、`np.float64` 或 `np.bool_`。在这些窄类型之外，NumPy 提供了更通用的类型，如 `np.integer`、`np.inexact` 或 `np.number`。

## 使 `np.ndarray` 具体化

以下示例说明了具体的 `np.ndarray` 定义：

一个一维的布尔数组：

```py
np.ndarray[tuple[int], np.dtype[np.bool_]]
```

一个无符号 8 位整数的 3D 数组：

```py
np.ndarray[tuple[int, int, int], np.dtype[np.uint8]]
```

一个二维（2D）的 Unicode 字符串数组：

```py
np.ndarray[tuple[int, int], np.dtype[np.str_]]
```

任何数值类型的 1D 数组：

```py
np.ndarray[tuple[int], np.dtype[np.number]]
```

## 使用 Mypy 进行静态类型检查

一旦将泛型的 `np.ndarray` 具体化，`mypy` 或类似的类型检查器可以在某些代码路径中识别与接口不兼容的值。

例如，下面的函数需要一个一维的有符号整数数组。如下所示，无符号整数或维度不是一的数组将失败 `mypy` 检查。

```py
def process1(x: np.ndarray[tuple[int], np.dtype[np.signedinteger]]): ...

a1 = np.empty(100, dtype=np.int16)
process1(a1) # mypy passes

a2 = np.empty(100, dtype=np.uint8)
process1(a2) # mypy fails
# error: Argument 1 to "process1" has incompatible type
# "ndarray[tuple[int], dtype[unsignedinteger[_8Bit]]]";
# expected "ndarray[tuple[int], dtype[signedinteger[Any]]]"  [arg-type]

a3 = np.empty((100, 100, 100), dtype=np.int64)
process1(a3) # mypy fails
# error: Argument 1 to "process1" has incompatible type
# "ndarray[tuple[int, int, int], dtype[signedinteger[_64Bit]]]";
# expected "ndarray[tuple[int], dtype[signedinteger[Any]]]"
```

## 使用 `sf.CallGuard` 进行运行时验证

并非所有的数组操作都能静态地定义结果数组的形状或 `dtype`。因此，静态分析无法捕获所有不匹配的接口。相比于在许多函数中创建冗余的验证代码，类型注解可以被重复用于与 NumPy 类型专用的工具进行运行时验证。

[StaticFrame](https://github.com/static-frame/static-frame) 的 `CallGuard` 接口提供了两个装饰器，`check` 和 `warn`，它们在验证错误时分别抛出异常或警告。这些装饰器将验证类型注解与运行时对象的特性进行匹配。

例如，通过在下面的函数中添加 `sf.CallGuard.check`，数组在验证时将失败，并抛出具有表达性的 `CallGuard` 异常：

```py
import static_frame as sf

@sf.CallGuard.check
def process2(x: np.ndarray[tuple[int], np.dtype[np.signedinteger]]): ...

b1 = np.empty(100, dtype=np.uint8)
process2(b1)
# static_frame.core.type_clinic.ClinicError:
# In args of (x: ndarray[tuple[int], dtype[signedinteger]]) -> Any
# └── In arg x
#     └── ndarray[tuple[int], dtype[signedinteger]]
#         └── dtype[signedinteger]
#             └── Expected signedinteger, provided uint8 invalid

b2 = np.empty((10, 100), dtype=np.int8)
process2(b2)
# static_frame.core.type_clinic.ClinicError:
# In args of (x: ndarray[tuple[int], dtype[signedinteger]]) -> Any
# └── In arg x
#     └── ndarray[tuple[int], dtype[signedinteger]]
#         └── tuple[int]
#             └── Expected tuple length of 1, provided tuple length of 2
```

## 结论

可以做更多的事情来改进 NumPy 类型。例如，`np.object_` 类型可以被做成泛型，这样对象数组中包含的 Python 类型就可以被定义。例如，一个包含整数对的一维对象数组可以被注解为：

```py
np.ndarray[tuple[int], np.dtype[np.object_[tuple[int, int]]]]
```

此外，`np.datetime64` 的单位目前还不能静态指定。例如，可以使用类似 `np.dtype[np.datetime64[Literal['D']]]` 或 `np.dtype[np.datetime64[Literal['ns']]]` 的注解来区分日期单位和纳秒单位。

即使存在限制，完全指定的 NumPy 类型注解也能捕获错误并提高代码质量。如图所示，静态分析可以识别形状或 `dtype` 不匹配的情况，而使用 `sf.CallGuard` 进行验证则可以提供强大的运行时保证。

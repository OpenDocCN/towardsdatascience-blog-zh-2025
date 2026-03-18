# 让 Hypothesis 在用户之前破坏你的 Python 代码

> 原文：[`towardsdatascience.com/let-hypothesis-break-your-python-code-before-your-users-do/`](https://towardsdatascience.com/let-hypothesis-break-your-python-code-before-your-users-do/)

<mdspan datatext="el1761849745873" class="mdspan-comment">作为一名开发者</mdspan>，你应该认真对待代码测试。你可能使用 pytest 编写单元测试，模拟依赖关系，并努力实现高代码覆盖率。然而，如果你像我一样，在完成测试套件编写后，你可能在心中有一个挥之不去的问题。

**“我是否考虑了所有边缘情况？”**

你可能会用正数、负数、零和空字符串测试你的输入。但关于奇怪的 Unicode 字符呢？或者 NaN 或无穷大的浮点数？或者空字符串的列表或复杂的嵌套 JSON？可能的输入空间是巨大的，而且很难想到你的代码可能以无数种不同的方式出错，尤其是如果你在时间压力下。

基于属性的测试将这个负担从**你**转移到了工具。你不需要手动挑选例子，而是声明一个**属性**——一个对所有输入都必须成立的真理。然后，Hypothesis 库会**生成**输入；如果需要，几百个，寻找反例，如果找到，就会将其**缩小**到最简单且仍然导致失败的情况。

在这篇文章中，我将向你介绍基于属性的强大概念及其在 Hypothesis 中的实现。我们将超越简单的函数，向你展示如何测试复杂的数据结构和有状态的类，以及如何微调 Hypothesis 以实现稳健和高效的测试。

## 那么，基于属性的测试到底是什么？

基于属性的测试是一种方法，你不需要为特定的、硬编码的例子编写测试，而是定义你代码的“属性”或“不变性”。属性是对你代码行为的高层次陈述，应该对所有**所有**有效输入都成立。然后，你使用一个测试框架，如 Hypothesis，它智能地生成广泛范围的输入，并试图找到一个“反例”——一个你的声明属性为假的特定输入。

Hypothesis 在基于属性的测试中的几个关键方面包括：

+   **生成测试**。Hypothesis 为你生成测试用例，从简单的到不寻常的，探索你可能错过的边缘情况。

+   **属性驱动**。它改变了你的思维方式，从“对于这个特定输入的输出是什么？”转变为“关于我函数行为有哪些普遍真理？”

+   **缩小**。这是 Hypothesis 的杀手特性。当它找到一个失败的测试用例（可能很大且复杂）时，它不仅报告它，还会自动将输入“缩小”到最小且最简单的可能示例，这通常会使调试变得容易得多。

+   **有状态的测试**。Hypothesis 可以测试不仅仅是纯函数，还可以测试复杂对象在一系列方法调用中的交互和状态变化。

+   **可扩展的策略**。Hypothesis 提供了一个强大的“策略”库，用于生成数据，并允许你组合它们或构建全新的策略以匹配你的应用程序的数据模型。

## 为什么 Hypothesis 很重要 / 常见用例

基于属性的测试的主要优势是其发现细微错误并提高你对代码正确性的信心，这远远超出了仅使用基于示例的测试所能达到的程度。它迫使你更深入地思考代码的契约和假设。

Hypothesis 特别适用于测试：

+   **序列化/反序列化**。一个经典的属性是，对于任何对象 x，decode(encode(x)) 应该等于 x。这对于测试处理 JSON 或自定义二进制格式的函数来说非常完美。

+   **复杂的业务逻辑**。任何具有复杂条件逻辑的函数都是一个很好的候选者。Hypothesis 将会探索你的代码中你可能未曾考虑到的路径。

+   **有状态的系统**。测试类和对象以确保没有一系列有效操作可以将对象置于损坏或无效的状态。

+   **针对参考实现进行测试**。你可以声明你的新、优化函数应该始终产生与更简单、已知、典范的参考实现相同的结果。

+   **接受复杂数据模型的函数**。测试接受 Pydantic 模型、dataclasses 或其他自定义对象作为输入的函数。

## 设置开发环境

你只需要 Python 和 pip。我们将安装 pytest 作为我们的测试运行器，hypothesis 本身，以及 pydantic 用于我们的一个高级示例。

```py
(base) tom@tpr-desktop:~$ python -m venv hyp-env
(base) tom@tpr-desktop:~$ source hyp-env/bin/activate
(hyp-env) (base) tom@tpr-desktop:~$

# Install pytest, hypothesis, and pydantic
(hyp-env) (base) tom@tpr-desktop:~$ pip install pytest hypothesis pydantic 

# create a new folder to hold your python code
(hyp-env) (base) tom@tpr-desktop:~$ mkdir hyp-project
```

Hypothesis 最好通过使用像 pytest 这样的成熟测试运行工具来运行，所以这就是我们将要做的。

### 代码示例 1 — 一个简单的测试

在这个最简单的例子中，我们有一个计算矩形面积的函数。它应该接受两个大于零的整数参数，并返回它们的乘积。

Hypothesis 测试是通过两个东西定义的：**@given** 装饰器和 **策略**，它被传递给装饰器。将策略视为 Hypothesis 将生成以测试你的函数的数据类型。这里有一个简单的例子。首先，我们定义我们想要测试的函数。

```py
# my_geometry.py

def calculate_rectangle_area(length: int, width: int) -> int:
  """
  Calculates the area of a rectangle given its length and width.

  This function raises a ValueError if either dimension is not a positive integer.
  """
  if not isinstance(length, int) or not isinstance(width, int):
    raise TypeError("Length and width must be integers.")

  if length <= 0 or width <= 0:
    raise ValueError("Length and width must be positive.")

  return length * width
```

接下来是测试函数。

```py
# test_rectangle.py

from my_geometry import calculate_rectangle_area
from hypothesis import given, strategies as st
import pytest

# By using st.integers(min_value=1) for both arguments, we guarantee
# that Hypothesis will only generate valid inputs for our function.
@given(
    length=st.integers(min_value=1), 
    width=st.integers(min_value=1)
)
def test_rectangle_area_with_valid_inputs(length, width):
    """
    Property: For any positive integers length and width, the area
    should be equal to their product.

    This test ensures the core multiplication logic is correct.
    """
    print(f"Testing with valid inputs: length={length}, width={width}")

    # The property we are checking is the mathematical definition of area.
    assert calculate_rectangle_area(length, width) == length * width
```

将 **@given** 装饰器添加到函数中，将其转换为 Hypothesis 测试。将策略（st.integers）传递给装饰器表示 Hypothesis 应在测试时为参数 **n** 生成随机整数，但我们进一步约束确保这两个整数都不能小于一。

我们可以通过以下方式调用它来运行此测试。

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_my_geometry.py

=========================================== test session starts ============================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_my_geometry.py Testing with valid inputs: length=1, width=1
Testing with valid inputs: length=6541, width=1
Testing with valid inputs: length=6541, width=28545
Testing with valid inputs: length=1295885530, width=1
Testing with valid inputs: length=1295885530, width=25191
Testing with valid inputs: length=14538, width=1
Testing with valid inputs: length=14538, width=15503
Testing with valid inputs: length=7997, width=1
...
...

Testing with valid inputs: length=19378, width=22512
Testing with valid inputs: length=22512, width=22512
Testing with valid inputs: length=3392, width=44
Testing with valid inputs: length=44, width=44
.

============================================ 1 passed in 0.10s =============================================
```

默认情况下，Hypothesis 将对你的函数执行 100 次测试，使用不同的输入。你可以通过使用 **settings** 装饰器来增加或减少这个数量。例如，

```py
from hypothesis import given, strategies as st,settings
...
...
@given(
    length=st.integers(min_value=1), 
    width=st.integers(min_value=1)
)
@settings(max_examples=3)
def test_rectangle_area_with_valid_inputs(length, width):
...
...

#
# Outputs
#
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_my_geometry.py
=========================================== test session starts ============================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_my_geometry.py 
Testing with valid inputs: length=1, width=1
Testing with valid inputs: length=1870, width=5773964720159522347
Testing with valid inputs: length=61, width=25429
.

============================================ 1 passed in 0.06s =============================================
```

### 代码示例 2—测试经典的“往返”属性

让我们看看一个经典属性：序列化和反序列化应该是可逆的。简而言之，decode(encode(X)) 应该返回 X。

我们将编写一个函数，它接受一个字典并将其编码为 URL 查询字符串。

在你的 hyp-project 文件夹中创建一个名为 my_encoders.py 的文件。

```py
# my_encoders.py
import urllib.parse

def encode_dict_to_querystring(data: dict) -> str:
    # A bug exists here: it doesn't handle nested structures well
    return urllib.parse.urlencode(data)

def decode_querystring_to_dict(qs: str) -> dict:
    return dict(urllib.parse.parse_qsl(qs))
```

这些是两个基本函数。它们可能出什么问题？现在让我们在 test_encoders.py 中测试它们：

# test_encoders.py

```py
# test_encoders.py

from hypothesis import given, strategies as st

# A strategy for generating dictionaries with simple text keys and values
simple_dict_strategy = st.dictionaries(keys=st.text(), values=st.text())

@given(data=simple_dict_strategy)
def test_querystring_roundtrip(data):
    """Property: decoding an encoded dict should yield the original dict."""
    encoded = encode_dict_to_querystring(data)
    decoded = decode_querystring_to_dict(encoded)

    # We have to be careful with types: parse_qsl returns string values
    # So we convert our original values to strings for a fair comparison
    original_as_str = {k: str(v) for k, v in data.items()}

    assert decoded == original_as_st
```

现在我们可以运行我们的测试。

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_encoders.py
=========================================== test session starts ============================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_encoders.py F

================================================= FAILURES =================================================
_______________________________________ test_for_nesting_limitation ________________________________________

    @given(data=st.recursive(
>       # Base case: A flat dictionary of text keys and simple values (text or integers).
                   ^^^
        st.dictionaries(st.text(), st.integers() | st.text()),
        # Recursive step: Allow values to be dictionaries themselves.
        lambda children: st.dictionaries(st.text(), children)
    ))

test_encoders.py:7:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

data = {'': {}}

    @given(data=st.recursive(
        # Base case: A flat dictionary of text keys and simple values (text or integers).
        st.dictionaries(st.text(), st.integers() | st.text()),
        # Recursive step: Allow values to be dictionaries themselves.
        lambda children: st.dictionaries(st.text(), children)
    ))
    def test_for_nesting_limitation(data):
        """
        This test asserts that the decoded data structure matches the original.
        It will fail because urlencode flattens nested structures.
        """
        encoded = encode_dict_to_querystring(data)
        decoded = decode_querystring_to_dict(encoded)

        # This is a deliberately simple assertion. It will fail for nested
        # dictionaries because the `decoded` version will have a stringified
        # inner dict, while the `data` version will have a true inner dict.
        # This is how we reveal the bug.
>       assert decoded == data
E       AssertionError: assert {'': '{}'} == {'': {}}
E
E         Differing items:
E         {'': '{}'} != {'': {}}
E         Use -v to get more diff
E       Falsifying example: test_for_nesting_limitation(
E           data={'': {}},
E       )

test_encoders.py:24: AssertionError
========================================= short test summary info ==========================================
FAILED test_encoders.py::test_for_nesting_limitation - AssertionError: assert {'': '{}'} == {'': {}}
```

好吧，这出乎意料。让我们尝试分析这个测试出了什么问题。简而言之，这个测试表明 encode/decode 函数对于嵌套字典不正确地工作。

+   **错误示例**。最重要的线索在最下面。Hypothesis 告诉我们导致代码出错的 **确切输入**。

```py
test_for_nesting_limitation(
    data={'': {}},
)
```

+   输入是一个字典，其中键是一个空字符串，值是一个空字典。这是一个人类可能会忽略的经典边缘情况。

+   **断言错误**：测试失败是因为失败的断言语句：

```py
AssertionError: assert {'': '{}'} == {'': {}}
```

这是问题的核心。测试中使用的原始数据是 {‘’：{}}。你的函数输出的解码结果是 {‘’：‘{}’}。这表明对于键‘’，值是不同的：

+   在解码后的值中，值是 **字符串** ‘{}’。

+   在数据中，值是 **字典** {}。

字符串不等于字典，所以断言 **assert decoded == data** 是 **False**，测试失败。

## 逐步追踪错误

我们使用的 encode_dict_to_querystring 函数是 urllib.parse.urlencode。当 urlencode 遇到一个字典值（如 {}）时，它不知道如何处理它，所以它只是将其转换为它的字符串表示（‘{}’）。

关于值原始 *类型*（它是一个字典）的信息 **永远丢失**。

当 decode_querystring_to_dict 函数读取数据回时，它正确地将值解码为字符串‘{}’。它无法知道它最初是一个字典。

### 解决方案：将嵌套值编码为 JSON 字符串

解决方案很简单，

1.  **编码**。在 URL 编码之前，检查字典中的每个值。如果一个值是字典或列表，首先将其转换为 JSON 字符串。

1.  **解码**。在 URL 解码后，检查每个值。如果一个值看起来像 JSON 字符串（例如，以 { 或 [ 开头），将其解析回 Python 对象。

1.  **使我们的测试更全面**。我们的 **given** 装饰器更复杂。简单来说，它告诉 Hypothesis 生成可以包含其他字典作为值的字典，允许任何深度的嵌套数据结构。例如，

+   一个简单的、扁平的字典：{‘name’: ‘Alice’，‘city’: ‘London’}

+   一个单层嵌套的字典：{‘user’: {‘id’: ‘123’, ‘name’: ‘Tom’}}

+   一个两层嵌套的字典：{‘config’: {‘database’: {‘host’: ‘localhost’}}}

+   等等……

这里是修复后的代码。

```py
# test_encoders.py

from my_encoders import encode_dict_to_querystring, decode_querystring_to_dict
from hypothesis import given, strategies as st

# =========================================================================
# TEST 1: This test proves that the NESTING logic is correct.
# It uses a strategy that ONLY generates strings, so we don't have to
# worry about type conversion. This test will PASS.
# =========================================================================
@given(data=st.recursive(
    st.dictionaries(st.text(), st.text()),
    lambda children: st.dictionaries(st.text(), children)
))
def test_roundtrip_preserves_nested_structure(data):
    """Property: The encode/decode round-trip should preserve nested structures."""
    encoded = encode_dict_to_querystring(data)
    decoded = decode_querystring_to_dict(encoded)
    assert decoded == data

# =========================================================================
# TEST 2: This test proves that the TYPE CONVERSION logic is correct
# for simple, FLAT dictionaries. This test will also PASS.
# =========================================================================
@given(data=st.dictionaries(st.text(), st.integers() | st.text()))
def test_roundtrip_stringifies_simple_values(data):
    """
    Property: The round-trip should convert simple values (like ints)
    to strings.
    """
    encoded = encode_dict_to_querystring(data)
    decoded = decode_querystring_to_dict(encoded)

    # Create the model of what we expect: a dictionary with stringified values.
    expected_data = {k: str(v) for k, v in data.items()}
    assert decoded == expected_data
```

现在，如果我们重新运行我们的测试，我们得到这个，

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest
=========================================== test session starts ============================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_encoders.py .                                                                                   [100%]

============================================ 1 passed in 0.16s =============================================
```

我们在上面处理的是展示使用假设进行测试如何有用的经典例子。我们本以为有两个简单且无错误的函数，结果并非如此。

### 代码示例 3—为 Pydantic 模型构建自定义策略

许多现实世界的函数不仅仅接受简单的字典；它们接受结构化对象，如 Pydantic 模型。假设也可以为这些自定义类型构建策略。

让我们在 my_models.py 中定义一个模型。

```py
# my_models.py
from pydantic import BaseModel, Field
from typing import List

class Product(BaseModel):
    id: int = Field(gt=0)
    name: str = Field(min_length=1)
    tags: List[str]
def calculate_shipping_cost(product: Product, weight_kg: float) -> float:
    # A buggy shipping cost calculator
    cost = 10.0 + (weight_kg * 1.5)
    if "fragile" in product.tags:
        cost *= 1.5 # Extra cost for fragile items
    if weight_kg > 10:
        cost += 20 # Surcharge for heavy items
    # Bug: what if cost is negative?
    return cost
```

现在，在 test_shipping.py 中，我们将构建一个策略来生成 Product 实例并测试我们的有误函数。

```py
# test_shipping.py
from my_models import Product, calculate_shipping_cost
from hypothesis import given, strategies as st

# Build a strategy for our Product model
product_strategy = st.builds(
    Product,
    id=st.integers(min_value=1),
    name=st.text(min_size=1),
    tags=st.lists(st.sampled_from(["electronics", "books", "fragile", "clothing"]))
)
@given(
    product=product_strategy,
    weight_kg=st.floats(min_value=-10, max_value=100, allow_nan=False, allow_infinity=False)
)
def test_shipping_cost_is_always_positive(product, weight_kg):
    """Property: The shipping cost should never be negative."""
    cost = calculate_shipping_cost(product, weight_kg)
    assert cost >= 0
```

那么测试输出呢？

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_shipping.py
========================================================= test session starts ==========================================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_shipping.py F

=============================================================== FAILURES ===============================================================
________________________________________________ test_shipping_cost_is_always_positive _________________________________________________

    @given(
>       product=product_strategy,
                   ^^^
        weight_kg=st.floats(min_value=-10, max_value=100, allow_nan=False, allow_infinity=False)
    )

test_shipping.py:13:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

product = Product(id=1, name='0', tags=[]), weight_kg = -7.0

    @given(
        product=product_strategy,
        weight_kg=st.floats(min_value=-10, max_value=100, allow_nan=False, allow_infinity=False)
    )
    def test_shipping_cost_is_always_positive(product, weight_kg):
        """Property: The shipping cost should never be negative."""
        cost = calculate_shipping_cost(product, weight_kg)
>       assert cost >= 0
E       assert -0.5 >= 0
E       Falsifying example: test_shipping_cost_is_always_positive(
E           product=Product(id=1, name='0', tags=[]),
E           weight_kg=-7.0,
E       )

test_shipping.py:19: AssertionError
======================================================= short test summary info ========================================================
FAILED test_shipping.py::test_shipping_cost_is_always_positive - assert -0.5 >= 0
========================================================== 1 failed in 0.12s ===========================================================
```

当你使用 pytest 运行时，假设会迅速找到一个反例：一个重量 _kg 为负数的商品可能导致负的运费。这是我们可能没有考虑到的边缘情况，但假设自动找到了它。

### 代码示例 4—测试有状态的类

假设不仅可以测试纯函数，还可以通过生成方法调用序列来测试具有内部状态的类，以尝试破坏它们。让我们测试一个简单的自定义 LimitedCache 类。

my_cache.py

```py
# my_cache.py
class LimitedCache:
    def __init__(self, capacity: int):
        if capacity <= 0:
            raise ValueError("Capacity must be positive")
        self._cache = {}
        self._capacity = capacity
        # Bug: This should probably be a deque or ordered dict for proper LRU
        self._keys_in_order = []

    def put(self, key, value):
        if key not in self._cache and len(self._cache) >= self._capacity:
            # Evict the oldest item
            key_to_evict = self._keys_in_order.pop(0)
            del self._cache[key_to_evict]

        if key not in self._keys_in_order:
            self._keys_in_order.append(key)
        self._cache[key] = value

    def get(self, key):
        return self._cache.get(key)

    @property
    def size(self):
        return len(self._cache)
```

这个缓存有几个与其驱逐策略相关的潜在错误。让我们使用假设基于规则的有限状态机来测试它，该状态机旨在通过生成随机的方法调用序列来测试具有内部状态的对象，以识别仅在特定交互后出现的错误。

创建文件 test_cache.py。

```py
from hypothesis import strategies as st
from hypothesis.stateful import RuleBasedStateMachine, rule, precondition
from my_cache import LimitedCache

class CacheMachine(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.cache = LimitedCache(capacity=3)

    # This rule adds 3 initial items to fill the cache
    @rule(
        k1=st.just('a'), k2=st.just('b'), k3=st.just('c'),
        v1=st.integers(), v2=st.integers(), v3=st.integers()
    )
    def fill_cache(self, k1, v1, k2, v2, k3, v3):
        self.cache.put(k1, v1)
        self.cache.put(k2, v2)
        self.cache.put(k3, v3)

    # This rule can only run AFTER the cache has been filled.
    # It tests the core logic of LRU vs FIFO.
    @precondition(lambda self: self.cache.size == 3)
    @rule()
    def test_update_behavior(self):
        """
        Property: Updating the oldest item ('a') should make it the newest,
        so the next eviction should remove the second-oldest item ('b').
        Our buggy FIFO cache will incorrectly remove 'a' anyway.
        """
        # At this point, keys_in_order is ['a', 'b', 'c'].
        # 'a' is the oldest.

        # We "use" 'a' again by updating it. In a proper LRU cache,
        # this would make 'a' the most recently used item.
        self.cache.put('a', 999) 

        # Now, we add a new key, which should force an eviction.
        self.cache.put('d', 4)

        # A correct LRU cache would evict 'b'.
        # Our buggy FIFO cache will evict 'a'.
        # This assertion checks the state of 'a'.
        # In our buggy cache, get('a') will be None, so this will fail.
        assert self.cache.get('a') is not None, "Item 'a' was incorrectly evicted"

# This tells pytest to run the state machine test
TestCache = CacheMachine.TestCase
```

假设会生成一系列的 put 和 get 操作。它会迅速识别出一系列 put 操作，导致缓存的大小超过其容量或其驱逐行为与我们的模型不同，从而揭示我们实现中的错误。

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_cache.py
========================================================= test session starts ==========================================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_cache.py F

=============================================================== FAILURES ===============================================================
__________________________________________________________ TestCache.runTest ___________________________________________________________

self = <hypothesis.stateful.CacheMachine.TestCase testMethod=runTest>

    def runTest(self):
>       run_state_machine_as_test(cls, settings=self.settings)

../hyp-env/lib/python3.11/site-packages/hypothesis/stateful.py:476:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
../hyp-env/lib/python3.11/site-packages/hypothesis/stateful.py:258: in run_state_machine_as_test
    state_machine_test(state_machine_factory)
../hyp-env/lib/python3.11/site-packages/hypothesis/stateful.py:115: in run_state_machine
    @given(st.data())
               ^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

self = CacheMachine({})

    @precondition(lambda self: self.cache.size == 3)
    @rule()
    def test_update_behavior(self):
        """
        Property: Updating the oldest item ('a') should make it the newest,
        so the next eviction should remove the second-oldest item ('b').
        Our buggy FIFO cache will incorrectly remove 'a' anyway.
        """
        # At this point, keys_in_order is ['a', 'b', 'c'].
        # 'a' is the oldest.

        # We "use" 'a' again by updating it. In a proper LRU cache,
        # this would make 'a' the most recently used item.
        self.cache.put('a', 999)

        # Now, we add a new key, which should force an eviction.
        self.cache.put('d', 4)

        # A correct LRU cache would evict 'b'.
        # Our buggy FIFO cache will evict 'a'.
        # This assertion checks the state of 'a'.
        # In our buggy cache, get('a') will be None, so this will fail.
>       assert self.cache.get('a') is not None, "Item 'a' was incorrectly evicted"
E       AssertionError: Item 'a' was incorrectly evicted
E       assert None is not None
E        +  where None = get('a')
E        +    where get = <my_cache.LimitedCache object at 0x7f0debd1da90>.get
E        +      where <my_cache.LimitedCache object at 0x7f0debd1da90> = CacheMachine({}).cache
E       Falsifying example:
E       state = CacheMachine()
E       state.fill_cache(k1='a', k2='b', k3='c', v1=0, v2=0, v3=0)
E       state.test_update_behavior()
E       state.teardown()

test_cache.py:44: AssertionError
======================================================= short test summary info ========================================================
FAILED test_cache.py::TestCache::runTest - AssertionError: Item 'a' was incorrectly evicted
========================================================== 1 failed in 0.20s ===========================================================
```

上述输出突显了代码中的一个错误。简单来说，这个输出显示缓存不是一个合适的**“最近最少使用” (LRU)** 缓存**。**它有以下显著的缺陷，

> *当你更新缓存中已经存在的项时，缓存未能记住它现在是“最新”的项。它仍然将其视为最旧的项，因此它会被提前（提前）从缓存中移除。*

### 代码示例 5—针对更简单的参考实现进行测试

在我们的最后一个例子中，我们将探讨一个典型情况。通常，程序员编写函数以替换旧的、较慢但其他方面完全正确的函数。你的新函数必须对相同的输入有与旧函数相同的输出。假设可以让你在这方面进行测试变得更加容易。

假设我们有一个简单的函数，sum_list_simple，以及一个新编写的、**“优化”**的 sum_list_fast 函数存在一个错误。

**my_sums.py**

```py
# my_sums.py
def sum_list_simple(data: list[int]) -> int:
    # This is our simple, correct reference implementation
    return sum(data)

def sum_list_fast(data: list[int]) -> int:
    # A new "fast" implementation with a bug (e.g., integer overflow for large numbers)
    # or in this case, a simple mistake.
    total = 0
    for x in data:
        # Bug: This should be +=
        total = x
    return total
```

**test_sums.py**

```py
# test_sums.py
from my_sums import sum_list_simple, sum_list_fast
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_fast_sum_matches_simple_sum(data):
    """
    Property: The result of the new, fast function should always match
    the result of the simple, reference function.
    """
    assert sum_list_fast(data) == sum_list_simple(data)
```

假设会迅速发现对于任何包含多于一个元素的列表，新函数会失败。让我们来看看。

```py
(hyp-env) (base) tom@tpr-desktop:~/hypothesis_project$ pytest -s test_my_sums.py
=========================================== test session starts ============================================
platform linux -- Python 3.11.10, pytest-8.4.0, pluggy-1.6.0
rootdir: /home/tom/hypothesis_project
plugins: hypothesis-6.135.9, anyio-4.9.0
collected 1 item

test_my_sums.py F

================================================= FAILURES =================================================
_____________________________________ test_fast_sum_matches_simple_sum _____________________________________

    @given(st.lists(st.integers()))
>   def test_fast_sum_matches_simple_sum(data):
                   ^^^

test_my_sums.py:6:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

data = [1, 0]

    @given(st.lists(st.integers()))
    def test_fast_sum_matches_simple_sum(data):
        """
        Property: The result of the new, fast function should always match
        the result of the simple, reference function.
        """
>       assert sum_list_fast(data) == sum_list_simple(data)
E       assert 0 == 1
E        +  where 0 = sum_list_fast([1, 0])
E        +  and   1 = sum_list_simple([1, 0])
E       Falsifying example: test_fast_sum_matches_simple_sum(
E           data=[1, 0],
E       )

test_my_sums.py:11: AssertionError
========================================= short test summary info ==========================================
FAILED test_my_sums.py::test_fast_sum_matches_simple_sum - assert 0 == 1
============================================ 1 failed in 0.17s =============================================
```

因此，测试失败是因为**“快速”**求和函数对于输入列表 [1, 0] 给出了错误的答案（0），而正确的答案，由**“简单”**求和函数提供，是 1。现在你知道了问题，你可以采取措施来修复它。

## 摘要

在这篇文章中，我们深入探讨了 Hypothesis 的基于属性的测试世界，不仅限于简单的例子，还展示了它如何应用于现实世界的测试挑战。我们了解到，通过定义我们代码的不变量，我们可以揭示传统测试可能错过的微妙错误。我们学习了如何：

+   测试“往返”属性，看看更复杂的数据策略如何揭示我们代码中的局限性。

+   构建自定义策略来生成用于测试业务逻辑的复杂 Pydantic 模型的实例。

+   使用 RuleBasedStateMachine 通过生成方法调用序列来测试有状态类的行为。

+   通过与更简单、已知良好的参考实现进行测试来验证复杂的、优化的函数。

在你的工具箱中添加基于属性的测试不会取代你现有的所有测试。然而，它将极大地增强它们，迫使你更清晰地思考代码的契约，并给你在正确性方面的信心大大提高。我鼓励你选择代码库中的一个函数或类，思考其基本属性，并让 Hypothesis 尽力证明你是错的。这将使你成为一个更好的开发者。

我只是触及了 Hypothesis 为你的测试所能做的表面功夫。更多信息，请参考他们官方的文档，可通过以下链接获取。

[`hypothesis.readthedocs.io/en/latest`](https://hypothesis.readthedocs.io/en/latest)

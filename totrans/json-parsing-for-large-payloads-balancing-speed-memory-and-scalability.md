# 大负载 JSON 解析：平衡速度、内存和可扩展性

> 原文：[`towardsdatascience.com/json-parsing-for-large-payloads-balancing-speed-memory-and-scalability/`](https://towardsdatascience.com/json-parsing-for-large-payloads-balancing-speed-memory-and-scalability/)

## 简介

<mdspan datatext="el1764656085968" class="mdspan-comment">想象一下，你为黑色星期五设置的营销活动取得了巨大成功，顾客开始涌入你的网站。你通常每小时有大约 1000 个客户事件的 Mixpanel 设置，结果在一小时内就产生了数百万个客户事件。因此，你的数据管道现在需要解析大量 JSON 数据并将其存储到数据库中。你发现你的标准 JSON 解析库无法扩展到突然的数据增长，你的近实时分析报告落后了。这时，你意识到一个高效的 JSON 解析库的重要性。除了处理大量负载外，JSON 解析库还应该能够序列化和反序列化高度嵌套的 JSON 负载。

在本文中，我们探讨了用于大负载的 Python 解析库。我们特别关注了 ujson、orjson 和 ijson 的功能。然后，我们对标准 JSON 库（stdlib/json）、ujson 和 orjson 的序列化和反序列化性能进行了基准测试。由于我们在整篇文章中使用了序列化和反序列化这两个术语，以下是对这些概念的重温。序列化涉及将你的 Python 对象转换为 JSON 字符串，而反序列化涉及从你的 Python 数据结构重建 JSON 字符串。

随着文章的推进，你将找到一个决策流程图，帮助你根据你的工作流程和独特的解析需求选择解析器。此外，我们还探讨了 NDJSON 及其解析 NDJSON 负载的库。让我们开始吧。

### Stdlib JSON

Stdlib JSON 支持所有基本 Python 数据类型的序列化，包括 dicts、lists 和 tuples。当调用 json.loads()函数时，它会一次性将整个 JSON 加载到内存中。这对于较小的负载来说是可行的，但对于较大的负载，json.loads()可能会导致关键性能问题，如内存不足错误和下游工作流程的阻塞。

```py
import json

with open("large_payload.json", "r") as f:
    json_data = json.loads(f)   #loads entire file into memory, all tokens at once
```

### ijson

对于数百兆字节的负载，建议使用 ijson。ijson，即“迭代 JSON”，一次读取一个标记，没有内存开销。在下面的代码中，我们比较了 json 和 ijson。

```py
#The ijson library reads records one token at a time
import ijson
with open("json_data.json", "r") as f:
    for record in ijson.items(f, "items.item"): #fetch one dict from the array
       process(record) 
```

如您所见，ijson 一次从 JSON 中获取一个元素，并将其加载到一个 Python 字典对象中。然后将其传递给调用函数，在这种情况下，是 process(record)函数。ijson 的整体工作原理已在下面的插图提供。

![](img/a8c40f4faa375e2cf6ddf52a530c5fc2.png)

ijson 的高级说明（图片由作者提供）

### ujson

![](img/7868117148fdaf9971e1e35319318c43.png)

Ujson – 内部结构（作者图片）

Ujson 在许多涉及大量 JSON 负载的应用程序中已被广泛使用，因为它被设计为 Python 中 stdlib JSON 的更快替代品。解析速度非常快，因为 ujson 的底层代码是用 C 编写的，并通过 Python 绑定连接到 Python 接口。在标准 JSON 库中需要改进的领域在 Ujson 中进行了优化，以提高速度和性能。但是，正如制作者在 PyPI 上提到的，Ujson 已不再用于新的项目，该库已被置于仅维护模式。以下是对 ujson 高级流程的说明。

```py
import ujson
taxonomy_data = '{"id":1, "genus":"Thylacinus", "species":"cynocephalus", "extinct": true}'
data_dict = ujson.loads(taxonomy_data) #Deserialize

with open("taxonomy_data.json", "w") as fh: #Serialize
    ujson.dump(data_dict, fh) 

with open("taxonomy_data.json", "r") as fh: #Deserialize
    data = ujson.load(fh)
    print(data)
```

我们转向下一个潜在的库，名为“orjson”。

### orjson

由于 Orjson 是用 Rust 编写的，它不仅优化了速度，还具备内存安全机制，以防止开发者在使用类似 ujson 这样的基于 C 的 JSON 库时遇到的缓冲区溢出问题。此外，Orjson 支持序列化多种额外的数据类型，超出了标准的 Python 数据类型，包括 dataclass 和 datetime 对象。与其它库相比，orjson 的另一个关键区别在于，orjson 的 dumps() 函数返回一个字节对象，而其它库返回一个字符串。以字节对象的形式返回数据是 orjson 高吞吐量主要原因之一。

```py
import orjson
book_payload = '{"id":1,"name":"The Great Gatsby","author":"F. Scott Fitzgerald","Publishing House":"Charles Scribner\'s Sons"}'
data_dict = orjson.loads(book_payload) #Deserialize
print(data_dict)          

with open("book_data.json", "wb") as f: #Serialize
    f.write(orjson.dumps(data_dict)) #Returns bytes object

with open("book_data.json", "rb") as f:#Deserialize
    book_data = orjson.loads(f.read())
    print(book_data)
```

既然我们已经探索了一些 JSON 解析库，让我们测试它们的序列化能力。

### 测试 JSON、ujson 和 orjson 的序列化能力

我们创建了一个包含整数、字符串和 datetime 变量的样本 dataclass 对象。

```py
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: int
    name: str
    created: datetime

u = User(id=1, name="Thomas", created=datetime.now())
```

然后，我们将它们传递给每个库以查看会发生什么。我们首先从 stdlib JSON 开始。

```py
import json
try:
    print("json:", json.dumps(u))
except TypeError as e:
    print("json error:", e)
```

如预期，我们得到了以下错误。（标准 JSON 库不支持序列化“dataclass”对象和 datetime 对象。）

![](img/33784eab7da3ce19918725082660a746.png)

接下来，我们使用 ujson 库进行相同的测试。

```py
import ujson
try:
print("json:", ujson.dumps(u))
except TypeError as e:
print("json error:", e)
```

![](img/058d62a9c2da91b35016d0f0710b6341.png)

如上图所示，ujson 无法序列化数据类对象和 datetime 数据类型。最后，我们使用 orjson 库进行序列化。

```py
import orjson
try:
    print("orjson:", orjson.dumps(u))
except TypeError as e:
    print("orjson error:", e)
```

我们看到 orjson 能够序列化数据类和 datetime 数据类型。

![](img/cc37edcf5dfeb826fa3f46b10735d02e.png)

### 与 NDJSON 一起工作（特别说明）

我们已经看到了 JSON 解析库，那么 NDJSON 呢？NDJSON（Newline Delimited JSON），正如你可能知道的，是一种格式，其中每一行都是一个 JSON 对象。换句话说，分隔符不是逗号，而是换行符。以下是一个 NDJSON 的例子。

```py
{"id": "A13434", "name": "Ella"}
{"id": "A13455", "name": "Charmont"}
{"id": "B32434", "name": "Areida"}
```

NDJSON 主要用于日志和流数据，因此，NDJSON 负载数据非常适合使用 ijson 库进行解析。对于小型到中型的 NDJSON 负载数据，建议使用 stdlib JSON。除了 ijson 和 stdlib JSON 之外，还有一个专门的 NDJSON 库。以下是一些展示每种方法的代码片段。

#### 使用 stdlib JSON 和 ijson 的 NDJSON

由于 NDJSON 不以逗号分隔，它不符合批量加载的条件，因为 stdlib json 期望看到字典列表。换句话说，stdlib JSON 的解析器寻找单个有效的 JSON 元素，但给定的有效载荷文件中有多个 JSON 元素。因此，必须逐行迭代解析文件，并将数据发送到调用函数进行进一步处理。

```py
import json
ndjson_payload = """{"id": "A13434", "name": "Ella"}
{"id": "A13455", "name": "Charmont"}
{"id": "B32434", "name": "Areida"}"""

#Writing NDJSON file
with open("json_lib.ndjson", "w", encoding="utf-8") as fh:
    for line in ndjson_payload.splitlines(): #Split string into JSON obj
        fh.write(line.strip() + "\n") #Write each JSON object as its line

#Reading NDJSON file using json.loads
with open("json_lib.ndjson", "r", encoding="utf-8") as fh:
    for line in fh:
        if line.strip():                       #Remove new lines
            item= json.loads(line)             #Deserialize
            print(item) #or send it to the caller function
```

使用 ijson，解析过程如下所示。使用标准 JSON，我们只有一个根元素，如果是单个 JSON，则是一个字典；如果是字典列表，则是一个数组。但使用 NDJSON，每一行都是其自己的根元素。ijson.items()中的“”参数告诉 ijson 解析器查看每个根元素。参数“”和 multiple_values=True 让 ijson 解析器知道文件中有多个 JSON 根元素，并且一次获取一行（每个 JSON）。

```py
import ijson
ndjson_payload = """{"id": "A13434", "name": "Ella"}
{"id": "A13455", "name": "Charmont"}
{"id": "B32434", "name": "Areida"}"""

#Writing the payload to a file to be processed by ijson
with open("ijson_lib.ndjson", "w", encoding="utf-8") as fh:
    fh.write(ndjson_payload)

with open("ijson_lib.ndjson", "r", encoding="utf-8") as fh:
    for item in ijson.items(fh, "", multiple_values=True):
        print(item)
```

最后，我们有专门的 NDJSON 库。它基本上是将 NDJSON 格式转换为标准 JSON。

```py
import ndjson
ndjson_payload = """{"id": "A13434", "name": "Ella"}
{"id": "A13455", "name": "Charmont"}
{"id": "B32434", "name": "Areida"}"""

#writing the payload to a file to be processed by ijson
with open("ndjson_lib.ndjson", "w", encoding="utf-8") as fh:
    fh.write(ndjson_payload)

with open("ndjson_lib.ndjson", "r", encoding="utf-8") as fh:
    ndjson_data = ndjson.load(fh)   #returns a list of dicts
```

正如你所见，NDJSON 文件格式通常可以使用 stdlib json 和 ijson 进行解析。对于非常大的有效载荷，ijson 是最佳选择，因为它内存高效。但如果你想要从其他 Python 对象生成 NDJSON 有效载荷，NDJSON 库是理想的选择。这是因为 ndjson.dumps()函数会自动将 Python 对象转换为 NDJSON 格式，而无需遍历这些数据结构。

现在我们已经探讨了 NDJSON，让我们回到基准测试 stdlib json、ujson 和 orjson 库。

### 我不认为 ijson 适合用于基准测试

由于‘ijson’是一个流式解析器，这与我们所查看的批量解析器非常不同。如果我们与这些批量解析器一起基准测试 ijson，我们就是在比较苹果和橘子。即使我们与其他解析器一起基准测试 ijson，我们也会产生一种错误的印象，即 ijson 是最慢的，而实际上它完全服务于不同的目的。ijson 针对内存效率进行了优化，因此其吞吐量低于批量解析器。

## 为基准测试目的生成合成 JSON 有效载荷

我们使用库‘mimesis’生成一个包含 100 万条记录的大型合成 JSON 有效载荷，用于基准测试。这些数据将被用于基准测试。以下代码可以用于创建此基准测试的有效载荷，如果你希望复制此操作。生成的文件大小将在 100 MB 到 150 MB 之间，我认为这足够大，可以用于基准测试。

```py
from mimesis import Person, Address
import json
person_name = Person("en")
complete_address = Address("en")

#streaming to a file
with open("large_payload.json", "w") as fh:
    fh.write("[")  #JSON array
    for i in range(1_000_000):
        payload = {
            "id": person_name.identifier(),
            "name": person_name.full_name(),
            "email": person_name.email(),
            "address": {
                "street": complete_address.street_name(),
                "city": complete_address.city(),
                "postal_code": complete_address.postal_code()
            }
        }
        json.dump(payload, fh)
        if i < 999_999: #To prevent a comma at the last entry
            fh.write(",") 
    fh.write("]")   #end JSON array
```

下面是生成数据的示例。如你所见，地址字段是嵌套的，以确保 JSON 不仅体积大，而且代表真实世界的分层 JSON。

```py
[
  {
    "id": "8177",
    "name": "Willia Hays",
    "email": "[[email protected]](/cdn-cgi/l/email-protection)",
    "address": {
      "street": "Emerald Cove",
      "city": "Crown Point",
      "postal_code": "58293"
    }
  },
  {
    "id": "5931",
    "name": "Quinn Greer",
    "email": "[[email protected]](/cdn-cgi/l/email-protection)",
    "address": {
      "street": "Ohlone",
      "city": "Bridgeport",
      "postal_code": "92982"
    }
  }
]
```

让我们从基准测试开始。

### 基准测试先决条件

我们使用 read()函数将 JSON 文件存储为字符串。然后，我们使用每个库（json、ujson 和 orjson）中的 loads()函数将 JSON 字符串反序列化为 Python 对象。首先，我们从原始 JSON 文本创建 payload_str 对象。

```py
with open("large_payload1.json", "r") as fh:
    payload_str = fh.read()   #raw JSON text
```

我们创建了一个带有两个参数的基准测试函数。第一个参数是要测试的函数。在这种情况下，它是 loads()函数。第二个参数是从上述文件构建的 payload_str。

```py
def benchmark_load(func, payload_str):
    start = time.perf_counter()
    for _ in range(3):
        func(payload_str)
    end = time.perf_counter()
    return end - start
```

我们使用上述函数来测试序列化和反序列化的速度。

### 反序列化速度基准测试

我们加载正在测试的三个库。然后，我们对每个库的 loads()函数运行 benchmark_load()函数。

```py
import json, ujson, orjson, time

results = {
    "json.loads": benchmark_load(json.loads, payload_str),
    "ujson.loads": benchmark_load(ujson.loads, payload_str),
    "orjson.loads": benchmark_load(orjson.loads, payload_str),
}

for lib, t in results.items():
    print(f"{lib}: {t:.4f} seconds")
```

如我们所见，orjson 在反序列化方面花费的时间最少。

![图片](img/fb71bcecee73d093ffd6f0a789323dda.png)

### 序列化速度基准测试

接下来，我们测试这些库的序列化速度。

```py
import json
import ujson
import orjson
import time

results = {
    "json.dumps": benchmark("json", json.dumps, payload_str),
    "ujson.dumps": benchmark("ujson", ujson.dumps, payload_str),
    "orjson.dumps": benchmark("orjson", orjson.dumps, payload_str),
}

for lib, t in results.items():
    print(f"{lib}: {t:.4f} seconds")
```

在比较运行时间时，我们看到 orjson 将 Python 对象序列化为 JSON 对象所需的时间最少。

![图片](img/a6a60c5c1fc49bc6d3dfbfc1367422e8.png)

## 选择适合您工作流程的最佳 JSON 库

![图片](img/a62ad5b3fd945d6a007e53e3616c2d9a.png)

选择最佳 JSON 库的指南（图片由作者提供）

### JSON 的剪贴板与工作流程技巧

假设你想要在记事本++或类似文本编辑器中查看你的 JSON，或者与团队成员在 Slack 上分享一个片段（来自大型负载），你很快就会遇到剪贴板或文本编辑器/IDE 崩溃的问题。在这种情况下，可以使用 Pyperclip 或 Tkinter。Pyperclip 适用于 50 MB 以内的负载，而 Tkinter 适用于中等大小的负载。对于大型负载，可以将 JSON 写入文件以查看数据。

## 结论

JSON 看起来可能毫不费力，但随着负载的增大和嵌套的增多，这些负载可能会迅速变成性能瓶颈。本文旨在强调每个 Python 解析库如何应对这一挑战。在选择 JSON 解析库时，速度和吞吐量并不总是主要标准。是工作流程决定了是否需要吞吐量、内存效率或长期可扩展性来解析负载。简而言之，JSON 解析不应是一劳永逸的方法。

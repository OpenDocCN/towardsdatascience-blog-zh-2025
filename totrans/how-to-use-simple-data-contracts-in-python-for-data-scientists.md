# 如何在 Python 中使用简单的数据协议为数据科学家服务

> 原文：[`towardsdatascience.com/how-to-use-simple-data-contracts-in-python-for-data-scientists/`](https://towardsdatascience.com/how-to-use-simple-data-contracts-in-python-for-data-scientists/)

## <mdspan datatext="el1764654507571" class="mdspan-comment">问题所在</mdspan>

诚实地讲：我们都有过这样的经历。

今天下午是星期五。你已经训练了一个模型，验证了它，并部署了推理管道。指标看起来一切正常。你关上笔记本电脑，享受周末的休息。

星期一早上，当你上班时，你看到消息**“管道失败”**。发生了什么事？当你部署推理管道时，一切都很完美。

事实上，问题可能是许多事情之一。也许上游工程团队将`user_id`列从整数更改为字符串。或者也许`price`列突然包含负数。或者我个人最喜欢的：列名从`created_at`更改为`createdAt`（驼峰命名法再次出现！）。

行业称之为**模式漂移**。我叫它头疼。

最近，人们一直在谈论很多关于**数据协议**的事情。通常，这涉及到向你推销一个昂贵的 SaaS 平台或复杂的微服务架构。但如果你只是一个试图防止 Python 管道爆炸的数据科学家或工程师，你并不一定需要企业级冗余。

* * *

## 工具：Pandera

让我们来看看如何使用**Pandera**库在 Python 中创建一个简单的数据协议。这是一个开源的 Python 库，允许你将模式定义为类对象。如果你使用过 FastAPI，它感觉非常类似 Pydantic，但它专门为 DataFrames 构建。

要开始，你可以简单地使用 pip 安装`pandera`：

```py
pip install pandera
```

* * *

## 一个真实案例：市场营销线索源

让我们看看一个经典的场景。你正在从第三方供应商那里摄取营销线索的 CSV 文件。

这是我们**期望**数据看起来像什么：

1.  **id**：一个整数（必须是唯一的）。

1.  **email**：一个字符串（实际上必须看起来像电子邮件）。

1.  **signup_date**：一个有效的 datetime 对象。

1.  **lead_score**：介于 0.0 和 1.0 之间的浮点数。

这里是我们接收到的原始数据的混乱现实：

```py
import pandas as pd
import numpy as np

# Simulating incoming data that MIGHT break our pipeline
data = {
    "id": [101, 102, 103, 104],
    "email": ["[[email protected]](/cdn-cgi/l/email-protection)", "[[email protected]](/cdn-cgi/l/email-protection)", "INVALID_EMAIL", "[[email protected]](/cdn-cgi/l/email-protection)"],
    "signup_date": ["2024-01-01", "2024-01-02", "2024-01-03", "2024-01-04"],
    "lead_score": [0.5, 0.8, 1.5, -0.1] # Note: 1.5 and -0.1 are out of bounds!
}

df = pd.DataFrame(data)
```

如果你将这个数据框输入到一个期望分数在 0 到 1 之间的模型中，你的预测将是垃圾。如果你尝试在`id`上连接并存在重复项，你的行计数将会爆炸。混乱的数据会导致混乱的数据科学！

### 第一步：定义协议

而不是写上一打`if`语句来检查数据质量，我们定义了一个**SchemaModel**。这是我们之间的协议。

```py
import pandera as pa
from pandera.typing import Series

class LeadsContract(pa.SchemaModel):
    # 1\. Check data types and existence
    id: Series[int] = pa.Field(unique=True, ge=0) 

    # 2\. Check formatting using regex
    email: Series[str] = pa.Field(str_matches=r"[^@]+@[^@]+\.[^@]+")

    # 3\. Coerce types (convert string dates to datetime objects automatically)
    signup_date: Series[pd.Timestamp] = pa.Field(coerce=True)

    # 4\. Check business logic (bounds)
    lead_score: Series[float] = pa.Field(ge=0.0, le=1.0)

    class Config:
        # This ensures strictness: if an extra column appears, or one is missing, throw an error.
        strict = True
```

查看上面的代码，以了解 Pandera 如何设置一个协议。你可以稍后查看 Pandera 文档时再关注细节。

### 第二步：强制执行协议

现在，我们需要将我们制定的合同应用到我们的数据上。一种简单的方法是运行`LeadsContract.validate(df)`。这可以工作，但会在找到的第一个错误处崩溃。在生产环境中，你通常希望知道文件中所有的问题，而不仅仅是第一行。

我们可以启用“懒惰”验证，一次性捕获所有错误。

```py
try:
    # lazy=True means "find all errors before crashing"
    validated_df = LeadsContract.validate(df, lazy=True)
    print("Data passed validation! Proceeding to ETL...")

except pa.errors.SchemaErrors as err:
    print("⚠️ Data Contract Breached!")
    print(f"Total errors found: {len(err.failure_cases)}")

    # Let's look at the specific failures
    print("\nFailure Report:")
    print(err.failure_cases[['column', 'check', 'failure_case']])
```

### 输出

如果你运行上面的代码，你不会得到一个通用的`KeyError`。你会得到一份具体的报告，详细说明合同是如何被违反的：

```py
⚠️ Data Contract Breached!
Total errors found: 3

Failure Report:
        column                 check      failure_case
0        email           str_matches     INVALID_EMAIL
1   lead_score   less_than_or_equal_to             1.5
2   lead_score   greater_than_or_equal_to         -0.1
```

在一个更现实的场景中，你可能会将输出记录到文件中，并设置警报，以便在出现问题时得到通知。

* * *

## 这为什么重要

这种方法改变了你工作的动态。

没有合同，你的代码会在转换逻辑的深处失败（或者更糟，它没有失败，但你却将错误数据写入仓库）。你花费数小时调试`NaN`值。

**有合同**：

1.  **快速失败**：管道在门口停止。错误数据永远不会进入你的核心逻辑。

1.  **明确责任**：你可以将那份失败报告发送给数据提供者，并说，“第 3 行和第 4 行违反了模式。请修复。”

1.  **文档**：`LeadsContract`类充当了活文档。新加入项目的成员不需要猜测列代表什么；他们可以直接阅读代码。你也避免了在 SharePoint、Confluence 或其他地方设置单独的数据合同，这些合同很快就会过时。

* * *

## “足够好”的解决方案

你当然可以更深入。你可以将其与**Airflow**集成，将指标推送到仪表板，或者使用**great_expectations**等工具进行更复杂的统计分析。

但在我看到的 90%的使用场景中，在 Python 脚本开始时进行简单的验证步骤就足够了，这样你就可以在周五晚上睡得香甜。

从小做起。为你的最混乱的数据集定义一个模式，将其包裹在 try/catch 块中，看看这能为你这周节省多少烦恼。当这种简单的方法不再适用时， THEN 我会考虑更复杂的工具来处理数据合同。

如果你对 AI、数据科学或数据工程感兴趣，请关注我或在[LinkedIn](https://www.linkedin.com/in/eirik-berge/)上与我联系。

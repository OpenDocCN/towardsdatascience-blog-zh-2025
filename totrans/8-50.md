# Airflow 数据区间：深入探讨

> 原文：[`towardsdatascience.com/airflow-data-intervals-a-deep-dive-15d0ccfb0661/`](https://towardsdatascience.com/airflow-data-intervals-a-deep-dive-15d0ccfb0661/)

![由 Gareth David 在 Unsplash 上的照片](img/09636bceb2810baa82d31ce2516d5c37.png)

由[Gareth David](https://unsplash.com/@gareth_david?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)在[Unsplash](https://unsplash.com/photos/grayscale-photo-of-body-of-water-m0chaAschUw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)上的照片

Apache Airflow 是一个强大的工作流程调度和监控工具，但它的行为有时可能感觉不符合直觉，尤其是在数据区间方面。

理解这些区间对于构建可靠的数据管道、确保幂等性和启用可重放性至关重要。通过有效地利用数据区间，你可以确保即使在重试或回填的情况下，你的工作流程也能产生一致和准确的结果。

在本文中，我们将详细探讨 Airflow 的数据区间，讨论其设计背后的原因，为什么引入了它们，以及它们如何简化并增强日常数据工程工作。

* * *

## Airflow 中的数据区间是什么？

数据区间位于 Apache Airflow 调度和执行工作流程的核心。简单来说，数据区间代表 DAG 运行负责处理的具体时间范围。

例如，在每日调度的工作流程中，每个数据区间从午夜（00:00）开始，到第二天午夜（24:00）结束。DAG 只有在数据区间结束后才会执行，确保该区间内的数据完整且准备好处理。

### 数据区间的直觉

数据区间的引入是由标准化和简化基于时间数据的工作流程操作的需求所驱动的。在许多数据工程场景中，任务需要处理特定时间段的数据，例如每小时日志或每日交易记录。如果没有明确的数据区间概念，工作流程可能会在不完整的数据上运行或与其他运行重叠，导致不一致和潜在的错误。

数据区间提供了一种结构化的方式来：

+   确保 DAG 只处理它们应该处理的数据

+   避免由迟到或仍在写入的数据引起的问题

+   为历史时期的工作流程启用精确的回填和重放功能

+   通过确保每个 DAG 运行处理一个明确定义且不可变的时间范围内的数据，支持幂等性。这防止了在重试或重放工作流程时出现数据重复或遗漏

通过将每个 DAG 运行与特定的数据区间绑定，Airflow 确保任务可以在不影响其他运行的情况下安全地重试，并且可以自信地重放工作流程，知道它们每次都会产生一致的结果。

### 数据区间是如何工作的？

当 DAG 被调度时，Airflow 为每个运行分配一个逻辑日期，这对应于其数据区间的开始。这个逻辑日期不是实际的执行时间，而是一个数据区间的标记。每个数据区间由以下定义：

+   **数据区间开始：**DAG 运行负责处理的时间范围的开始。

+   **数据区间结束：**同一 DAG 运行的同一时间范围的结束。

+   **逻辑日期：**表示数据区间开始的戳记，用作 DAG 运行的参考点。

例如：

如果一个 DAG 计划每天运行，`start_date`为 2025 年 1 月 1 日，第一个数据区间将具有：

+   **数据区间开始：**2025 年 1 月 1 日，00:00

+   **数据区间结束：**2025 年 1 月 2 日，00:00

+   **逻辑日期：**2025 年 1 月 1 日

+   该区间的 DAG 运行将在区间结束时执行，即在 2025 年 1 月 2 日午夜。

这种方法确保 DAG 在其代表的时间区间上操作的是完整的数据集。

* * *

## `start_date`的作用

DAG 中的`start_date`定义了第一个数据区间的逻辑开始。这是一个关键参数，它决定了 DAG 何时开始跟踪其数据区间，并影响回填功能。

当你设置`start_date`时，它作为 Airflow 计算数据区间的参考点。再次强调，DAG 在部署后不会立即执行；它会在启动运行之前等待第一个数据区间结束。这确保了该区间的数据是完整且准备好的，可以进行处理。

### 为什么`start_date`很重要

`start_date`在 Airflow 确定数据区间边界方面也起着重要作用。通过定义一个固定的起始点，Airflow 确保所有区间都得到一致的计算，从而使工作流程能够以结构化和可靠的方式处理数据。

通过将 DAG 运行锚定到一致的`start_date`，你确保了可重复的工作流程，这些工作流程在不同环境和时间范围内可以可预测地运行。

1.  **区间逻辑开始：**`start_date`确定了第一个数据区间的开始，这为所有后续区间奠定了基础。例如，如果你的`start_date`是 2025 年 1 月 1 日，并且调度是`@daily`，第一个区间将跨越 2025 年 1 月 1 日 00:00 至 2025 年 1 月 2 日 00:00。

1.  **与调度对齐：**确保`start_date`与 DAG 的调度对齐对于可预测的执行至关重要。两者之间的不匹配可能导致处理中出现意外的间隙或重叠。

1.  **回填支持：**精心选择的`start_date`使你能够通过触发过去区间的 DAG 运行来回填历史数据。这种能力对于需要处理旧数据集的工作流程至关重要。

1.  **可重复性：**通过将 DAG 运行锚定到一致的`start_date`，你确保了可重复的工作流程，这些工作流程在不同环境和时间范围内可以可预测地运行。

在下面展示的示例 DAG 中，

+   DAG 被安排从 2025 年 1 月 1 日起每天运行

+   `catchup=True`参数确保如果 DAG 在`start_date`之后部署，它将回补所有错过的时间间隔

```py
from datetime import datetime

from airflow import DAG
from airflow.operators.dummy import DummyOperator

with DAG(
    dag_id='example_dag',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=True,  # Enables backfilling
) as dag:
    start = DummyOperator(task_id='start')
    end = DummyOperator(task_id='end')

    start >> end
```

如果 DAG 在 2025 年 1 月 1 日上午 10:00 部署，第一次运行不会立即执行。相反，它将在 2025 年 1 月 2 日午夜运行，处理 2025 年 1 月 1 日的数据。

如果 DAG 在 2025 年 1 月 3 日部署，它将在执行当前间隔之前回补 2025 年 1 月 1 日和 1 月 2 日的运行。

### start_date 的常见错误

虽然`start_date`参数很简单，但其不当配置可能会导致意外行为或工作流程失败。了解这些陷阱对于避免常见错误并确保平稳执行至关重要。

**任意选择 `start_date`**在没有考虑数据可用性或计划的情况下随机选择`start_date`可能会导致运行失败或不完整处理。

**使用动态值**避免将`start_date`设置为动态值，例如`datetime.now()`。由于`start_date`旨在作为一个固定的参考点，使用动态值可能会导致不可预测的间隔，并使得回补变得不可能。

**忽略时区**始终确保`start_date`考虑了数据源的时间区，以避免间隔错位。

**配置不当的回补设置**如果禁用了`catchup`，则将不会处理错过的时间间隔，可能会导致数据缺失。

* * *

## Airflow 模板参考

Airflow 提供了几个[模板变量](https://airflow.apache.org/docs/apache-airflow/stable/templates-ref.html)，以简化与数据间隔的交互。这些变量对于创建动态的、可维护的和幂等的流程至关重要。

通过使用这些模板，您可以确保您的 DAG 能够适应变化的数据间隔，而无需使用硬编码的值，这使得它们更加健壮且易于调试。

Airflow 在底层使用[Jinja 模板](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html#concepts-jinja-templating)来实现这种动态行为，允许您直接在任务参数或脚本中嵌入这些变量。

### 数据间隔模板变量

以下是一些常用的模板引用列表，您可以将它们嵌入到您的 DAG 中：

+   **`{{ data_interval_start }}`**：表示当前数据间隔的开始。使用这个变量来查询或处理位于间隔开始的数据。

+   **`{{ data_interval_end }}`**：表示当前数据间隔的结束。这对于定义数据处理任务的边界很有帮助。

+   **`{{ logical_date }}`**：DAG 运行的逻辑日期，与数据间隔的开始相一致。这通常用于日志记录或元数据目的。

+   **`{{ prev_data_interval_start_success }}`**：前一个成功 DAG 运行的数据区间的开始。对于依赖于早期运行输出的任务，请使用此变量。

+   **`{{ prev_data_interval_end_success }}`**：前一个成功 DAG 运行的数据区间的结束。这确保了处理顺序数据的流程的连续性。

您可以在 [Airflow 文档的相关部分](https://airflow.apache.org/docs/apache-airflow/stable/templates-ref.html) 中找到所有可用模板引用的更完整列表。

### 在编写 DAG 时模板引用的重要性

模板变量是编写灵活且高效的 Airflow DAG 的一个重要组成部分。它们提供了一种动态引用数据区间关键属性的方法，确保您的流程能够适应变化，并且易于维护。

这些变量之所以如此重要，以下是一些主要原因：

1.  **动态适应性**：模板变量允许您的 DAG 自动调整到当前数据区间，从而消除硬编码特定日期或时间范围的需求。

1.  **幂等性**：通过将任务参数绑定到特定的数据区间，您确保了重运行或重试无论何时执行都会产生相同的结果。

1.  **易于维护**：使用模板变量可以降低错误风险并简化更新。例如，如果您的数据处理逻辑发生变化，您可以调整模板而无需重写 DAG。

1.  **简化回填操作**：这些变量使得回填历史数据变得简单，因为每个 DAG 运行都会自动与适当的数据区间关联。

1.  **利用 Jinja 模板**：Jinja 模板允许您在任务命令、SQL 查询或脚本中动态嵌入这些变量。这确保了您的流程保持灵活且具有上下文感知性。

* * *

## 可视化数据区间

为了更好地理解数据区间，请考虑以下可视化。

![表示 Airflow 中数据区间如何工作的可视化 - 来源：作者](img/f453557e959fa0721cca33a352bf1768.png)

表示 Airflow 中数据区间工作原理的可视化 – 来源：[作者](https://medium.com/@gmyrianthous)

每个 DAG 都有一个单一的开始日期，并且每个 DAG 运行仅在对应的数据区间结束后执行。下表说明了每个每日运行中三个感兴趣的数据区间变量的值。

```py
| Logical Date | Data Interval Start | Data Interval End |
| ------------ | ------------------- | ----------------- |
| 2025-01-01   | 2025-01-01 00:00    | 2025-01-02 00:00  |
| 2025-01-02   | 2025-01-02 00:00    | 2025-01-03 00:00  |
| 2025-01-03   | 2025-01-03 00:00    | 2025-01-04 00:00  |
| 2025-01-04   | 2025-01-04 00:00    | 2025-01-05 00:00  |
...
```

* * *

## 一个工作示例

在下面的示例 DAG 中，我们尝试从一个接受日期参数的公共 API（jsonplaceholder）中获取帖子。在我们的场景中，我们希望指定日期参数的值为区间的开始。换句话说，如果 DAG 应该在今天运行，我们希望使用昨天的日期调用 API。

```py
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.http.operators.http import SimpleHttpOperator
from airflow.utils.dates import days_ago
from airflow.utils.dates import timedelta
from datetime import datetime

def get_posts_for_data_interval_start_date(**kwargs):
    """
    Fetch Posts from jsonplaceholder
    """
    import requests

    data_interval_start_dt = kwargs['data_interval_start']

    # Format the date as required by the API
    formatted_date = data_interval_start_dt.strftime('%Y-%m-%d')    
    print(f"Calling API for date: {formatted_date}")

    # Call the API
    response = requests.get(
        'https://jsonplaceholder.typicode.com/posts', 
        params={'date': formatted_date}
    )

    if response.status_code == 200:
        print(f'API call successful for {formatted_date}')
    else:
        print(f'API call failed for {formatted_date} with status code {response.status_code}')

dag = DAG(
    'test_dag',
    default_args={
        'owner': 'airflow',
        'retries': 1,
        'retry_delay': timedelta(minutes=5),
    },
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=True,  # Enable backfilling, to run missed intervals
)

api_task = PythonOperator(
    task_id='call_api_for_date',
    python_callable=get_posts_for_data_interval_start_date,
    dag=dag,
)
```

DAG 被安排从 2025 年 1 月 1 日起每天运行。它利用 `data_interval_start` 模板变量动态传递每次运行的日期。具体来说，`data_interval_start` 对应于每个 DAG 运行的执行窗口的开始，使得 API 能够接收到正确的日期参数。

`start_date` 参数决定了 DAG 执行开始的时间，而 `catchup=True` 设置确保了错过的时间区间会被回填。这意味着如果 DAG 被延迟或系统出现故障，Airflow 将自动执行错过日期的任务，确保没有数据被遗漏。

DAG 的核心是一个调用 API 的 Python 任务，它将格式化的日期作为 `data_interval_start` 传递。这允许一个灵活的、基于区间的 API 查询系统。

* * *

## 最后的想法

理解 Airflow 的数据区间对于构建可靠的流程至关重要。通过学习 `start_date`、逻辑日期和模板变量如何协同工作，你可以创建准确高效地处理数据的管道。无论是回填历史数据还是管理复杂的流程，这些原则确保你的管道运行顺畅且一致。

掌握数据区间将帮助你设计更容易维护和适应的工作流程，使你在数据工程中的日常工作更加高效和可预测。

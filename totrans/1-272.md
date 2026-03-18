# 从 Python 到 JavaScript：n8n 中数据分析的剧本，包含代码节点示例

> 原文：[`towardsdatascience.com/from-python-to-javascript-a-playbook-for-data-analytics-in-n8n-with-code-node-examples/`](https://towardsdatascience.com/from-python-to-javascript-a-playbook-for-data-analytics-in-n8n-with-code-node-examples/)

<mdspan datatext="el1758174294461" class="mdspan-comment">当我创建我的第一个 n8n 工作流程时，作为一个数据科学家，我感觉我像是在作弊。</mdspan>

我可以不阅读 30 页的文档就连接到 API，从 Gmail 或 Sheets 触发工作流程，并在几分钟内部署一些有用的东西。

然而，一个显著的缺点是 n8n 并未原生优化以在客户使用的云实例中运行 Python 环境。

和许多数据科学家一样，我的数据分析日常工具箱建立在 **NumPy** 和 **Pandas** 之上。

为了保持在我的舒适区，我经常将计算外包给外部 API，而不是使用 n8n JavaScript 代码节点。

![图片](img/b24b4375a364d7240e99a4fe9be4eece.png)[(https://towardsdatascience.com/ai-agents-for-supply-chain-optimisation-production-planning/)]

使用 API 函数调用的生产计划 n8n 工作流程 – (图片由 Samir Saci 提供)

例如，这是使用**生产计划优化工具**所做的工作，该工具通过包括调用 FastAPI 微服务的 Agent 节点的工作流程进行编排。

这种方法有效，但我有一些客户要求**在他们的 n8n 用户界面上完全可见数据分析任务**。

我意识到我只需要学习足够的 JavaScript，就能使用 n8n 的原生代码节点进行数据处理。

![图片](img/93fc9d160419f0ee1e0b2e355f679765.png)

JavaScript 节点按项目分组销售的示例 – (图片由 Samir Saci 提供)

在这篇文章中，我们将尝试在 n8n 代码节点内的小型 JavaScript 片段，以执行日常的数据分析任务。

为了这次练习，我将使用销售交易数据集，并对其进行 ABC 和 Pareto 分析，这些分析在供应链管理中广泛使用。

![图片](img/ced6268499c2d3f5faf4f4352e425d12.png)

ABC XYZ & Pareto 图在供应链管理中广泛使用 – (图片由 Samir Saci 提供)

我将在 n8n 代码节点中提供 Pandas 与 JavaScript 的并排示例，使我们能够直接将熟悉的 Python 数据分析步骤转换为自动化的 n8n 工作流程。

![图片](img/91ee713756b8babf119832190198c02e.png)

JavaScript 与 Pandas 的示例 – (图片由 Samir Saci 提供)

策略是利用云企业 n8n 实例的能力（即不使用社区节点）来实现这些解决方案，用于小型数据集或快速原型设计。

![图片](img/3ffbefbb5c4d14f2478918b076bc256d.png)

我们将一起构建的实验工作流程 – (图片由 Samir Saci 提供)

我将以一个快速的性能与 FastAPI 调用的比较研究来结束实验。

**您可以关注我并使用文章中分享的 Google Sheet 和工作流程模板来复制整个工作流程**。

让我们开始吧！

## 在 n8n 中使用 JavaScript 构建数据分析工作流程

在开始构建节点之前，我将介绍这个分析的环境。

### 供应链管理的 ABC & 帕累托图

对于这个教程，我建议您构建一个简单的流程，该流程从**Google Sheets 中的销售交易**中提取数据，并将其转换为全面的 ABC 和帕累托图表。

这将复制我初创公司 LogiGreen 开发的 [LogiGreen Apps](https://www.logi-green.com/logigreen-applications-for-supply-chain-optimization) 中的 **ABC 和帕累托分析**模块。

![图片](img/a3808addd9773fdb583547310642b996.png)

[LogiGreen Apps](https://www.logi-green.com/logigreen-applications-for-supply-chain-optimization) 的 ABC 分析模块 – （图像由 Samir Saci 提供）

目标是为超市连锁店的库存团队生成一系列视觉效果，帮助他们了解其门店的销售分布。

我们将专注于生成两个视觉效果。

第一张图表显示了销售项目的 **ABC-XYZ 分析**：

![图片](img/d46c8c4e98274d4d82c0773ffffe4f9b.png)

ABC XYZ 图 – （图像由 Samir Saci 提供）

+   **X 轴（周转率百分比）**：每个项目对总收入的贡献。

+   **Y 轴（变异系数）**：每个项目的需求变化。

+   垂直的红色线条根据周转率将项目分为**A、B 和 C**三个类别。

+   水平的蓝色线条标记**稳定需求与变动需求**（CV=1）。

一起，它突出了哪些项目是**高价值且稳定的（A，CV 低）**，与那些**低价值或高度可变**的项目相比，指导库存管理中的优先级排序。

第二个视觉效果是**销售周转的帕累托分析**：

![图片](img/4594cc01ddab96b20e1ee7ed26f6ece6.png)

由 [Logigreen App](https://logigreenconsulting.odoo.com/logigreen-applications-for-supply-chain-optimization) 生成的帕累托图 – 图像由 Samir Saci 提供

+   **X 轴：**SKU 百分比（按销售额排名）。

+   **Y 轴：**年度总周转率的累积百分比。

+   曲线说明了少数项目如何对大部分收入做出贡献。

简而言之，这突出了（或不突出）经典的帕累托法则，该法则确认 80% 的销售额可以来自 20% 的 SKU。

> 我是如何生成这两个视觉效果的？我只是简单地使用了 Python。

在我的 YouTube 频道上，我分享了一个[完整教程](https://youtu.be/Qglr9Yqa44I)，介绍了如何使用 Pandas 和 Matplotlib 来实现它。

本教程的目标是使用**仅 n8n 的原生 JavaScript 节点**来准备销售交易并生成这些视觉效果。

### 在 n8n 中构建数据分析工作流程

我建议构建一个手动触发的流程，以方便在开发期间进行调试。

![图片](img/e692c4c6129fa1af6bff92a730e1d0a5.png)

手动触发的最终工作流程，从 Google Sheets 收集数据以生成视觉效果 – （图像由 Samir Saci 提供）

要遵循此教程，您需要

+   复制此链接中可用的电子表格：[Google 表格](https://docs.google.com/spreadsheets/d/1nsS_pj3o8yMEIQ59QBU80sx20HtkkTgCKJB32sOge1o/edit?usp=sharing)

+   在我的[n8n 创作者个人资料](https://n8n.io/creators/samirsaci/)上下载模板

您现在可以使用第二个节点连接您的副本，该节点将提取工作表中的数据集：`输入数据`。

![图片](img/6c6500fe40cd4814b4cf5195810c3874.png)

将第二个节点连接到您的 Google 表格副本以收集输入数据 - （图片由 Samir Saci 提供）

此数据集包括按日粒度的零售销售交易：

+   `ITEM`: 可在多个商店销售的项目

+   `SKU`: 代表在特定商店销售的`SKU`

+   `FAMILY`: 一组项目

+   `CATEGORY`: 产品类别可以包含多个家族

+   `STORE`: 代表销售位置的代码

+   交易的`DAY`

+   `QTY`: 销售数量（单位）

+   `TO`: 以欧元计的销售数量

输出是表格内容的 JSON 格式，可供其他节点摄取。

*Python 代码*

```py
import pandas as pd
df = pd.read_csv("sales.csv") 
```

我们现在可以开始处理数据集以构建我们的两个可视化。

#### **步骤 1：过滤掉没有销售的交易**

让我们从过滤掉销售`QTY`等于零的交易这个简单的动作开始。

![图片](img/c1d6a5993229c38ff0533015af554fa3.png)

使用过滤节点过滤掉没有销售的交易 - （图片由 Samir Saci 提供）

我们不需要 JavaScript；一个简单的**过滤**节点就可以完成这项工作。

*Python 代码*

```py
df = df[df["QTY"] != 0]
```

#### **步骤 2：准备帕累托分析数据**

我们首先需要按`ITEM`聚合销售并按营业额对产品进行排名。

*Python 代码*

```py
sku_agg = (df.groupby("ITEM", as_index=False)
             .agg(TO=("TO","sum"), QTY=("QTY","sum"))
             .sort_values("TO", ascending=False))
```

在我们的工作流程中，此步骤将在 JavaScript 节点`TO, QTY GroupBY ITEM`中完成：

```py
const agg = {};
for (const {json} of items) {
  const ITEM = json.ITEM;
  const TO = Number(json.TO);
  const QTY = Number(json.QTY);
  if (!agg[ITEM]) agg[ITEM] = { ITEM, TO: 0, QTY: 0 };
  agg[ITEM].TO += TO;
  agg[ITEM].QTY += QTY;
}
const rows = Object.values(agg).sort((a,b)=> b.TO - a.TO);
return rows.map(r => ({ json: r })); 
```

此节点返回按`ITEM`计量的销售排名表，包括数量（QTY）和营业额（TO）：

1.  我们以 ITEM 为键初始化 agg 字典

1.  我们在项目列表中循环遍历 n8n 行

+   将 TO 和 QTY 转换为数字

+   将 QTY 和 TO 值添加到每个 ITEM 的运行总和中

1.  我们最终将字典转换为按 TO 降序排序的数组并返回项目

![图片](img/cfdd96041185acb7b3ee64203c9dc8bb.png)

输出按`ITEM`聚合的销售数据 - （图片由 Samir Saci 提供）

我们现在有了数据，可以执行销售数量（QTY）或营业额（TO）的**帕累托分析**。

因此，我们需要计算累计销售额并按 SKU 从高到低排序。

*Python 代码*

```py
abc = sku_agg.copy()  # from Step 2, already sorted by TO desc
total = abc["TO"].sum() or 1.0
abc["cum_turnover"] = abc["TO"].cumsum()
abc["cum_share"]    = abc["cum_turnover"] / total             
abc["sku_rank"]     = range(1, len(abc) + 1)
abc["cum_skus"]     = abc["sku_rank"] / len(abc)               
abc["cum_skus_pct"] = abc["cum_skus"] * 100 
```

此步骤将在代码节点`Pareto Analysis`中完成：

```py
const rows = items
  .map(i => ({
    ...i.json,
    TO: Number(i.json.TO || 0),   
    QTY: Number(i.json.QTY || 0),
  }))
  .sort((a, b) => b.TO - a.TO);

const n = rows.length; // number of ITEM
const totalTO = rows.reduce((s, r) => s + r.TO, 0) || 1;
```

我们从上一个节点收集数据集`items`

1.  对于每一行，我们清理`TO`和`QTY`字段（以防我们缺少值）

1.  我们**按营业额降序**对所有 SKU 进行排序。

1.  我们将项目数量和总营业额存储在变量中

```py
let cumTO = 0;
rows.forEach((r, idx) => {
  cumTO += r.TO;
  r.cum_turnover = cumTO;                     
  r.cum_share = +(cumTO / totalTO).toFixed(6); 
  r.sku_rank = idx + 1;
  r.cum_skus = +((idx + 1) / n).toFixed(6);   
  r.cum_skus_pct = +(r.cum_skus * 100).toFixed(2);
});

return rows.map(r => ({ json: r }));
```

然后我们按顺序循环遍历所有项目。

1.  使用变量`cumTO`计算累计贡献

1.  为每一行添加几个**帕累托指标**：

+   `cum_turnover`: 到此项目为止的累计营业额

+   `cum_share`: 营业额累计份额

+   `sku_rank`：项目的排名位置

+   `cum_skus`：SKU 总数中累积 SKU 的数量作为分数

+   `cum_skus_pct`：与`cum_skus`相同，但以百分比表示。

然后，我们就完成了**帕累托图**的数据准备工作。

![图片](img/e13c05dd698aabe6394c8bb5f8a7e383.png)

最终结果 – (图片由 Samir Saci 提供)

此数据集将由节点 `Update Pareto Sheet` 存储在 `Pareto` 工作表中。

通过一点魔法，我们可以在第一个工作表中生成此图：

![图片](img/5ae5ed9ec0b382284bea24647f9fba61.png)

使用 n8n 工作流程处理的数据生成的帕累托图 – (图片由 Samir Saci 提供)

我们现在可以继续 ABC XYZ 图。

#### 步骤 3：计算需求变异性和销售贡献

我们可以考虑帕累托图的销售贡献，但我们将每个图表视为独立的。

我将把节点“需求变异性和‘销售 x 销售百分比’”的代码分成多个部分以提高清晰度。

*块 1：定义均值和标准差函数*

```py
function mean(a){ return a.reduce((s,x)=>s + x, 0) / (a.length || 1); }
function stdev_samp(a){
  if (a.length <= 1) return 0;
  const m = mean(a);
  const v = a.reduce((s,x)=> s + (x - m) ** 2, 0) / (a.length - 1);
  return Math.sqrt(v);
}
```

这两个函数将用于变异系数（Cov）

+   `mean(a)`：计算数组的平均值。

+   `stdev_samp(a)`：计算**样本标准差**

它们以我们在这个第二块中构建的每个 `ITEM` 的每日销售分布为输入。

*块 2：创建每个 `ITEM` 的每日销售分布*

```py
const series = {};  // ITEM -> { day -> qty_sum }
let totalQty = 0;

for (const { json } of items) {
  const item = String(json.ITEM);
  const day  = String(json.DAY);
  const qty  = Number(json.QTY || 0);

  if (!series[item]) series[item] = {};
  series[item][day] = (series[item][day] || 0) + qty;
  totalQty += qty;
}
```

*Python 代码*

```py
import pandas as pd
import numpy as np
df['QTY'] = pd.to_numeric(df['QTY'], errors='coerce').fillna(0)
daily_series = df.groupby(['ITEM', 'DAY'])['QTY'].sum().reset_index()
```

现在我们可以计算应用于每日销售分布的指标。

```py
const out = [];
for (const [item, dayMap] of Object.entries(series)) {
  const daily = Object.values(dayMap); // daily sales quantities
  const qty_total = daily.reduce((s,x)=>s+x, 0);
  const m = mean(daily);               // average daily sales
  const sd = stdev_samp(daily);        // variability of sales
  const cv = m ? sd / m : null;        // coefficient of variation
  const share_qty_pct = totalQty ? (qty_total / totalQty) * 100 : 0;

  out.push({
    ITEM: item,
    qty_total,
    share_qty_pct: Number(share_qty_pct.toFixed(2)),
    mean_qty: Number(m.toFixed(3)),
    std_qty: Number(sd.toFixed(3)),
    cv_qty: cv == null ? null : Number(cv.toFixed(3)),
  });
}
```

对于每个 `ITEM`，我们计算

+   `qty_total`：总销售额

+   `mean_qty`：平均每日销售额。

+   `std_qty`：每日销售额的标准差。

+   `cv_qty`：变异系数（XYZ 分类的变异度量）

+   `share_qty_pct`：对总销售额的贡献百分比（用于 ABC 分类）

这里是 Python 版本，以防你迷路：

```py
summary = daily_series.groupby('ITEM').agg(
    qty_total=('QTY', 'sum'),
    mean_qty=('QTY', 'mean'),
    std_qty=('QTY', 'std')
).reset_index()

summary['std_qty'] = summary['std_qty'].fillna(0)

total_qty = summary['qty_total'].sum()
summary['cv_qty'] = summary['std_qty'] / summary['mean_qty'].replace(0, np.nan)
summary['share_qty_pct'] = 100 * summary['qty_total'] / total_qty
```

我们几乎完成了。

我们只需要按贡献度降序排序，为 ABC 类别映射做准备：

```py
out.sort((a,b) => b.share_qty_pct - a.share_qty_pct);
return out.map(r => ({ json: r }));
```

对于每个 `ITEM`，我们现在都有了创建散点图所需的关键指标。

![图片](img/b3992659c3728776c8a63ff7e0a892bc.png)

节点 `Demand Variability x Sales %` 的输出 – (图片由 Samir Saci 提供)

目前缺少的只有 ABC 类别。

**步骤 4：添加 ABC 类别**

我们将前一个节点的输出作为输入。

```py
let rows = items.map(i => i.json);
rows.sort((a, b) => b.share_qty_pct - a.share_qty_pct);
```

万一的话，我们按**销售额百分比**降序排序 `ITEMS` → 最重要的 SKU 排在前面。

(*此步骤可以省略，因为它通常已经在之前的代码节点结束时完成。*)

然后我们可以根据硬编码的条件应用类别：

+   **A**：代表前**5%**销售额的 SKU

+   **B**：代表接下来**15%**销售额的 SKU

+   **C**：20%之后的所有内容。

```py
let cum = 0;
for (let r of rows) {
  cum += r.share_qty_pct;

  // 3) Assign class based on cumulative %
  if (cum <= 5) {
    r.ABC = 'A';   // top 5%
  } else if (cum <= 20) {
    r.ABC = 'B';   // next 15%
  } else {
    r.ABC = 'C';   // rest
  }

  r.cum_share = Number(cum.toFixed(2));
}

return rows.map(r => ({ json: r }));
```

这可以通过 Python 代码这样做。

```py
df = df.sort_values('share_qty_pct', ascending=False).reset_index(drop=True)
df['cum_share'] = df['share_qty_pct'].cumsum()
def classify(cum):
    if cum <= 5:
        return 'A'
    elif cum <= 20:
        return 'B'
    else:
        return 'C'
df['ABC'] = df['cum_share'].apply(classify)
```

现在可以使用这些结果生成此图，该图可在 Google Sheets 的第一个工作表中找到：

![图片](img/d6d01bb553210b7b13a3049f5c7a0814.png)

使用工作流程处理的数据生成的 ABC XYZ 图 – (图片由 Samir Saci 提供)

我（可能由于我对 Google Sheets 的有限了解）努力寻找一个“手动”解决方案来创建这个具有正确颜色映射的散点图。

因此，我使用了在 Google Sheet 中可用的 Google Apps Script 来创建它。

![](img/06fb4f6cfbe5dc03311a5ef55098ddf7.png)

包含在 [Google Sheet](https://docs.google.com/spreadsheets/d/1nsS_pj3o8yMEIQ59QBU80sx20HtkkTgCKJB32sOge1o/edit?usp=sharing) 中的脚本以生成可视化效果 – （图片由 Samir Saci 提供）

作为额外的好处，我在 n8n 模板中添加了更多执行相同类型 GroupBy 操作以按店铺或商品-店铺对计算销售的节点。

![](img/3ffbefbb5c4d14f2478918b076bc256d.png)

我们共同构建的实验工作流程 – （图片由 Samir Saci 提供）

他们可以用来创建这样的可视化：

![](img/7162b57a68a9fe94d54a4e930348af47.png)

每家店铺的每日总销售数量 – （图片由 Samir Saci 提供）

为了结束这个教程，我们可以自信地宣布工作已经完成。

为了进行工作流程的实时演示，您可以查看这个简短的教程

我们的客户，在他们的 n8n 云实例上运行此工作流程，现在可以了解数据处理中的每个步骤。

> 但代价是什么？我们在性能上是否有所损失？

这是我们将在下一节中发现的。

## 性能比较研究：n8n JavaScript 节点与 FastAPI 中的 Python

为了回答这个问题，我准备了一个简单的实验。

在 n8n 内部使用两种不同的方法处理相同的 dataset 和转换：

1.  **全部在 JavaScript 节点中**，函数直接在 n8n 内部。

1.  **通过将 JavaScript 逻辑替换为对 Python 端点的 HTTP 请求来外包到 FastAPI 微服务中**。

![](img/1be416d0caed6e43084b31dd5cbbf7ff.png)

使用 FastAPI 微服务的简单工作流程 – （图片由 Samir Saci 提供）

这两个端点连接到函数，可以直接从我在其中托管微服务的 VPS 实例加载数据。

```py
@router.post("/launch_pareto")
async def launch_speedtest(request: Request):
    try:
        session_id = request.headers.get('session_id', 'session')

        folder_in = f'data/session/speed_test/input'
        if not path.exists(folder_in):
                makedirs(folder_in)

        file_path = folder_in + '/sales.csv'
        logger.info(f"[SpeedTest]: Loading data from session file: {file_path}")
        df = pd.read_csv(file_path, sep=";")
        logger.info(f"[SpeedTest]: Data loaded successfully: {df.head()}")

        speed_tester = SpeedAnalysis(df)
        output = await speed_tester.process_pareto()

        result = output.to_dict(orient="records")
        result = speed_tester.convert_numpy(result)

        logger.info(f"[SpeedTest]: /launch_pareto completed successfully for {session_id}")
        return result
    except Exception as e:
        logger.error(f"[SpeedTest]: Error /launch_pareto: {str(e)}\n{traceback.format_exc()}")
        raise HTTPException(status_code=500, detail=f"Failed to process Speed Test Analysis: {str(e)}")

@router.post("/launch_abc_xyz")
async def launch_abc_xyz(request: Request):
    try:
        session_id = request.headers.get('session_id', 'session')

        folder_in = f'data/session/speed_test/input'
        if not path.exists(folder_in):
                makedirs(folder_in)

        file_path = folder_in + '/sales.csv'
        logger.info(f"[SpeedTest]: Loading data from session file: {file_path}")
        df = pd.read_csv(file_path, sep=";")
        logger.info(f"[SpeedTest]: Data loaded successfully: {df.head()}")

        speed_tester = SpeedAnalysis(df)
        output = await speed_tester.process_abcxyz()

        result = output.to_dict(orient="records")
        result = speed_tester.convert_numpy(result)

        logger.info(f"[SpeedTest]: /launch_abc_xyz completed successfully for {session_id}")
        return result
    except Exception as e:
        logger.error(f"[SpeedTest]: Error /launch_abc_xyz: {str(e)}\n{traceback.format_exc()}")
        raise HTTPException(status_code=500, detail=f"Failed to process Speed Test Analysis: {str(e)}") 
```

我只想关注数据处理的性能。

`SpeedAnalysis` 包括上一节中列出的所有数据处理步骤

+   按 `ITEM` 对销售进行分组

+   按 `ITEM` 降序排序并计算累计销售额

+   通过 `ITEM` 计算销售分布的标准差和平均值

```py
class SpeedAnalysis:
    def __init__(self, df: pd.DataFrame):
        config = load_config()

        self.df = df

    def processing(self):
        try:
            sales = self.df.copy()
            sales = sales[sales['QTY']>0].copy()
            self.sales = sales

        except Exception as e:
            logger.error(f'[SpeedTest] Error for processing : {e}\n{traceback.format_exc()}')

    def prepare_pareto(self):
        try:
            sku_agg = self.sales.copy()
            sku_agg = (sku_agg.groupby("ITEM", as_index=False)
             .agg(TO=("TO","sum"), QTY=("QTY","sum"))
             .sort_values("TO", ascending=False))

            pareto = sku_agg.copy()  
            total = pareto["TO"].sum() or 1.0
            pareto["cum_turnover"] = pareto["TO"].cumsum()
            pareto["cum_share"]    = pareto["cum_turnover"] / total              
            pareto["sku_rank"]     = range(1, len(pareto) + 1)
            pareto["cum_skus"]     = pareto["sku_rank"] / len(pareto)               
            pareto["cum_skus_pct"] = pareto["cum_skus"] * 100
            return pareto                    
        except Exception as e:
            logger.error(f'[SpeedTest]Error for prepare_pareto: {e}\n{traceback.format_exc()}')

    def abc_xyz(self):
            daily = self.sales.groupby(["ITEM", "DAY"], as_index=False)["QTY"].sum()
            stats = (
                daily.groupby("ITEM")["QTY"]
                .agg(
                    qty_total="sum",
                    mean_qty="mean",
                    std_qty="std"
                )
                .reset_index()
            )
            stats["cv_qty"] = stats["std_qty"] / stats["mean_qty"].replace(0, np.nan)
            total_qty = stats["qty_total"].sum()
            stats["share_qty_pct"] = (stats["qty_total"] / total_qty * 100).round(2)
            stats = stats.sort_values("share_qty_pct", ascending=False).reset_index(drop=True)
            stats["cum_share"] = stats["share_qty_pct"].cumsum().round(2)
            def classify(cum):
                if cum <= 5:
                    return "A"
                elif cum <= 20:
                    return "B"
                else:
                    return "C"
            stats["ABC"] = stats["cum_share"].apply(classify)
            return stats

    def convert_numpy(self, obj):
        if isinstance(obj, dict):
            return {k: self.convert_numpy(v) for k, v in obj.items()}
        elif isinstance(obj, list):
            return [self.convert_numpy(v) for v in obj]
        elif isinstance(obj, (np.integer, int)):
            return int(obj)
        elif isinstance(obj, (np.floating, float)):
            return float(obj)
        else:
            return obj

    async def process_pareto(self):
        """Main processing function that calls all other methods in order."""
        self.processing()
        outputs = self.prepare_pareto()
        return outputs

    async def process_abcxyz(self):
        """Main processing function that calls all other methods in order."""
        self.processing()
        outputs = self.abc_xyz().fillna(0)
        logger.info(f"[SpeedTest]: ABC-XYZ analysis completed {outputs}.")
        return outputs
```

现在我们有了这些端点，我们可以开始测试。

![](img/9ef2e2df6d97863cbf0cc0d638d6f886.png)

实验结果（顶部：使用原生代码节点进行处理 / 底部：FastAPI 微服务） – （图片由 Samir Saci 提供）

结果显示在上文：

+   **仅 JavaScript 的工作流程**：整个过程在 **11.7 秒多一点** 内完成。

    *大部分时间都花在了更新表格和在 n8n 节点内进行迭代计算上。*

+   **基于 FastAPI 的工作流程**：等效的“外包”过程在 **~11.0 秒** 内完成。

    *将大量计算卸载到 Python 微服务中，它们处理得比原生 JavaScript 节点更快。*

换句话说，**将复杂的计算外包给 Python 实际上可以提高**性能。

原因是 FastAPI 端点直接执行优化的 Python 函数，而 n8n 内部的 JavaScript 节点必须迭代（使用循环）。

对于大型数据集，我想象的 delta 可能不是微不足道的。

# 结论

这证明了你可以在 n8n 中使用小的 JavaScript 片段进行简单的数据处理。

然而，我们的供应链分析产品可能需要更高级的处理，涉及优化和高级统计库。

![图片](img/53f55b2ffc3524b91ec1ec1a2ceab5bf.png)

[生产计划优化 AI 工作流程](https://towardsdatascience.com/ai-agents-for-supply-chain-optimisation-production-planning/) (图片由 Samir Saci 提供)

因此，客户可以接受处理一个“黑盒”方法，正如在[生产计划工作流程](https://towardsdatascience.com/ai-agents-for-supply-chain-optimisation-production-planning/)中看到的，该工作流程在[这篇 Towards Data Science 文章](https://towardsdatascience.com/ai-agents-for-supply-chain-optimisation-production-planning/)中介绍。

但对于**轻量级处理任务**，我们可以将它们集成到工作流程中，为非代码用户提供可见性。

对于另一个项目，我使用 n8n 连接供应链 IT 系统，使用电子数据交换（EDI）进行采购订单的传输。

![图片](img/c0a2bdca624661072689b98ffca499f4.png)

[电子数据交换（EDI）解析工作流程示例](https://n8n.io/workflows/3221-electronic-data-interchange-edi-message-parsing-with-gmail-and-google-sheet/) – (图片由 Samir Saci 提供)

这个工作流程，部署给一家小型物流公司，完全使用 JavaScript 节点解析 EDI 消息。

![图片](img/ea05403590a347940aaafb6e548e997c.png)

电子数据交换消息示例 – (图片由 Samir Saci 提供)

正如你可以在本教程中发现的那样，我们使用了 100%的 JavaScript 节点来执行电子数据交换消息的解析。

这帮助我们提高了解决方案的鲁棒性，并通过将维护工作转交给客户来减少我们的工作量。

> 最好的方法是什么？

对我来说，n8n 应该作为一个**编排和集成工具**使用，连接到我们的**核心分析产品**。

这些分析产品需要特定的输入格式，可能不符合我们客户的数据。

因此，我建议使用 JavaScript 代码节点来执行此预处理。

![图片](img/ad738347d6fafe008498d2fd4f0e10ff.png)

[分销计划优化算法工作流程](https://www.youtube.com/watch?v=DPWPTWXylas)的工作流程 – (图片由 Samir Saci 提供)

例如，上述工作流程将一个包含输入数据的 Google Sheets 连接到一个运行**分销计划优化算法**的 FastAPI 微服务**。

这个想法是将我们的优化算法集成到分销计划员使用的 Google Sheets 中，以组织商店的配送。

![图片](img/4818c3e14449e21e114513c54ab164aa.png)

计划团队使用的电子表格 – (图片由 Samir Saci 提供)

使用 JavaScript 代码节点将收集自 Google Sheets 的数据转换为我们的算法所需的输入格式。

通过在工作流内部完成工作，它始终处于运行自己实例的工作流客户的控制之下。

我们可以将优化部分放在我们实例上托管的一个微服务中。

为了更好地理解设置，请随意查看这个简短的演示

我希望这个教程和上面的示例已经给了你足够的洞察力，让你了解使用 n8n 在数据分析方面可以做什么。

请随意与我分享你对这种方法以及你认为如何改进以提高工作流性能的看法。

## 关于我

让我们在[领英](https://www.linkedin.com/in/samir-saci/)和[Twitter](https://twitter.com/Samir_Saci_)上建立联系。我是一名供应链工程师，使用数据分析来改善物流运营并降低成本。

如需咨询或关于分析和可持续供应链转型的建议，请通过[Logigreen Consulting](https://www.logi-green.com/)联系我。

查找你的供应链分析完整指南：[Analytics Cheat Sheet](https://bit.ly/supply-chain-cheat)。

如果你对数据分析和对供应链感兴趣，请查看我的网站。

[**Samir Saci | 数据科学 & 效率**](https://samirsaci.com/)

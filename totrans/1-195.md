# 使用大型语言模型进行时间序列分析的提示工程

> 原文：[`towardsdatascience.com/prompt-engineering-for-time-series-analysis-with-large-language-models/`](https://towardsdatascience.com/prompt-engineering-for-time-series-analysis-with-large-language-models/)

<mdspan datatext="el1760556011037" class="mdspan-comment">与**时间序列**数据一起工作通常与常规分析不同，主要是因为每个数据科学家最终都会遇到的**挑战**有关时间依赖性。

如果你能仅通过**正确的提示**来加速并提高你的分析，会怎么样？

大型语言模型（**LLMs**）已经成为了时间序列分析的一个颠覆者。如果你将 LLM 与**智能**提示工程相结合，它们可以开启分析师尚未尝试的方法的大门。

他们擅长发现模式、检测异常和做出预测。

本指南汇集了从简单数据准备到高级模型验证的经过验证的策略。到结束时，你将拥有实用的工具，让你**领先一步**。

这里的一切都有**研究**和现实世界的例子支持，所以你将带着实用的工具离开，而不仅仅是理论！

这是两篇系列文章中的第一篇，探讨了提示工程如何提升你的时间序列分析：

+   **第一部分：** 时间序列核心策略的提示（本文）

+   **第二部分：** [高级模型开发的提示](https://towardsdatascience.com/llm-powered-time-series-analysis/)

👉 本文中的所有**提示**都可在本文末尾作为速查表获取 😉

在本文中：

1.  核心时间序列提示工程策略

1.  时间序列预处理和分析的提示

1.  使用 LLM 进行异常检测

1.  时间依赖数据特征工程

1.  提示工程**速查表**！

* * *

## 1. 核心时间序列提示工程策略

### 1.1 基于补丁的提示进行预测

**Patch Instruct 框架**

一个好方法是将时间序列分解成重叠的“补丁”，并使用结构化提示将这些补丁输入 LLM。这种方法称为*PatchInstruct*，非常有效，并且保持了大约相同的准确性。

**示例实现：**

```py
## System  
You are a time-series forecasting expert in meteorology and sequential modeling.  
Input: overlapping patches of size 3, reverse chronological (most recent first).  

## User  
Patches:  
- Patch 1: [8.35, 8.36, 8.32]  
- Patch 2: [8.45, 8.35, 8.25]  
- Patch 3: [8.55, 8.45, 8.40]  
...  
- Patch N: [7.85, 7.95, 8.05]  

## Task  
1\. Forecast next 3 values.  
2\. In ≤40 words, explain recent trend.  

## Constraints  
- Output: Markdown list, 2 decimals.  
- Ensure predictions align with observed trend.  

## Example  
- Input: [5.0, 5.1, 5.2] → Output: [5.3, 5.4, 5.5].  

## Evaluation Hook  
Add: "Confidence: X/10\. Assumptions: [...]". 
```

**为什么它有效：**

+   LLM 会注意到数据中的短期时间模式。

+   使用比原始数据转储更少的标记（因此，成本更低）。

+   保持可解释性，因为你可以稍后重建补丁。

* * *

### 1.2 基于上下文的零样本提示

让我们想象你需要一个快速的**基线预测**。

**零样本**提示与**上下文**相结合适用于此。你只需向模型提供一个关于数据集、频率和预测范围的清晰描述，它就可以在没有额外训练的情况下识别模式！

```py
## System  
You are a time-series analysis expert specializing in [domain].  
Your task is to identify patterns, trends, and seasonality to forecast accurately.  

## User  
Analyze this time series: [x1, x2, ..., x96]  
- Dataset: [Weather/Traffic/Sales/etc.]  
- Frequency: [Daily/Hourly/etc.]  
- Features: [List features]  
- Horizon: [Number] periods ahead  

## Task  
1\. Forecast [Number] periods ahead.  
2\. Note key seasonal or trend patterns.  

## Constraints  
- Output: Markdown list of predictions (2 decimals).  
- Add ≤40-word explanation of drivers.  

## Evaluation Hook  
End with: "Confidence: X/10\. Assumptions: [...]". 
```

### 1.3 邻域增强提示

有时候，一个时间序列不足以满足需求。我们可以添加“邻近”的相似时间序列，然后 LLM 能够发现共同的结构并提高预测的准确性：

```py
## System  
You are a time-series analyst with access to 5 similar historical series.  
Use these neighbors to identify shared patterns and refine predictions.  

## User  
Target series: [current time series data]  

Neighbors:  
- Series 1: [ ... ]  
- Series 2: [ ... ]  
...  

## Task  
1\. Predict the next [h] values of the target.  
2\. Explain in ≤40 words how neighbors influenced the forecast.  

## Constraints  
- Output: Markdown list of [h] predictions (2 decimals).  
- Highlight any divergences from neighbors.  

## Evaluation Hook  
End with: "Confidence: X/10\. Assumptions: [...]". 
```

* * *

## 2. 时间序列预处理和分析提示

### 2.1 稳定性测试和转换

数据科学家在建模时间序列数据之前必须做的第一件事是检查序列是否**平稳**。

如果不是，他们需要应用如差分、对数或 Box-Cox 等转换。

**测试平稳性和应用转换的提示**

```py
## System  
You are a time-series analyst.  

## User  
Dataset: [N] observations  
- Time period: [specify]  
- Frequency: [specify]  
- Suspected trend: [linear / non-linear / seasonal]  
- Business context: [domain]  

## Task  
1\. Explain how to test for stationarity using:  
   - Augmented Dickey-Fuller  
   - KPSS  
   - Visual inspection  
2\. If non-stationary, suggest transformations: differencing, log, Box-Cox.  
3\. Provide Python code (statsmodels + pandas).  

## Constraints  
- Keep explanation ≤120 words.  
- Code should be copy-paste ready.  

## Evaluation Hook  
End with: "Confidence: X/10\. Assumptions: [...]". 
```

* * *

### 2.2 自相关和滞后特征分析

时间序列中的**自相关**衡量当前值在不同滞后下与其自身过去值的关联强度。

通过合适的图表（**ACF/PACF**），你可以观察到最重要的滞后，并围绕它们构建特征。

**自相关提示**

```py
## System  
You are a time-series expert.    

## User  
Dataset: [brief description]  
- Length: [N] observations  
- Frequency: [daily/hourly/etc.]  
- Raw sample: [first 20–30 values]  

## Task  
1\. Provide Python code to generate ACF & PACF plots.  
2\. Explain how to interpret:  
   - AR lags  
   - MA components  
   - Seasonal patterns  
3\. Recommend lag features based on significant lags.  
4\. Show Python code to engineer these lags (handle missing values).  

## Constraints  
- Output: ≤150 words explanation + Python snippets.  
- Use statsmodels + pandas.  

## Evaluation Hook  
End with: "Confidence: X/10\. Key lags flagged: [list]". 
```

* * *

### 2.3 季节分解和趋势分析

分解帮助你看到数据背后的故事，并帮助你在不同的**层**（趋势、季节性和残差）中看到它。

**季节分解提示**

```py
## System  
You are a time-series expert.   

## User  
Data: [time series]  
- Suspected seasonality: [daily/weekly/yearly]  
- Business context: [domain]  

## Task  
1\. Apply STL decomposition.  
2\. Compute:  
   - Seasonal strength Qs = 1 - Var(Residual)/Var(Seasonal+Residual)  
   - Trend strength Qt = 1 - Var(Residual)/Var(Trend+Residual)  
3\. Interpret trend & seasonality for business insights.  
4\. Recommend modeling approaches.  
5\. Provide Python code for visualization.  

## Constraints  
- Keep explanation ≤150 words.  
- Code should use statsmodels + matplotlib.  

## Evaluation Hook  
End with: "Confidence: X/10\. Key business implications: [...]". 
```

* * *

## 3. 使用 LLMs 进行异常检测

### 3.1 直接提示进行异常检测

时间序列中的异常检测通常不是一个有趣的任务，并且需要很多时间。

LLMs 可以像一位警觉的分析师一样，在你的数据中识别出外部值。

**异常检测提示**

```py
## System  
You are a senior data scientist specializing in time-series anomaly detection.  

## User  
Context:  
- Domain: [Financial/IoT/Healthcare/etc.]  
- Normal operating range: [specify if known]  
- Time period: [specify]  
- Sampling frequency: [specify]  
- Data: [time series values]  

## Task  
1\. Detect anomalies with timestamps/indices.  
2\. Classify as:  
   - Point anomalies  
   - Contextual anomalies  
   - Collective anomalies  
3\. Assign confidence scores (1–10).  
4\. Explain reasoning for each detection.  
5\. Suggest potential causes (domain-specific).  

## Constraints  
- Output: Markdown table (columns: Index, Type, Confidence, Explanation, Possible Cause).  
- Keep narrative ≤150 words.  

## Evaluation Hook  
End with: "Overall confidence: X/10\. Further data needed: [...]". 
```

* * *

### 3.2 基于预测的异常检测

而不是直接查看异常，另一种聪明的策略是首先预测“应该”发生什么，然后衡量现实与这些期望之间的偏差。

这些偏差可以突出那些在其他方式下不会显眼的**异常**。

这里有一个现成的提示你可以尝试使用：

```py
## System  
You are an expert in forecasting-based anomaly detection.  

## User  
- Historical data: [time series]  
- Forecast horizon: [N periods]  

## Method  
1\. Forecast the next [N] periods.  
2\. Compare actual vs forecasted values.  
3\. Compute residuals (errors).  
4\. Flag anomalies where |actual - forecast| > threshold.  
5\. Use z-score & IQR methods to set thresholds.  

## Task  
Provide:  
- Forecasted values  
- 95% prediction intervals  
- Anomaly flags with severity levels  
- Recommended threshold values  

## Constraints  
- Output: Markdown table (columns: Period, Forecast, Interval, Actual, Residual, Anomaly Flag, Severity).  
- Keep explanation ≤120 words.  

## Evaluation Hook  
End with: "Confidence: X/10\. Threshold method used: [z-score/IQR]". 
```

* * *

## 4. 时间依赖数据的特征工程

智能特征可以成就或毁掉你的模型。

选项实在太多了：滞后到滚动窗口、周期性特征和外部变量。你可以添加很多内容来捕捉时间依赖性。

### 4.1 自动特征创建

真正的魔法发生在你工程化有意义**特征**来捕捉**趋势**、**季节性**和**时间动态**的时候。LLMs 实际上可以通过为你生成一系列有用的特征来自动化这个过程。

**综合特征工程提示：**

```py
## System  
You are a feature engineering expert for time series.  

## User  
Dataset: [description]  
- Target variable: [specify]  
- Temporal granularity: [hourly/daily/etc.]  
- Business domain: [context]  

## Task  
Create temporal features across 5 categories:  
1\. **Lag Features**  
   - Simple lags, seasonal lags, cross-variable lags  
2\. **Rolling Window Features**  
   - Moving averages, std/min/max, quantiles  
3\. **Time-based Features**  
   - Hour, day, month, quarter, year, DOW, WOY, is_weekend, is_holiday, time since events  
4\. **Seasonal & Cyclical Features**  
   - Fourier terms, sine/cosine transforms, interactions  
5\. **Change-based Features**  
   - Differences, pct changes, volatility measures  

## Constraints  
- Output: Python code using pandas/numpy.  
- Add short guidance on feature selection (importance/collinearity).  

## Evaluation Hook  
End with: "Confidence: X/10\. Features most impactful for [domain]: [...]". 
```

### 4.2 外部变量集成

有可能目标序列不足以解释整个故事。

有一些**外部因素**经常影响我们的数据，比如天气、经济指标或特殊事件。它们可以提供上下文并改善预测。

关键在于知道如何正确地整合它们，而不会破坏时间规则。以下是一个将外生变量纳入分析的提示。

**外生变量提示**：

```py
## System  
You are a time-series modeling expert.  
Task: Integrate external variables (exogenous features) into a forecasting pipeline.  

## User  
Primary series: [target variable]  
External variables: [list]  
Data availability: [past only / future known / mixed]  

## Task  
1\. Assess variable relevance (correlation, cross-correlation).  
2\. Align frequencies and handle resampling.  
3\. Create interaction features between external & target.  
4\. Apply time-aware cross-validation.  
5\. Select features suited for time-series models.  
6\. Handle missing values in external variables.  

## Constraints  
- Output: Python code for  
  - Data alignment & resampling  
  - Cross-correlation analysis  
  - Feature engineering with external vars  
  - Model integration:  
    - ARIMA (with exogenous vars)  
    - Prophet (with regressors)  
    - ML models (with external features)  

## Evaluation Hook  
End with: "Confidence: X/10\. Most impactful external variables: [...]". 
```

## 最后的想法

希望这个指南给你带来了很多可以消化和尝试的内容。

这是一个充满研究技术的工具箱，用于在时间序列分析中使用 LLMs。

**时间序列数据成功**的关键在于我们尊重时间数据的特性，精心设计提示以突出这些特性，并使用正确的评估方法进行验证。

感谢阅读！请期待第二部分 😉

* * *

👉 在**[Sara 的 AI 自动化摘要](https://saranfn.substack.com/)**中**获取完整的提示表单** — 每周帮助技术专业人士用 AI 自动化实际工作。您还将获得访问 AI 工具库的权限。

我在这里提供关于职业成长和转型的**指导**[链接](https://topmate.io/sara_nobrega)。

**如果您想支持我的工作**，您可以**购买我喜欢的咖啡**[链接](https://buymeacoffee.com/saranobregu)：卡布奇诺。😊

* * *

## 参考文献

[用于预测分析和时间序列预测的 LLMs](https://www.rohan-paul.com/p/llms-for-predictive-analytics-and)

[用更少的努力实现更智能的时间序列预测](https://dzone.com/articles/llm-advantage-smarter-time-series-predictions)

[通过基于补丁的提示和分解使用 LLMs 进行时间序列预测](https://arxiv.org/html/2506.12953v1)

[时间序列中的 LLMs：转变 AI 中的数据分析](https://futureagi.com/blogs/time-series-data-analysis-2025)

[kdd.org/exploration_files/p109-Time_Series_Forecasting_with_LLMs.pdf](https://www.kdd.org/exploration_files/p109-Time_Series_Forecasting_with_LLMs.pdf)

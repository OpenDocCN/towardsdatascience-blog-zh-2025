# LLM 驱动的时序分析

> 原文：[`towardsdatascience.com/llm-powered-time-series-analysis/`](https://towardsdatascience.com/llm-powered-time-series-analysis/)

<mdspan datatext="el1762571011303" class="mdspan-comment">处理**时间序列**数据总是带来它自己的一套**谜题**。每个数据科学家最终都会遇到那个墙，传统方法开始感觉……有限。

但如果你能通过构建、调整和验证高级预测模型来超越这些限制，只使用**正确的提示**呢？

大型语言模型（**LLMs**）正在改变时间序列建模的游戏规则。当你将它们与智能、结构化的提示工程相结合时，它们可以帮助你探索大多数分析师尚未考虑的方法。

它们可以引导你完成 ARIMA 设置、Prophet 调整，甚至 LSTMs 和 transformers 这样的深度学习架构。

本指南介绍了模型开发、验证和解释的高级提示技术。最后，你将拥有一套实用的提示集，帮助你更快、更有信心地构建、比较和微调模型。

这里的一切都基于**研究**和真实世界的示例，所以你将带着现成的工具离开。

这是两篇系列文章的第二篇，探讨了提示工程如何提升你的时间序列分析：

+   **第一部分：**[时间序列核心策略的提示](https://towardsdatascience.com/prompt-engineering-for-time-series-analysis-with-large-language-models/)

+   **第二部分：**高级模型开发的提示（本文）

👉 本文和上一篇文章中所有的**提示**都可在本文末尾作为速查表找到😉

在本文中：

1.  高级模型开发提示

1.  模型验证和解释的提示

1.  真实世界实施示例

1.  最佳实践和高级技巧

1.  提示工程**速查表**！

* * *

## 1. 高级模型开发提示

让我们从重量级选手开始。正如你可能知道的，ARIMA 和 Prophet 对于结构化和可解释的工作流程仍然很棒，而 LSTMs 和 transformers 在复杂、非线性动态方面表现出色。

最好的部分？有了正确的提示，你可以节省大量时间，因为 LLMs 变成了你的个人助理，可以设置、调整和检查每一步，而不会迷失方向。

### 1.1 ARIMA 模型选择和验证

在我们继续之前，让我们确保经典的基线是稳固的。使用下面的提示来识别正确的 ARIMA 结构，验证假设，并锁定一个值得信赖的预测流程，你可以将其与其他所有内容进行比较。

**全面的 ARIMA 建模提示：**

```py
"You are an expert time series modeler. Help me build and validate an ARIMA model:

Dataset: [description]
Data: [sample of time series]

Phase 1 - Model Identification:
1\. Test for stationarity (ADF, KPSS tests)
2\. Apply differencing if needed
3\. Plot ACF/PACF to determine initial (p,d,q) parameters
4\. Use information criteria (AIC, BIC) for model selection

Phase 2 - Model Estimation:
1\. Fit ARIMA(p,d,q) model
2\. Check parameter significance
3\. Validate model assumptions:
   - Residual analysis (white noise, normality)
   - Ljung-Box test for autocorrelation
   - Jarque-Bera test for normality

Phase 3 - Forecasting & Evaluation:
1\. Generate forecasts with confidence intervals
2\. Calculate forecast accuracy metrics (MAE, MAPE, RMSE)
3\. Perform walk-forward validation

Provide complete Python code with explanations."
```

### 1.2 Prophet 模型配置

有已知的节假日、清晰的季节性节奏或你希望“优雅处理”的转折点吗？Prophet 是你的朋友。

下面的提示框定了业务背景，调整季节性，并构建了一个交叉验证设置，以便你可以在生产中信任输出结果。

**Prophet 模型设置提示**：

```py
"As a Facebook Prophet expert, help me configure and tune a Prophet model:

Business context: [specify domain]
Data characteristics:
- Frequency: [daily/weekly/etc.]
- Historical period: [time range]
- Known seasonalities: [daily/weekly/yearly]
- Holiday effects: [relevant holidays]
- Trend changes: [known changepoints]

Configuration tasks:
1\. Data preprocessing for Prophet format
2\. Seasonality configuration:
   - Yearly, weekly, daily seasonality settings
   - Custom seasonal components if needed
3\. Holiday modeling for [country/region]
4\. Changepoint detection and prior settings
5\. Uncertainty interval configuration
6\. Cross-validation setup for hyperparameter tuning

Sample data: [provide time series]

Provide Prophet model code with parameter explanations and validation approach." 
```

### 1.3 LSTM 和深度学习模型指导

当您的序列混乱、非线性或具有长期交互的多变量时，是时候升级了。

使用下面的 LSTM 提示来构建从预处理到训练技巧的端到端深度学习管道，这些技巧可以从概念验证扩展到生产。

**LSTM 架构设计提示**：

```py
"You are a deep learning expert specializing in time series. Design an LSTM architecture for my forecasting problem:

Problem specifications:
- Input sequence length: [lookback window]
- Forecast horizon: [prediction steps]
- Features: [number and types]
- Dataset size: [training samples]
- Computational constraints: [if any]

Architecture considerations:
1\. Number of LSTM layers and units per layer
2\. Dropout and regularization strategies
3\. Input/output shapes for multivariate series
4\. Activation functions and optimization
5\. Loss function selection
6\. Early stopping and learning rate scheduling

Provide:
- TensorFlow/Keras implementation
- Data preprocessing pipeline
- Training loop with validation
- Evaluation metrics calculation
- Hyperparameter tuning suggestions" 
```

## 2. 模型验证和解释

您知道，优秀的模型既准确又可靠，并且可解释。

这一部分帮助您对性能进行压力测试，并揭示模型真正学习的内容。从稳健的交叉验证开始，然后深入诊断，以便您能够信任数字背后的故事。

### 2.1 时间序列交叉验证

**向前验证提示**：

```py
"Design a robust validation strategy for my time series model:

Model type: [ARIMA/Prophet/ML/Deep Learning]
Dataset: [size and time span]
Forecast horizon: [short/medium/long term]
Business requirements: [update frequency, lead time needs]

Validation approach:
1\. Time series split (no random shuffling)
2\. Expanding window vs sliding window analysis
3\. Multiple forecast origins testing
4\. Seasonal validation considerations
5\. Performance metrics selection:
   - Scale-dependent: MAE, MSE, RMSE
   - Percentage errors: MAPE, sMAPE  
   - Scaled errors: MASE
   - Distributional accuracy: CRPS

Provide Python implementation for:
- Cross-validation splitters
- Metrics calculation functions
- Performance comparison across validation folds
- Statistical significance testing for model comparison" 
```

### 2.2 模型解释和诊断

残差是否干净？区间是否校准？哪些特征重要？下面的提示为您提供了一个全面的诊断路径，以便您的模型值得信赖。

**全面模型诊断提示**：

```py
"Perform thorough diagnostics for my time series model:

Model: [specify type and parameters]
Predictions: [forecast results]
Residuals: [model residuals]

Diagnostic tests:
1\. Residual Analysis:
   - Autocorrelation of residuals (Ljung-Box test)
   - Normality tests (Shapiro-Wilk, Jarque-Bera)
   - Heteroscedasticity tests
   - Independence assumption validation

2\. Model Adequacy:
   - In-sample vs out-of-sample performance
   - Forecast bias analysis
   - Prediction interval coverage
   - Seasonal pattern capture assessment

3\. Business Validation:
   - Economic significance of forecasts
   - Directional accuracy
   - Peak/trough prediction capability
   - Trend change detection

4\. Interpretability:
   - Feature importance (for ML models)
   - Component analysis (for decomposition models)
   - Attention weights (for transformer models)

Provide diagnostic code and interpretation guidelines." 
```

* * *

## 3. 真实世界实现示例

因此，我们已经探讨了提示如何指导您的建模工作流程，但您实际上如何使用它们呢？

我现在将向您展示一个快速且可重复的例子，说明您如何在训练时间序列模型后，立即在您的**自己的笔记本**中使用其中一个**提示**。

这个想法很简单：我们将从这篇文章中选取一个提示（*向前验证提示*），发送到**OpenAI API**，并让一个 LLM 在你的分析工作流程中直接给出**反馈**或代码建议。

**步骤 1：创建一个小的辅助函数来向 API 发送提示**

这个函数，`ask_llm()`，使用您的**API 密钥**连接到**OpenAI**的响应 API，并发送提示的内容。

不要忘记您的`OPENAI_API_KEY`！在运行之前，您应该在环境变量中保存它。

之后，您可以选择丢弃文章中的任何提示，并获得建议，甚至可以直接运行的代码。

```py
# %pip -q install openai  # Only if you don't already have the SDK

import os
from openai import OpenAI

def ask_llm(prompt_text, model="gpt-4.1-mini"):
    """
    Sends a single-user-message prompt to the Responses API and returns text.
    Switch 'model' to any available text model in your account.
    """
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        print("Set OPENAI_API_KEY to enable LLM calls. Skipping.")
        return None

    client = OpenAI(api_key=api_key)
    resp = client.responses.create(
        model=model,
        input=[{"role": "user", "content": prompt_text}]
    )
    return getattr(resp, "output_text", None) 
```

假设您的模型已经训练好，您可以用简单的英语描述您的设置，并通过提示模板发送。

在这种情况下，我们将使用**向前验证提示**来让 LLM 为您生成一个稳健的验证方法和相关的代码想法。

```py
walk_forward_prompt = f"""
Design a robust validation strategy for my time series model:

Model type: ARIMA/Prophet/ML/Deep Learning (we used SARIMAX with exogenous regressors)
Dataset: Daily synthetic retail sales; 730 rows from 2022-01-01 to 2024-12-31
Forecast horizon: 14 days
Business requirements: short-term accuracy, weekly update cadence

Validation approach:
1\. Time series split (no random shuffling)
2\. Expanding window vs sliding window analysis
3\. Multiple forecast origins testing
4\. Seasonal validation considerations
5\. Performance metrics selection:
   - Scale-dependent: MAE, MSE, RMSE
   - Percentage errors: MAPE, sMAPE
   - Scaled errors: MASE
   - Distributional accuracy: CRPS

Provide Python implementation for:
- Cross-validation splitters
- Metrics calculation functions
- Performance comparison across validation folds
- Statistical significance testing for model comparison
"""

wf_advice = ask_llm(walk_forward_prompt)
print(wf_advice or "(LLM call skipped)") 
```

一旦运行这个单元格，LLM 的响应将直接出现在您的笔记本中，通常是一个简短的指南或代码片段，您可以复制、修改并测试。

这是一个简单的**工作流程**，但出人意料地强大：您不需要在文档和实验之间切换上下文，而是直接将模型循环到您的笔记本中。

您可以使用任何早期的提示重复这个相同的模式，例如，用**全面模型诊断提示**替换，让 LLM 解释您的残差或为您的预测提出改进建议。

## 4. 最佳实践和高级技巧

### 4.1 提示优化策略

**迭代提示细化**：

1.  从基本的提示开始，逐渐增加复杂性，不要一开始就试图做到完美。

1.  测试不同的提示结构（角色扮演与直接指令等）

1.  使用不同的数据集验证提示的有效性

1.  使用相关示例进行少样本学习

1.  总是添加领域知识和业务背景！

**关于令牌效率（如果成本是考虑因素）：**

+   在信息完整性和令牌使用之间保持平衡

+   使用基于补丁的方法来减少输入大小

+   实现提示缓存以处理重复模式

+   与你的团队考虑准确性和计算成本之间的权衡

不要忘记进行大量的诊断，以确保你的结果值得信赖，并且随着数据和业务问题的演变或变化，持续优化你的提示。记住，这是一个迭代的过程，而不是试图一开始就达到完美。

感谢阅读！

* * *

👉 **订阅** **[Sara 的 AI 自动化摘要](https://saranfn.substack.com/) **以获取完整的提示表单——** *帮助技术专业人士每周使用 AI 自动化实际工作。* 你还将获得访问 AI 工具库的权限。

我在这里提供 **职业成长和转型** [指导](https://topmate.io/sara_nobrega)。

**如果你想支持我的工作**，你可以 [买我喜欢的**咖啡**](https://buymeacoffee.com/saranobregu)：一杯卡布奇诺。

* * *

## 参考文献

[MingyuJ666/Time-Series-Forecasting-with-LLMs: [KDD Explore’24]Time Series Forecasting with LLMs: Understanding and Enhancing Model Capabilities](https://github.com/mingyuj666/time-series-forecasting-with-llms)

[LLMs 用于预测分析和时间序列预测](https://www.rohan-paul.com/p/llms-for-predictive-analytics-and)

[用更少的努力实现更智能的时间序列预测](https://dzone.com/articles/llm-advantage-smarter-time-series-predictions)

[通过基于补丁的提示和分解进行 LLMs 的时间序列预测](https://arxiv.org/html/2506.12953v1)

[LLMs 在时间序列中：转变 AI 中的数据分析](https://futureagi.com/blogs/time-series-data-analysis-2025)

[kdd.org/exploration_files/p109-Time_Series_Forecasting_with_LLMs.pdf](https://www.kdd.org/exploration_files/p109-Time_Series_Forecasting_with_LLMs.pdf)

# 如何在两个具有统计学显著性的回归模型之间进行区分

> 原文：[`towardsdatascience.com/how-to-tell-among-two-regression-models-with-statistical-significance-e58d8ce5af17/`](https://towardsdatascience.com/how-to-tell-among-two-regression-models-with-statistical-significance-e58d8ce5af17/)

## 引言

在分析数据时，人们经常需要比较两个回归模型以确定哪个模型最适合一块数据。通常，一个模型是**更复杂**模型的一个**更简单**版本，该模型包含额外的参数。然而，更多的参数并不总是保证更复杂的模型实际上更好，因为它们可能只是过度拟合了数据。

要确定增加的复杂性是否具有**统计学上的显著性**，我们可以使用所谓的**嵌套模型的 F 检验**。这种统计技术评估由于额外参数导致的残差平方和（RSS）的减少是否有意义，或者仅仅是偶然发生的。

在这篇文章中，我解释了嵌套模型的 F 检验，然后我展示了一个逐步算法，使用伪代码演示了其实现，并提供了一段 Matlab 代码，您可以立即运行或在自己的系统中重新实现（在这里我选择了 Matlab，因为它让我快速访问统计和拟合函数，我不想在这些函数上浪费时间）。在整个文章中，我们将看到嵌套模型的 F 检验在几个设置中的应用示例，包括我构建到示例 Matlab 代码中的某些示例。

## 嵌套模型的 F 检验

### 什么是嵌套模型？

如果一个简单模型是更复杂模型的特殊情况，则两个模型是**嵌套的**。换句话说，简单模型可以通过将一些参数设置为零或固定它们，或者简单地通过删除一个或多个项从复杂模型中获得。

例如，[在这篇论文](https://www.nature.com/articles/s42004-024-01127-0)中，作者们遵循了一个光谱可观测量（来自[NMR 实验，了解更多信息请点击这里](https://medium.com/advances-in-biological-science/nuclear-magnetic-resonance-spectroscopy-the-overlooked-powerhouse-of-biology-chapter-1-7b3d74dad714))与浓度的关系，并且他们发现大多数情况下这种依赖性是线性的，但在某些情况下它是由线性加双曲饱和结合形状的组合：

![图表由作者根据这篇论文改编，其许可允许重用。](img/2ce063732bef30c91fd34ba57ad0de2d.png)

图表由作者根据[这篇论文](https://www.nature.com/articles/s42004-024-01127-0)改编，其许可允许重用。

这意味着可以拟合一个看起来就像线性回归的**简单模型**：

![](img/c84415265576d1d3e0f8d3217b652313.png)

或者更复杂，通过添加结合项来扩展更简单的模型：

![](img/9cd9293e143fb884765fb8e9f0794c46.png)

在这种情况下，简单模型有两个参数，复杂模型在扩展自线性项的项中有两个额外的参数。如果你移除双曲绑定项，复杂模型将简化为简单模型，使得这两个模型是嵌套的。

### F 检验是如何工作的

F 检验比较两个模型的残差平方和（RSS），以检验复杂模型实现的 RSS 减少是否具有统计学意义。RSS 衡量观测数据与拟合模型之间的总平方差，我们使用[F 统计量（斯涅德科尔的 F）](https://en.wikipedia.org/wiki/F-distribution)来比较每个模型的 RSS，计算如下：

![图片](img/7a300ff6e76da05241278aed3f83f4c1.png)

其中…

+   *RSS 1* 和 *RSS 2* 分别是简单模型（1）和复杂模型（2）的 RSS（注意按照这个顺序减去确保差值等于或大于零）

+   *p2* 和 *p1* 是每个模型的参数数量，这里分别是 4 和 2。

+   *n* 是我们拟合的点数。

F 统计量遵循具有（分子）和（分母）自由度的 F 分布，我们可以在表中查找或在大多数统计软件和库中的函数中获取。从这些数据中，可以得到 p 值以确定统计显著性。

### 解释 F 检验

没有什么异常。如果**p 值 < 某个阈值**（例如，0.05），则复杂模型与简单模型相比，对数据的拟合具有统计学意义。否则，简单模型就足够了。

顺便说一句，到这一点，你可能对查看这篇关于统计显著性的文章感兴趣：

> [**重新思考统计显著性**](https://towardsdatascience.com/rethinking-statistical-significance-a6150f588b9a)

## 模型比较算法

比较两个嵌套模型的逐步算法如下：

1.  **定义两个模型**：具有较少参数的简单模型；具有附加参数的复杂模型，在某些情况下包括简单模型。

1.  **将两个模型拟合到数据中**以估计它们的参数并计算它们的 RSS。

1.  **从 RSS 值计算 F 统计量**，如指示。

1.  **使用 F 分布计算 p 值**并检查显著性。

### 伪代码

以下伪代码概述了比较两个嵌套模型的逻辑。我根据上面提到的论文中的示例实现了这段伪代码。

```py
Input: x (independent variable), y (dependent variable)
Output: Parameters of both models, p-value, and decision

1\. Define Simple Model: y_simple = a*x + h
2\. Define Complex Model: y_complex = a*x + h + c*x / (d + x)
3\. Fit the Simple Model to the data (x, y) to find parameters and compute RSS1.
4\. Fit the Complex Model to the data (x, y) to find parameters and compute RSS2.
5\. Compute F-statistic:
      F = ((RSS1 - RSS2) / (p2 - p1)) / (RSS2 / (n - p2))
6\. Compute p-value from the F-distribution with (p2 - p1, n - p2) degrees of freedom.
7\. If p-value < alpha (e.g., 0.05):
       Print "The complex model is significantly better."
   Else:
       Print "The simpler model is sufficient."
8\. Plot the data with both model fits for visualization (optional).
```

## MATLAB 代码实现

我知道 Matlab 在这里并不流行，但在这个特定情况下，它对我正在处理的问题来说很方便，因为它拥有我需要的所有库和函数来进行统计和拟合，无需进行任何特殊的调用或源库。这样，我可以专注于 F 检验本身。

但当然，你可以自由地将其适应你喜欢的编程语言、统计软件包或库！

这里是算法的完整 Matlab 实现，以一个名为 _compare*models*的函数形式呈现，该函数确实就是做这件事（以及一个负责拟合的函数）：

```py
function compare_models(x, y)
    % Fit Simple Model: y = a*x + h
    simple_model = @(b, x) b(1)*x + b(2);
    b0_simple = [1; 0];
    [params_simple, RSS1] = fit_model(simple_model, b0_simple, x, y);

    % Fit Complex Model: y = a*x + h + c*x / (d + x)
    complex_model = @(b, x) b(1)*x + b(2) + b(3)*x ./ (b(4) + x);
    b0_complex = [1; 0; 1; 1];
    [params_complex, RSS2] = fit_model(complex_model, b0_complex, x, y);

    % Degrees of freedom
    n = length(y);
    p1 = 2; p2 = 4;
    df1 = p2 - p1; df2 = n - p2;

    % Compute F-statistic and p-value
    F = ((RSS1 - RSS2) / df1) / (RSS2 / df2);
    p_value = 1 - fcdf(F, df1, df2);

    % Print results
    fprintf('Simple Model Parameters: a = %.4f, h = %.4fn', params_simple(1), params_simple(2));
    fprintf('Complex Model Parameters: a = %.4f, h = %.4f, c = %.4f, d = %.4fn', ...
        params_complex(1), params_complex(2), params_complex(3), params_complex(4));
    fprintf('F-statistic: %.4f, p-value: %.4en', F, p_value);

    % Decision
    if p_value < 0.05
        fprintf('The complex model is significantly better.n');
    else
        fprintf('The simpler model is sufficient.n');
    end
end

%% Auxiliary function that runs the least-squares fitting procedure
function [params, RSS] = fit_model(model, b0, x, y)
    opts = optimset('Display', 'off');
    [params, ~] = lsqcurvefit(model, b0, x, y, [], [], opts);
    residuals = y - model(params, x);
    RSS = sum(residuals.²);
end
```

### 示例用法

我们可以在两个向量中准备一些示例数据，然后只需调用函数并传递两个参数：

```py
x = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]';
y = [0, 2.1, 4.1, 6.3, 8.1, 10.2, 12.0, 13.9, 15.1, 17.2, 19.0]';

compare_models(x, y);
```

输出将类似于以下内容：

```py
Simple Model Parameters: a = 2.1234, h = 1.2345
Complex Model Parameters: a = 1.9876, h = 1.4567, c = 5.6789, d = 2.3456
F-statistic: 12.3456, p-value: 1.23e-03
The complex model is significantly better.
```

### 一些更多示例

我们可以轻松创建 Matlab 代码，不仅会返回复杂模型是否显著更好，还会显示一个图表，以便我们作为专家可以批判性地检查情况：

![作者截图，展示了如何将一条线和一条线加双曲绑定拟合数据点。](img/c0e266748f24f3122fc82519e1c08892.png)

作者截图，展示了如何将一条线和一条线加双曲绑定拟合数据点。

这里是那个确切示例的完整代码，为了使所有内容更加紧凑和直接，我没有展示之前定义的函数：

```py
clc
clear
close all

% Example data
x = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]';

y = 1.*x + 3 + 22.*x./(x+1);

% Plot original data
figure;
plot(x, y, 'k.', 'MarkerSize', 15); % Original data in black
hold on;
xlabel('x');
ylabel('y');
title('Model Comparison: Simple vs Complex');
grid on;

% Run the comparison
[params_simple, params_complex, p_value, is_significant] = compare_fits(x, y);

% Generate fitted data for the two models
x_fit = linspace(min(x), max(x), 100)'; % Fine grid for smooth curve

% Simple model: y = a*x + h
y_simple = params_simple(1) * x_fit + params_simple(2);

% Complex model: y = a*x + h + c*x/(d+x)
y_complex = params_complex(1) * x_fit + params_complex(2) + ...
            params_complex(3) * x_fit ./ (params_complex(4) + x_fit);

% Overlay the fitted models
plot(x_fit, y_simple, 'r-', 'LineWidth', 2, 'DisplayName', 'Simple Model: y = a*x + h'); % Red curve
plot(x_fit, y_complex, 'g-', 'LineWidth', 2, 'DisplayName', 'Complex Model: y = a*x + h + c*x/(d+x)'); % Green curve

% Add legend
legend('Data', 'Simple Model', 'Complex Model', 'Location', 'best');
hold off;
```

## 通过拟合进行筛选

我们在这里看到的是作为一种方法提出，用以统计显著地判断一个模型是否比另一个更简单的模型更适合。然而，有一个更有趣的“次要”应用源于这个“直接”应用。事实上，正是这个次要应用促使我研究这个问题：你可以自动对各种(x, y)向量进行拟合，以自动检测趋势。

例如，在我开始这篇帖子时提到的例子中，可以对各种浓度和信号集进行拟合和 F 检验，以自动确定在哪些情况下可以检测到以统计显著拟合更复杂模型的形式的绑定。

很容易编辑我提供的 Matlab 代码，运行各种(x, y)集，并在更复杂模型在统计上更适合时打印标志，可能还会绘制数据和拟合，以便用户可以检查结果并基于专业知识做出判断：

![作者截图展示了 Matlab 程序自动检测数据集中哪些 y 向量与 x 的线性模型相比更适合的输出。](img/55daabf0effbfc6752fa8169091ab207.png)

作者截图展示了 Matlab 程序自动检测数据集中哪些 y 向量与 x 的线性模型相比更适合的输出。

## 结论

我们通过嵌套模型的 F 检验来到这里，这是一种强大的统计技术，它比较两个回归模型，并确定更复杂模型的增加复杂性是否“在统计上合理”。通过实施这种方法，你确保你的模型选择基于统计证据而不是主观偏好。

如果你喜欢这个，这里有一些我写的其他深入实践解释，你可能喜欢或觉得有用：

> [**使用蒙特卡洛模拟传播误差的强大和简单性**](https://towardsdatascience.com/the-power-and-simplicity-of-propagating-errors-with-monte-carlo-simulations-9c8dcca9d90d)
> 
> [**主成分分析 definitive 指南**](https://towardsdatascience.com/the-definitive-guide-to-principal-components-analysis-84cd73640302)
> 
> [**评估降维过程质量的框架**](https://lucianosphere.medium.com/a-framework-to-evaluate-the-quality-of-dimensionality-reduction-procedures-509f8f3520d0)
> 
> [**预测能力评分：计算、优点、缺点和 JavaScript 代码**](https://towardsdatascience.com/predictive-power-score-calculation-pros-cons-and-javascript-code-165ec4c593ca)

## 我的博客接下来会写些什么？

> 在评论中告诉我你希望在这个层面上涵盖的其他主题。
> 
> 我不能保证太多，但我将尽力处理你的建议！

## 你可能还喜欢的一些其他主题的文章

> [**通过显式考虑单位来改进符号回归方法**](https://towardsdatascience.com/a-better-symbolic-regression-method-by-explicitly-considering-units-35b3630165b)
> 
> [**Web 上数据可视化和分析的顶级库**](https://towardsdatascience.com/the-most-advanced-libraries-for-data-visualization-and-analysis-on-the-web-e823535e0eb1)
> 
> [**gLM2 和基因组学的多模态基础模型**](https://medium.com/advances-in-biological-science/glm2-and-multimodal-foundational-models-for-genomics-d53167bfd5d7)
> 
> [**科学家们认真对待模仿人类思维的巨型语言模型**](https://towardsdatascience.com/scientists-go-serious-about-large-language-models-mirroring-human-thinking-faa64a36ad71)

* * *

***[www.lucianoabriata.com](https://www.lucianoabriata.com/)** 我写关于我广泛兴趣范围内的所有事情：自然、科学、技术、编程等。**[成为 Medium 会员](https://lucianosphere.medium.com/membership)** 以访问所有故事（我从中获得的小额收入不会对你造成任何费用）并**[通过电子邮件订阅以获取我的新故事](https://lucianosphere.medium.com/subscribe)**。要**咨询小型工作**，请查看我的**[服务页面](https://lucianoabriata.altervista.org/services/index.html)**。你可以**[在此联系我](https://lucianoabriata.altervista.org/office/contact.html)**。

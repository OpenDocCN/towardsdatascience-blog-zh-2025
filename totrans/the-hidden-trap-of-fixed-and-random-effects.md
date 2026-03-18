# 固定效应和随机效应的隐藏陷阱

> 原文：[`towardsdatascience.com/the-hidden-trap-of-fixed-and-random-effects/`](https://towardsdatascience.com/the-hidden-trap-of-fixed-and-random-effects/)

## 随机效应和固定效应是什么？

在设计研究时，我们通常旨在将独立变量与那些对我们无兴趣的变量分离，以观察它们对因变量的真实影响。例如，假设我们想研究使用 Github Copilot（独立变量）对开发者生产力（因变量）的影响。一种方法是衡量开发者使用 Copilot 花费的时间以及他们完成编码任务的速度。乍一看，我们可能会观察到强烈的正相关：Copilot 使用越多，任务完成越快。

然而，其他因素也可能影响开发者完成工作速度的快慢。例如，公司 A 可能拥有更快的 CI/CD 流水线或处理更小、更简单的任务，而公司 B 可能需要漫长的代码审查或处理更复杂、耗时更长的任务。如果我们不考虑这些组织上的差异，我们可能会错误地得出结论，认为 Copilot 对 B 公司开发者的效果较差，尽管真正减慢他们速度的是环境，而不是 Copilot。

这类群体层面的变化——跨团队、公司或项目之间的差异——通常被称为**“随机效应”**或**“固定效应”**。

固定效应是我们感兴趣的变量，其中每个群体都通过独热编码单独处理。这样，由于组内变异被巧妙地包含在每个虚拟变量中，我们假设每个群体的方差是相似的，或者说是同质性的。

\[y_i = \beta_0 + \beta_1 x_i + \gamma_1 D_{1i} + \gamma_2 D_{2i} + \cdots + \varepsilon_i\]

其中 D[1i]、D[2i]、…分别代表群体 D[1i]、D[2i]、…的虚拟变量，γ₁、γ₂、…分别代表每个相应群体的固定效应系数。

相反，随机效应通常不是我们感兴趣的变量。我们假设每个群体是更广泛人群的一部分，每个群体的效应位于该人群更广泛的概率分布中。因此，每个群体的方差是非同质性的。

\[ y_{ij} = \beta_0 + \beta_1 x_{ij} + u_j + \varepsilon_{ij} \]

其中 u[j]是样本 i 中群体 j 的随机效应，从一个分布中抽取，通常是正态分布𝒩(0, σ²ᵤ)。

## 重新思考固定效应和随机效应

但是，如果你没有仔细思考这些效应实际上捕捉到的变化类型，只是随意地将这些效应插入到你的模型中，可能会误导你的分析。

我最近参与了一个分析[*AI 模型的环境影响*](https://towardsdatascience.com/rethinking-environmental-costs-of-training-ai-why-we-should-look-beyond-hardware/)的项目，我研究了 AI 模型的某些架构特征（参数数量、计算数量、数据集大小和训练时间）和硬件选择（硬件类型、硬件数量）如何影响训练过程中的能源消耗。我发现`训练时间`、`硬件数量`和`硬件类型`对能源消耗有显著影响。这种关系可以大致建模为：

\[ \text{能源} = \text{训练时间} + \text{硬件数量} + \text{硬件} \]

由于我认为组织之间可能存在差异，例如编码风格、代码结构或算法偏好，我相信将`组织`作为随机效应包括在内将有助于解释所有这些未观察到的潜在差异。为了测试我的假设，我比较了两个模型的结果：一个包含`组织`，另一个不包含`组织`，以查看哪一个更适合。在这两个模型中，因变量`能源`极度右偏斜，因此我应用了对数变换以稳定其方差。在这里，我使用了广义线性模型（GLM），因为我的数据分布并不正常。

```py
glm <- glm(
  log_Energy ~ Training_time_hour + 
               Hardware_quantity + 
               Training_hardware,
               data = df)
summary(glm)

glm_random_effects <- glmer(
  log_Energy ~ Training_time_hour + 
               Hardware_quantity + 
               Training_hardware + 
               (1 | Organization), // Random effects
               data = df)
summary(glm_random_effects)
AIC(glm_random_effects)
```

不包含`组织`的 GLM 模型产生了 AIC 值为 312.55，其中`训练时间`、`硬件数量`和某些类型的`硬件`在统计上显著。

```py
> summary(glm)

Call:
glm(formula = log_Energy ~ Training_time_hour + Hardware_quantity + 
    Training_hardware, data = df)

Coefficients:
                                                 Estimate Std. Error t value Pr(>|t|)    
(Intercept)                                     7.134e+00  1.393e+00   5.123 5.07e-06 ***
Training_time_hour                              1.509e-03  2.548e-04   5.922 3.08e-07 ***
Hardware_quantity                               3.674e-04  9.957e-05   3.690 0.000563 ***
Training_hardwareGoogle TPU v3                  1.887e+00  1.508e+00   1.251 0.216956    
Training_hardwareGoogle TPU v4                  3.270e+00  1.591e+00   2.055 0.045247 *  
Training_hardwareHuawei Ascend 910              2.702e+00  2.485e+00   1.087 0.282287    
Training_hardwareNVIDIA A100                    2.528e+00  1.511e+00   1.674 0.100562    
Training_hardwareNVIDIA A100 SXM4 40 GB         3.103e+00  1.750e+00   1.773 0.082409 .  
Training_hardwareNVIDIA A100 SXM4 80 GB         3.866e+00  1.745e+00   2.216 0.031366 *  
Training_hardwareNVIDIA GeForce GTX 285        -4.077e+00  2.412e+00  -1.690 0.097336 .  
Training_hardwareNVIDIA GeForce GTX TITAN X    -9.706e-01  1.969e+00  -0.493 0.624318    
Training_hardwareNVIDIA GTX Titan Black        -8.423e-01  2.415e+00  -0.349 0.728781    
Training_hardwareNVIDIA H100 SXM5 80GB          3.600e+00  1.864e+00   1.931 0.059248 .  
Training_hardwareNVIDIA P100                   -1.663e+00  1.899e+00  -0.876 0.385436    
Training_hardwareNVIDIA Quadro P600            -1.970e+00  2.419e+00  -0.814 0.419398    
Training_hardwareNVIDIA Quadro RTX 4000        -1.367e+00  2.424e+00  -0.564 0.575293    
Training_hardwareNVIDIA Quadro RTX 5000        -2.309e+00  2.418e+00  -0.955 0.344354    
Training_hardwareNVIDIA Tesla K80               1.761e+00  1.988e+00   0.886 0.380116    
Training_hardwareNVIDIA Tesla V100 DGXS 32 GB   3.415e+00  1.833e+00   1.863 0.068501 .  
Training_hardwareNVIDIA Tesla V100S PCIe 32 GB  3.698e+00  2.413e+00   1.532 0.131852    
Training_hardwareNVIDIA V100                   -3.638e-01  1.582e+00  -0.230 0.819087    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for gaussian family taken to be 3.877685)

    Null deviance: 901.45  on 69  degrees of freedom
Residual deviance: 190.01  on 49  degrees of freedom
AIC: 312.55

Number of Fisher Scoring iterations: 2
```

另一方面，包含`组织`的 GLM 模型产生了 AIC 值为 300.38，远低于之前的模型，表明模型拟合度更好。然而，当仔细观察时，我注意到一个显著的问题：其他变量的统计显著性已经消失，就像`组织`从它们那里夺走了显著性！

```py
> summary(glm_random_effects)
Linear mixed model fit by REML ['lmerMod']
Formula: log_Energy ~ Training_time_hour + Hardware_quantity + Training_hardware +  
    (1 | Organization)
   Data: df

REML criterion at convergence: 254.4

Scaled residuals: 
     Min       1Q   Median       3Q      Max 
-1.65549 -0.24100  0.01125  0.26555  1.51828 

Random effects:
 Groups       Name        Variance Std.Dev.
 Organization (Intercept) 3.775    1.943   
 Residual                 1.118    1.057   
Number of obs: 70, groups:  Organization, 44

Fixed effects:
                                                 Estimate Std. Error t value
(Intercept)                                     6.132e+00  1.170e+00   5.243
Training_time_hour                              1.354e-03  2.111e-04   6.411
Hardware_quantity                               3.477e-04  7.035e-05   4.942
Training_hardwareGoogle TPU v3                  2.949e+00  1.069e+00   2.758
Training_hardwareGoogle TPU v4                  2.863e+00  1.081e+00   2.648
Training_hardwareHuawei Ascend 910              4.086e+00  2.534e+00   1.613
Training_hardwareNVIDIA A100                    3.959e+00  1.299e+00   3.047
Training_hardwareNVIDIA A100 SXM4 40 GB         3.728e+00  1.551e+00   2.404
Training_hardwareNVIDIA A100 SXM4 80 GB         4.950e+00  1.478e+00   3.349
Training_hardwareNVIDIA GeForce GTX 285        -3.068e+00  2.502e+00  -1.226
Training_hardwareNVIDIA GeForce GTX TITAN X     4.503e-02  1.952e+00   0.023
Training_hardwareNVIDIA GTX Titan Black         2.375e-01  2.500e+00   0.095
Training_hardwareNVIDIA H100 SXM5 80GB          4.197e+00  1.552e+00   2.704
Training_hardwareNVIDIA P100                   -1.132e+00  1.512e+00  -0.749
Training_hardwareNVIDIA Quadro P600            -1.351e+00  1.904e+00  -0.710
Training_hardwareNVIDIA Quadro RTX 4000        -2.167e-01  2.503e+00  -0.087
Training_hardwareNVIDIA Quadro RTX 5000        -1.203e+00  2.501e+00  -0.481
Training_hardwareNVIDIA Tesla K80               1.559e+00  1.445e+00   1.079
Training_hardwareNVIDIA Tesla V100 DGXS 32 GB   3.751e+00  1.536e+00   2.443
Training_hardwareNVIDIA Tesla V100S PCIe 32 GB  3.487e+00  1.761e+00   1.980
Training_hardwareNVIDIA V100                    7.019e-01  1.434e+00   0.489

Correlation matrix not shown by default, as p = 21 > 12.
Use print(x, correlation=TRUE)  or
    vcov(x)        if you need it

fit warnings:
Some predictor variables are on very different scales: consider rescaling
> AIC(glm_random_effects)
[1] 300.3767
```

仔细思考后，这很有道理。某些组织可能始终倾向于特定的硬件类型，或者较大的组织可能能够负担得起更昂贵的硬件和资源来训练更大的 AI 模型。换句话说，这里的随机效应可能重叠并过度解释了我们可用独立变量的变化，因此它们吸收了我们试图研究的大部分内容。

这突出了一个重要的观点：虽然随机效应或固定效应是控制不希望出现的组间差异的有用工具，但它们也可能无意中捕捉到我们独立变量的潜在变化。在我们盲目地将这些效应引入模型，希望它们能够愉快地吸收所有噪声之前，我们应该仔细考虑这些效应真正代表的是什么。

* * *

参考文献：Steve Midway，R 中的数据分析，[`bookdown.org/steve_midway/DAR/random-effects.html`](https://bookdown.org/steve_midway/DAR/random-effects.html)

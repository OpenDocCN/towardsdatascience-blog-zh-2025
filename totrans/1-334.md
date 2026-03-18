# 随机微分方程和温度 — NASA 气候数据第二部分

> 原文：[`towardsdatascience.com/stochastic-differential-equations-and-temperature-nasa-climate-data-pt-2/`](https://towardsdatascience.com/stochastic-differential-equations-and-temperature-nasa-climate-data-pt-2/)

你可能会想知道为什么我们要将温度时间序列建模为 Ornstein-Uhlenbeck 过程，以及这是否有任何实际意义。好吧，它确实有。虽然 O-U 过程在物理学中用于模拟摩擦下的粒子速度，但它也用于金融中模拟利率、波动性或均值回归。这种温度应用源于一个我有个人经验领域。我的论文*“神经网络在天气衍生品定价中的应用*”专注于开发这些衍生品定价的新方法。

O-U 过程常用于天气衍生品的定价。这是一种金融衍生品，旨在根据基础气候指数（温度、降雨、风速、降雪）支付赔偿。这些衍生品使任何面临气候风险的人都能管理他们的风险敞口。这可能包括担心干旱的农民、担心供暖成本上升的能源公司或担心季节性零售商的糟糕季节。你可以在[天气衍生品](https://link.springer.com/book/10.1007/978-1-4614-6071-8)中了解更多信息，Antonis Alexandridis K.（我目前正在与他合作）和 Achilleas D. Zapranis。

随机微分方程（SDEs）出现在不确定性的任何地方。**Black-Scholes**、**薛定谔方程**和**几何布朗运动**都是随机微分方程。这些方程描述了当随机成分参与时系统如何演变。SDEs 在不确定性预测中特别有用，无论是**人口增长**、**疫情传播**还是**股价**。今天，我们将探讨如何使用**Ornstein-Uhlenbeck**过程来模拟温度，这个过程与**热方程**非常相似，热方程是一个描述温度流动的偏微分方程。

![](img/54532d6c0edcc8c6e8d8ddabbd3cdce3.png)

图表由作者绘制

**在我过去的故事中：**

+   [使用 Python 和 NASA/POWER API 收集卫星气候数据](https://towardsdatascience.com/how-to-access-nasas-climate-data-and-how-its-powering-the-fight-against-climate-change-pt-1/) <<< 第一部分

+   [使用物理信息神经网络（PINN）求解热方程。](https://towardsdatascience.com/physics-informed-neural-networks-for-inverse-pde-problems/)

> 气候数据也可在我的 GitHub 上找到，因此我们可以跳过这一步。

**今天我们将：**

+   解释 O-U 过程如何模拟温度时间序列。

+   讨论均值回归过程、随机微分方程以及 O-U 过程与热方程的关系。

+   使用 Python 将 Ornstein-Uhlenbeck（O-U）过程，一个随机微分方程（SDE），拟合到 NASA 气候数据。

![](img/2bfa14cfba8079f339bae384ffb940b8.png)

作者提供的图表

在上图，我们可以看到波动率如何随季节变化。这强调了为什么 O-U 过程非常适合建模这个时间序列的一个重要原因，因为它不假设波动率是恒定的。

### O-U 过程、均值回归和热方程

我们将使用 O-U 过程来模拟在固定点随时间变化的温度。在我们的方程中，T(t) 代表时间（t）的温度函数。正如你直观理解的那样，地球的温度围绕 365 天的均值循环。这意味着季节性变化是通过 s(t) 来建模的。我们可以将这些均值视为巴罗，阿拉斯加和尼亚美，尼日尔温度图中蓝色和橙色的线。

\[dT(t) = ds(t) – \kappa \left( T(t) – s(t) \right) \, dt + \sigma(t) \, dB(t)\]

![](img/54c725e20ab930ed64aad77332cea790.png)

作者提供的图表

![](img/dd8f813921da98ae6483497fe4d7a4c6.png)

作者提供的图表

Kappa (κ) 是均值回归的速度。正如其标题所暗示的那样，这个常数控制了我们的温度回归到季节性均值的速度。在这个上下文中，均值回归意味着如果今天是一个寒冷的日子，比平均温度低，那么我们预计明天的或下周的温度会变暖，因为它围绕那个季节性均值波动。

\[

dT(t) = ds(t) – {\color{red}{\kappa}} \big( T(t) – s(t) \big)\, dt + \sigma(t)\, dB(t)

\]

这个常数在热方程中起着类似的作用，它描述了当一根棒被加热时，其温度增加的速度。在热方程中，κ 是热扩散率，它描述了热量通过材料的流动速度。

\[

\frac{\partial u(\mathbf{x},t)}{\partial t} = {\color{red}{\kappa}} \, \nabla² u(\mathbf{x},t) + q(\mathbf{x},t)

\]

+   **κ = 0:** 无均值回归。过程简化为有漂移的布朗运动。

+   **0 < κ < 1:** 弱均值回归。冲击持续存在，自相关缓慢衰减。

+   **κ > 1:** 强均值回归。偏差被迅速纠正，过程紧密围绕 μ 集中。

### 傅里叶级数

估计均值和波动率

\[s(t) = a + b t + \sum_{k=1}^{K_1} \left( \alpha_k \sin\left( \frac{2\pi k t}{365} \right) + \beta_k \cos\left( \frac{2\pi k t}{365} \right) \right)\]

\[\sigma²(t) = c + \sum_{k=1}^{K_2} \left( \gamma_k \sin\left( \frac{2\pi k t}{365} \right) + \delta_k \cos\left( \frac{2\pi k t}{365} \right) \right)\]

## 将 O-U 过程拟合到孟买数据

我们拟合温度过程的均值

+   使用傅里叶级数拟合均值

+   模型短期依赖性——并使用 AR(1) 过程计算均值回归参数 κ

+   使用傅里叶级数拟合波动率

### 拟合均值

通过将我们的数据拟合到 80% 的数据，我们可以在以后使用我们的随机微分方程（SDE）来预测温度。我们的均值是通过 OLS 回归将 s(t) 拟合到我们的数据中得到的。

![](img/9acf068685c2b3e31f5e4bc1f0dd7998.png)

作者提供的图表

```py
# --------------------
# Config (parameters and paths for analysis)
# --------------------

CITY        = "Mumbai, India"     # Name of the city being analyzed (used in labels/plots)
SEED        = 42                  # Random seed for reproducibility of results
SPLIT_FRAC  = 0.80                # Fraction of data to use for training (rest for testing)
MEAN_HARM   = 2                   # Number of harmonic terms to use for modeling the mean (seasonality)
VOL_HARM    = 3                   # Number of harmonic terms to use for modeling volatility (seasonality)
LJUNG_LAGS  = 10                  # Number of lags for Ljung-Box test (check autocorrelation in residuals)
EPS         = 1e-12               # Small value to avoid division by zero or log(0) issues
MIN_TEST_N  = 8                   # Minimum number of test points required to keep a valid test set

# --------------------
# Paths (where input/output files are stored)
# --------------------

# Base directory in Google Drive where climate data and results are stored
BASE_DIR    = Path("/content/drive/MyDrive/TDS/Climate")

# Subdirectory specifically for outputs of the Mumbai analysis
OUT_BASE    = BASE_DIR / "Benth_Mumbai"

# Subfolder for saving plots generated during analysis
PLOTS_DIR   = OUT_BASE / "plots"

# Ensure the output directories exist (create them if they don’t)
OUT_BASE.mkdir(parents=True, exist_ok=True)
PLOTS_DIR.mkdir(parents=True, exist_ok=True)

# Path to the climate data CSV file (input dataset)
CSV_PATH    = BASE_DIR / "climate_data.csv" 
```

在观察回归的残差时，在噪声中看到信号要困难得多。我们可能会倾向于假设这是白噪声，但我们会犯错误。这些数据中仍然存在依赖性，一种序列依赖性。如果我们希望以后使用傅里叶级数来建模波动性，我们必须考虑这种数据中的依赖性。

![图片](img/d92c8848a11005dfc0e24d580963afdc.png)

作者绘制的图

```py
# =================================================================
# 1) MEAN MODEL: residuals after fitting seasonal mean + diagnostics
# =================================================================

# Residuals after subtracting fitted seasonal mean (before AR step)
mu_train = mean_fit.predict(train)
x_train = train["DAT"] - mu_train

# Save mean model regression summary to text file
save_model_summary_txt(mean_fit, DIAG_DIR / f"{m_slug}_mean_OLS_summary.txt")

# --- Plot: observed DAT vs fitted seasonal mean (train + test) ---
fig = plt.figure(figsize=(12,5))
plt.plot(train["Date"], train["DAT"], lw=1, alpha=0.8, label="DAT (train)")
plt.plot(train["Date"], mu_train, lw=2, label="μ̂(t) (train)")

# Predict and plot fitted mean for test set
mu_test = mean_fit.predict(test[["trend"] + [c for c in train.columns if c.startswith(("cos","sin"))][:2*MEAN_HARM]])
plt.plot(test["Date"], test["DAT"], lw=1, alpha=0.8, label="DAT (test)")
plt.plot(test["Date"], mu_test, lw=2, label="μ̂(t) (test)")

plt.title("Mumbai — DAT and seasonal mean fit")
plt.xlabel("Date"); plt.ylabel("Temperature (DAT)")
plt.legend()
fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_mean_fit_timeseries.png", dpi=160); plt.close(fig)

# --- Plot: mean residuals (time series) ---
fig = plt.figure(figsize=(12,4))
plt.plot(train["Date"], x_train, lw=1)
plt.axhline(0, color="k", lw=1)
plt.title("Mumbai — Residuals after mean fit (x_t = DAT - μ̂)")
plt.xlabel("Date"); plt.ylabel("x_t")
fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_mean_residuals_timeseries.png", dpi=160); plt.close(fig)
```

### κ、ϕ和自回归过程

拟合一个 AR(1)过程。这告诉你均值回归的速度。

一个 AR(1)过程可以捕捉到我们在时间步 t 的残差与 t-1 时刻的值之间的依赖关系。κ的确切值取决于位置。对于孟买，κ = 0.861。这意味着温度很快就会回到平均值。

\[X_t = c + \phi X_{t-1} + \varepsilon_t, \quad \varepsilon_t \sim \mathcal{N}(0,\sigma²)\]

```py
# =======================================================
# 2) AR(1) MODEL: fit and residual diagnostics
# =======================================================

# Extract AR(1) parameter φ and save summary
phi = float(ar_fit.params.iloc[0]) if len(ar_fit.params) else np.nan
save_model_summary_txt(ar_fit, DIAG_DIR / f"{m_slug}_ar1_summary.txt")

# --- Scatterplot: x_t vs x_{t-1} with fitted line φ x_{t-1} ---
x_t = x_train.iloc[1:].values
x_tm1 = x_train.iloc[:-1].values
x_pred = phi * x_tm1
fig = plt.figure(figsize=(5.8,5.2))
plt.scatter(x_tm1, x_t, s=10, alpha=0.6, label="Observed")
xline = np.linspace(np.min(x_tm1), np.max(x_tm1), 2)
plt.plot(xline, phi*xline, lw=2, label=f"Fitted: x_t = {phi:.3f} x_(t-1)")
plt.title("Mumbai — AR(1) scatter: x_t vs x_{t-1}")
plt.xlabel("x_{t-1}"); plt.ylabel("x_t"); plt.legend()
fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_ar1_scatter.png", dpi=160); plt.close(fig)

# --- Time series: actual vs fitted AR(1) values ---
fig = plt.figure(figsize=(12,4))
plt.plot(train["Date"].iloc[1:], x_t, lw=1, label="x_t")
plt.plot(train["Date"].iloc[1:], x_pred, lw=2, label=r"$\hat{x}_t = \phi x_{t-1}$")
plt.axhline(0, color="k", lw=1)
plt.title("Mumbai — AR(1) fitted vs observed deviations")
plt.xlabel("Date"); plt.ylabel("x_t")
plt.legend()
fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_ar1_timeseries_fit.png", dpi=160); plt.close(fig)

# --- Save AR diagnostics (φ + Ljung-Box test p-value) ---
from statsmodels.stats.diagnostic import acorr_ljungbox
try:
    lb = acorr_ljungbox(e_train, lags=[10], return_df=True)
    lb_p = float(lb["lb_pvalue"].iloc[0])   # test for autocorrelation
    lb_stat = float(lb["lb_stat"].iloc[0])
except Exception:
    lb_p = np.nan; lb_stat = np.nan

pd.DataFrame([{"phi": phi, "ljungbox_stat_lag10": lb_stat, "ljungbox_p_lag10": lb_p}])\
    .to_csv(DIAG_DIR / f"{m_slug}_ar1_diagnostics.csv", index=False)
```

![图片](img/7167c5d7f15f3a1f54d675ce062c1dca.png)

作者绘制的图

在上面的图中，我们可以看到我们的 AR(1)过程（橙色）如何从 1981 年到 2016 年估计下一天的温度。我们可以进一步证明 AR(1)过程的使用，并通过绘制 Xₜ和 Xₜ₋₁来获得对其的进一步直觉。在这里，我们可以看到这些值是如何明显相关的。通过这种方式构建，我们也可以看到 AR(1)过程实际上就是线性回归。我们甚至不需要添加漂移/截距项，因为我们已经移除了平均值。因此，我们的截距始终为零。

![图片](img/8e300206eb12076ff11b0270abb644ed.png)

作者绘制的图

现在，观察我们的 AR(1)残差，我们可以开始思考如何建模温度随时间的波动性。从下面的图中，我们可以看到我们的残差似乎以周期性的方式随时间波动。我们可以假设一年中残差较高的点对应于波动性较高的时间段。

```py
# --- Residuals from AR(1) model ---
e_train = ar_fit.resid.astype(float).values
fig = plt.figure(figsize=(12,4))
plt.plot(train["Date"].iloc[1:], e_train, lw=1)
plt.axhline(0, color="k", lw=1)
plt.title("Mumbai — AR(1) residuals ε_t")
plt.xlabel("Date"); plt.ylabel("ε_t")
fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_ar1_residuals_timeseries.png", dpi=160); plt.close(fig) 
```

![图片](img/561d756f1786a51786fd0f4682b8f62a.png)

作者绘制的图

### 波动性傅里叶级数

我们的解决方案是使用傅里叶级数来建模波动性，就像我们为平均值所做的那样。为此，我们必须对我们的残差εₜ进行一些转换。我们考虑εₜ²，因为波动性按定义不能为负。

\[\varepsilon_t = \sigma_t \eta_t \qquad \varepsilon_t^{2} = \sigma_t^{2}\eta_t^{2}\]

我们可以将这些残差视为由标准化冲击ηₜ的部分组成，均值为 0，方差为 1（代表随机性），以及波动性σₜ。我们想要隔离σₜ。通过取对数可以更容易地实现这一点。但更重要的是，这样做使得我们的误差项是可加的。

\[\displaystyle y_t := \log(\varepsilon_t²) = \underbrace{\log\sigma_t²}_{\text{确定性，季节性}} + \underbrace{\log\eta_t²}_{\text{随机噪声}}\]

总结来说，对 log(εₜ²)进行建模有助于：

+   保持σ^t₂ > 0

+   减少异常值对拟合的影响，这意味着它们将对拟合产生较小的影响。

在我们拟合 log(εₜ²)之后，如果我们想恢复εₜ²，我们稍后可以对其进行指数化。

\[\displaystyle \widehat{\sigma}_t^{\,2} \;=\; \exp\!\big(\widehat y_t\big)\]

```py
# =======================================================
# 3) VOLATILITY REGRESSION: fit log(ε_t²) on seasonal harmonics
# =======================================================
if vol_fit is not None:
    # Compute log-squared residuals (proxy for variance)
    log_eps2 = np.log(e_train**2 + 1e-12)

    # Use cosine/sine harmonics as regressors for volatility
    feats = train.iloc[1:][[c for c in train.columns if c.startswith(("cos","sin"))][:2*VOL_HARM]]
    vol_terms = [f"{b}{k}" for k in range(1, VOL_HARM + 1) for b in ("cos","sin")]
    Xbeta = np.asarray(vol_fit.predict(train.iloc[1:][vol_terms]), dtype=float)

    # --- Time plot: observed vs fitted log-variance ---
    fig = plt.figure(figsize=(12,4))
    plt.plot(train["Date"].iloc[1:], log_eps2, lw=1, label="log(ε_t²)")
    plt.plot(train["Date"].iloc[1:], Xbeta, lw=2, label="Fitted log-variance")
    plt.title("Mumbai — Volatility regression (log ε_t²)")
    plt.xlabel("Date"); plt.ylabel("log variance")
    plt.legend()
    fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_volatility_logvar_timeseries.png", dpi=160); plt.close(fig)

    # --- Plot: estimated volatility σ̂(t) over time ---
    fig = plt.figure(figsize=(12,4))
    plt.plot(train["Date"].iloc[1:], sigma_tr, lw=1.5, label="σ̂ (train)")
    if 'sigma_te' in globals():
        plt.plot(test["Date"], sigma_te, lw=1.5, label="σ̂ (test)")
    plt.title("Mumbai — Conditional volatility σ̂(t)")
    plt.xlabel("Date"); plt.ylabel("σ̂")
    plt.legend()
    fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_volatility_sigma_timeseries.png", dpi=160); plt.close(fig)

    # --- Coefficients of volatility regression (with CI) ---
    vol_coef = coef_ci_frame(vol_fit).sort_values("estimate")
    fig = plt.figure(figsize=(8, max(4, 0.4*len(vol_coef))))
    y = np.arange(len(vol_coef))
    plt.errorbar(vol_coef["estimate"], y, xerr=1.96*vol_coef["stderr"], fmt="o", capsize=3)
    plt.yticks(y, vol_coef["term"])
    plt.axvline(0, color="k", lw=1)
    plt.title("Mumbai — Volatility model coefficients (95% CI)")
    plt.xlabel("Estimate")
    fig.tight_layout(); fig.savefig(DIAG_DIR / f"{m_slug}_volatility_coefficients.png", dpi=160); plt.close(fig)

    # Save volatility regression summary
    save_model_summary_txt(vol_fit, DIAG_DIR / f"{m_slug}_volatility_summary.txt")
else:
    print("Volatility model not available (too few points or regression failed). Skipping vol plots.")

print("Diagnostics saved to:", DIAG_DIR)
```

![图片](img/dd61061a92cffc15fbae062eda4d0f57.png)

由作者绘制的图表

\[dT(t) = ds(t) – \kappa (T(t) – s(t)),dt + \exp\left(\frac{1}{2}(\alpha + \beta^T z(t))\right), dB(t)\]

## 对数稳定化的演示

下面的图表说明了对数如何帮助拟合波动率。残差εₜ²包含显著的异常值，部分原因是平方运算确保在稳定尺度上的正值。没有对数稳定，大的冲击主导了拟合，最终结果是偏颇的。

![图片](img/27ba7528a34290861567bf6deb1b89a9.png)

由作者绘制的图表

![图片](img/6bab10906ee6eb43723386c06da1df67.png)

由作者绘制的图表

![图片](img/a0d7cea01a4ef580473e95c5869bd2f8.png)

由作者绘制的图表

![图片](img/332537ec4d1c3c2f1df9bf00fd276861.png)

由作者绘制的图表

```py
# Fix the seasonal plot from the previous cell (indexing bug) and
# add a *second scenario* with rare large outliers to illustrate instability
# when regressing on skewed eps² directly.

# ------------------------------
# 1) Seasonal view (fixed indexing)
# ------------------------------
# Recreate the arrays from the prior simulation cell by re-executing the same seed & setup
np.random.seed(7)                 # deterministic reproducibility
n_days = 730                      # two synthetic years (365 * 2)
t = np.arange(n_days)             # day index 0..729
omega = 2 * np.pi / 365.0         # seasonal frequency (one-year period)

# True (data-generating) log-variance is seasonal via sin/cos
a_true, b_true, c_true = 0.2, 0.9, -0.35
log_var_true = a_true + b_true * np.sin(omega * t) + c_true * np.cos(omega * t)

# Convert log-variance to standard deviation: sigma = exp(0.5 * log var)
sigma_true = np.exp(0.5 * log_var_true)

# White noise innovations; variance scaled by sigma_true
z = np.random.normal(size=n_days)
eps = sigma_true * z              # heteroskedastic residuals
eps2 = eps**2                     # "variance-like" target if regressing raw eps²

# Design matrix with intercept + annual sin/cos harmonics
X = np.column_stack([np.ones(n_days), np.sin(omega * t), np.cos(omega * t)])

# --- Fit A: OLS on raw eps² (sensitive to skew/outliers) ---
beta_A, *_ = np.linalg.lstsq(X, eps2, rcond=None)
var_hat_A = X @ beta_A            # fitted variance (can be negative from OLS)
var_hat_A = np.clip(var_hat_A, 1e-8, None)  # clip to avoid negatives
sigma_hat_A = np.sqrt(var_hat_A)  # convert to sigma

# --- Fit B: OLS on log(eps²) (stabilizes scale & reduces skew) ---
eps_safe = 1e-12                  # small epsilon to avoid log(0)
y_log = np.log(eps2 + eps_safe)   # stabilized target
beta_B, *_ = np.linalg.lstsq(X, y_log, rcond=None)
log_var_hat_B = X @ beta_B
sigma_hat_B = np.exp(0.5 * log_var_hat_B)

# Day-of-year index for seasonal averaging across years
doy = t % 365

# Correct ordering for the first year's DOY values
order365 = np.argsort(doy[:365])

def seasonal_mean(x):
    """
    Average the two years day-by-day to get a single seasonal curve.
    Assumes x has length 730 (two years); returns length-365 array.
    """
    return 0.5 * (x[:365] + x[365:730])

# Plot one "synthetic year" view of the seasonal sigma pattern
plt.figure(figsize=(12, 5))
plt.plot(np.sort(doy[:365]), seasonal_mean(sigma_true)[order365], label="True sigma seasonality")
plt.plot(np.sort(doy[:365]), seasonal_mean(sigma_hat_A)[order365], label="Fitted from eps² regression", alpha=0.9)
plt.plot(np.sort(doy[:365]), seasonal_mean(sigma_hat_B)[order365], label="Fitted from log(eps²) regression", alpha=0.9)
plt.title("Seasonal Volatility Pattern: True vs Fitted (one-year view) – Fixed")
plt.xlabel("Day of year")
plt.ylabel("Sigma")
plt.legend()
plt.tight_layout()
plt.show()

# ------------------------------
# 2) OUTLIER SCENARIO to illustrate instability
# ------------------------------
np.random.seed(21)                # separate seed for the outlier experiment

def run_scenario(n_days=730, outlier_rate=0.05, outlier_scale=8.0):
    """
    Generate two-year heteroskedastic residuals with occasional huge shocks
    to mimic heavy tails. Compare:
      - Fit A: regress raw eps² on sin/cos (can be unstable, negative fits)
      - Fit B: regress log(eps²) on sin/cos (more stable under heavy tails)
    Return fitted sigmas and error metrics (MAE/MAPE), plus diagnostics.
    """
    t = np.arange(n_days)
    omega = 2 * np.pi / 365.0

    # Same true seasonal log-variance as above
    a_true, b_true, c_true = 0.2, 0.9, -0.35
    log_var_true = a_true + b_true * np.sin(omega * t) + c_true * np.cos(omega * t)
    sigma_true = np.exp(0.5 * log_var_true)

    # Base normal innovations
    z = np.random.normal(size=n_days)

    # Inject rare, huge shocks to create heavy tails in eps²
    mask = np.random.rand(n_days) < outlier_rate
    z[mask] *= outlier_scale

    # Heteroskedastic residuals and their squares
    eps = sigma_true * z
    eps2 = eps**2

    # Same sin/cos design
    X = np.column_stack([np.ones(n_days), np.sin(omega * t), np.cos(omega * t)])

    # --- Fit A: raw eps² on X (OLS) ---
    beta_A, *_ = np.linalg.lstsq(X, eps2, rcond=None)
    var_hat_A_raw = X @ beta_A
    neg_frac = np.mean(var_hat_A_raw < 0.0)        # fraction of negative variance predictions
    var_hat_A = np.clip(var_hat_A_raw, 1e-8, None) # clip to ensure non-negative variance
    sigma_hat_A = np.sqrt(var_hat_A)

    # --- Fit B: log(eps²) on X (OLS on log-scale) ---
    y_log = np.log(eps2 + 1e-12)
    beta_B, *_ = np.linalg.lstsq(X, y_log, rcond=None)
    log_var_hat_B = X @ beta_B
    sigma_hat_B = np.exp(0.5 * log_var_hat_B)

    # Error metrics comparing fitted sigmas to the true sigma path
    mae = lambda a, b: np.mean(np.abs(a - b))
    mape = lambda a, b: np.mean(np.abs((a - b) / (a + 1e-12))) * 100

    mae_A = mae(sigma_true, sigma_hat_A)
    mae_B = mae(sigma_true, sigma_hat_B)
    mape_A = mape(sigma_true, sigma_hat_A)
    mape_B = mape(sigma_true, sigma_hat_B)

    return {
        "t": t,
        "sigma_true": sigma_true,
        "sigma_hat_A": sigma_hat_A,
        "sigma_hat_B": sigma_hat_B,
        "eps2": eps2,
        "y_log": y_log,
        "neg_frac": neg_frac,
        "mae_A": mae_A, "mae_B": mae_B,
        "mape_A": mape_A, "mape_B": mape_B
    }

# Run with 5% outliers scaled 10x to make the point obvious
res = run_scenario(outlier_rate=0.05, outlier_scale=10.0)

print("\nOUTLIER SCENARIO (5% of days have 10x shocks) — illustrating instability when using eps² directly")
print(f"  MAE  (sigma):  raw eps² regression = {res['mae_A']:.4f}   |   log(eps²) regression = {res['mae_B']:.4f}")
print(f"  MAPE (sigma):  raw eps² regression = {res['mape_A']:.2f}% |   log(eps²) regression = {res['mape_B']:.2f}%")
print(f"  Negative variance predictions before clipping (raw fit): {res['neg_frac']:.2%}")

# Visual comparison: true sigma vs two fitted approaches under outliers
plt.figure(figsize=(12, 5))
plt.plot(res["t"], res["sigma_true"], label="True sigma(t)")
plt.plot(res["t"], res["sigma_hat_A"], label="Fitted sigma from eps² regression", alpha=0.9)
plt.plot(res["t"], res["sigma_hat_B"], label="Fitted sigma from log(eps²) regression", alpha=0.9)
plt.title("True vs Fitted Volatility with Rare Large Shocks")
plt.xlabel("Day")
plt.ylabel("Sigma")
plt.legend()
plt.tight_layout()
plt.show()

# Show how the targets behave when outliers are present
plt.figure(figsize=(12, 5))
plt.plot(res["t"], res["eps2"], label="eps² (now extremely heavy-tailed due to outliers)")
plt.title("eps² under outliers: unstable target for regression")
plt.xlabel("Day")
plt.ylabel("eps²")
plt.legend()
plt.tight_layout()
plt.show()

plt.figure(figsize=(12, 5))
plt.plot(res["t"], res["y_log"], label="log(eps²): compressed & stabilized")
plt.title("log(eps²) under outliers: stabilized scale")
plt.xlabel("Day")
plt.ylabel("log(eps²)")
plt.legend()
plt.tight_layout()
plt.show() 
```

## 结论

现在我们已经将这些参数拟合到这个 SDE 中，我们可以考虑使用它来进行预测。但到达这里并不容易。我们的解决方案严重依赖于两个关键概念。

+   函数可以通过傅里叶级数进行近似。

+   均值回归参数κ相当于我们 AR(1)过程中的ϕ。

每次拟合随机微分方程，每次进行任何*随机*操作时，我都会觉得很有趣，想象一下这个词是从*stokhos*（意为瞄准）演变而来的。更有趣的是，这个词变成了*stokhazesthai*（意为瞄准/猜测）。这个过程肯定涉及了一些错误和糟糕的瞄准。

第一部分：

Tallarico, M. H. (2025 年 7 月 1 日). 如何访问 NASA 的气候数据——以及它是如何推动对抗气候变化的：第一部分：从建筑设计到食品安全。走向数据科学。[链接](https://towardsdatascience.com/how-to-access-nasas-climate-data-and-how-its-powering-the-fight-against-climate-change-pt-1/。[谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:Tyk-4Ss8FVUC)

我的下一篇文章：

Tallarico, M. H. (2025 年 12 月 24 日). 奥卡姆剃刀与贝叶斯-霍奇伯格：选择你的 P 值校正：时间α = 10⁻⁹⁹太大：超重元素和欺骗。走向数据科学。[链接](https://towardsdatascience.com/the-time-10-99-was-too-big-superheavy-elements-and-deceit/)。[谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:W7OEmFMy1HYC).

我的前一篇文章：

Tallarico, M. H. (2025 年 7 月 29 日). 物理信息神经网络用于逆偏微分方程问题：使用 DeepXDE 求解热方程。走向数据科学。[链接](https://towardsdatascience.com/physics-informed-neural-networks-for-inverse-pde-problems/)。[谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:d1gkVwhDpl0C).

## 参考文献

Alexandridis, A. K., & Zapranis, A. D. (2013). *天气衍生品：天气相关风险的建模和定价*. Springer. https://doi.org/10.1007/978-1-4614-6071-8

Benth, F. E., & Šaltytė Benth, J. (2007). 温度波动及其天气衍生品定价. 定量金融，7(5)，553–561\. https://doi.org/10.1080/14697680601155334

Tallarico, M. H., & Olivares, P. (2024). 使用卫星数据进行天气衍生品定价的神经和时间序列方法：性能和制度适应性. arXiv. [链接](https://arxiv.org/abs/2411.12013). [谷歌学术](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:u5HHmVD_uO8C).

Tallarico, M. H. (2023). 使用时间序列和机器学习方法对多伦多温度进行建模和预测. Academia.edu. [链接](https://www.academia.edu/161958961/Modeling_and_Forecasting_Temperature_in_Toronto_Using_Times_Series_and_Machine_Learning_Methods). [Academia](https://www.academia.edu/161958961/Modeling_and_Forecasting_Temperature_in_Toronto_Using_Times_Series_and_Machine_Learning_Methods).

Tallarico, M. H. (2025, July 1). 如何访问 NASA 的气候数据——以及它是如何助力对抗气候变化的：第一部分：从建筑设计到粮食安全. Towards Data Science. [链接](https://towardsdatascience.com/how-to-access-nasas-climate-data-and-how-its-powering-the-fight-against-climate-change-pt-1/). [谷歌学术.](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uCZbo_kAAAAJ&citation_for_view=uCZbo_kAAAAJ:Tyk-4Ss8FVUC)

* * *

[网站](https://marcoheningtallarico.com/) | [领英](https://www.linkedin.com/in/marco-hening-tallarico/) | [GitHub](https://github.com/marco-hening-tallarico?tab=repositories)

![Marco Hening Tallarico](img/5ed32597a2d8c82e935751f25ad6cc90.png)

作者

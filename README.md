# Quantitative Market Risk Modeling: Value at Risk (VaR), Expected Shortfall (ES), and GARCH Dynamics for Vietnam Banking Equities (2020–2026)

[![Financial Risk Management](https://img.shields.io/badge/Domain-Quantitative_Risk_Management-0A2540?style=for-the-badge&logo=cashapp)](https://github.com/)
[![Methods](https://img.shields.io/badge/Methodology-VaR_%7C_ES_%7C_GARCH(1%2C1)_%7C_EWMA-0052CC?style=for-the-badge)](https://github.com/)
[![Backtesting](https://img.shields.io/badge/Validation-Kupiec_POF_Test-4C8C2B?style=for-the-badge)](https://github.com/)
[![Excel Modeling](https://img.shields.io/badge/Implementation-Microsoft_Excel_Solver_%26_Monte_Carlo-107C41?style=for-the-badge&logo=microsoftexcel)](https://github.com/)
[![Regulatory Framework](https://img.shields.io/badge/Regulatory-Basel_II_%2F_Basel_III_IMA-D9381E?style=for-the-badge)](https://www.bis.org/bcbs/)
[![Academic Research](https://img.shields.io/badge/Research-UFM_Quantitative_Finance-5C2D91?style=for-the-badge)](https://ufm.edu.vn/)

---

## Executive Summary

In emerging financial markets, the banking sector constitutes the systemic backbone of credit allocation and capital liquidity. However, banking equities exhibit pronounced non-linear vulnerability to macroeconomic turbulence, monetary tightening cycles, and global geopolitical shocks. Standard risk management architectures frequently rely on normal distribution assumptions, leading to severe underestimations of tail risk during extreme drawdown events.

This quantitative research project provides an end-to-end empirical market risk evaluation of premier Vietnamese banking equities (**VCB**, **BID**, **CTG**, and **MBB**) on the Ho Chi Minh City Stock Exchange (HOSE) over the high-volatility period from **January 02, 2020 to February 27, 2026** ($T = 1,534$ trading days / $1,533$ continuous log-returns).

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                  QUANTITATIVE PIPELINE OVERVIEW                                  │
├───────────────────┬──────────────────────────┬─────────────────────────┬─────────────────────────┤
│ Data Ingestion    │ Volatility Dynamics      │ Tail Risk Estimation    │ Statistical Validation  │
│ 1,534 Daily Closes│ • EWMA (λ = 0.94)        │ • Historical Simulation │ • Kupiec POF LR Test    │
│ VCB, BID, CTG, MBB│ • ARCH(1) MLE            │ • Parametric (VCV)      │ • Failure Rate Analysis │
│ HOSE (2020–2026)  │ • GARCH(1,1) MLE Solver  │ • Monte Carlo (10k Sim) │ • Basel Traffic Light   │
└───────────────────┴──────────────────────────┴─────────────────────────┴─────────────────────────┘
```

### Core Research Questions & Key Takeaways
1. **The "Normality Trap" in Tail Risk**: While Parametric and Monte Carlo models adequately capture moderate losses at the $95\%$ confidence level, they **fail catastrophically at the $99\%$ confidence level** (Kupiec p-values $< 10^{-5}$). In contrast, non-parametric **Historical Simulation** demonstrates rigorous statistical robustness across all significance levels ($p_{95\%} = 0.9673$, $p_{99\%} = 0.8644$).
2. **Coherent Tail Risk via Expected Shortfall (ES)**: At $\alpha = 1\%$, the Expected Shortfall ($5.814\text{M VND}$ on a $100\text{M VND}$ portfolio) exceeds VaR ($4.484\text{M VND}$) by $+29.6\%$, exposing deep tail losses that single-quantile VaR fails to capture.
3. **Volatility Clustering & Persistence**: State-owned commercial banks (**CTG** and **BID**) exhibit extreme volatility memory ($\alpha + \beta = 0.8946$ and $0.8514$, respectively), confirming that post-shock volatility remains elevated over long horizons. GARCH(1,1) optimized via Maximum Likelihood Estimation (MLE) substantially outperforms ARCH(1) and EWMA in log-likelihood maximization ($\text{LLF} \approx 3,817 - 3,925$ vs. $\text{LLF} \approx 1,248 - 1,403$).

---

## Quantitative Methodology & Mathematical Formulations

```
                              ┌───────────────────────────────┐
                              │  Asset Return Process r_t     │
                              └──────────────┬────────────────┘
                                             │
                      ┌──────────────────────┴──────────────────────┐
                      ▼                                             ▼
       ┌─────────────────────────────┐               ┌─────────────────────────────┐
       │ Static / Distributional VaR │               │ Dynamic Volatility σ_t      │
       ├─────────────────────────────┤               ├─────────────────────────────┤
       │ • Historical Simulation     │               │ • EWMA (λ = 0.94)           │
       │ • Parametric Variance-Cov   │               │ • ARCH(1) Solver MLE        │
       │ • Monte Carlo Simulation    │               │ • GARCH(1,1) Solver MLE     │
       └──────────────┬──────────────┘               └──────────────┬──────────────┘
                      │                                             │
                      └──────────────────────┬──────────────────────┘
                                             ▼
                              ┌───────────────────────────────┐
                              │     Backtesting Framework     │
                              │ • Kupiec POF Likelihood Ratio │
                              │ • Christoffersen Independence │
                              └───────────────────────────────┘
```

### 1. Asset Return Transformation
Continuous daily log-returns $r_t$ are generated from adjusted closing prices $P_t$:
$$r_t = \ln\left(\frac{P_t}{P_{t-1}}\right) = \ln(P_t) - \ln(P_{t-1})$$

For an $N$-asset portfolio with allocation weights $\mathbf{w} = [w_1, w_2, \dots, w_N]^T$ ($\sum w_i = 1$), portfolio return $r_{p,t}$ and expected portfolio variance $\sigma_p^2$ are:
$$r_{p,t} = \sum_{i=1}^N w_i r_{i,t}, \quad \sigma_p^2 = \mathbf{w}^T \mathbf{\Sigma} \mathbf{w} = \sum_{i=1}^N \sum_{j=1}^N w_i w_j \sigma_{ij}$$
where $\mathbf{\Sigma}$ denotes the sample covariance matrix.

---

### 2. Value at Risk (VaR) Formulations

Given initial portfolio equity $V_0$, confidence level $(1 - \alpha)$, and holding period $T$:

#### A. Parametric Variance-Covariance Method
Assuming returns follow $r_p \sim \mathcal{N}(\mu_p, \sigma_p^2)$:
$$\text{VaR}_{(1-\alpha)} = V_0 \cdot \left( z_{\alpha} \sigma_p - \mu_p \right)$$
where $z_{\alpha} = \Phi^{-1}(1 - \alpha)$ is the standard normal critical value ($z_{0.05} = 1.64485$, $z_{0.01} = 2.32635$).

#### B. Historical Simulation (Non-Parametric)
Constructs the empirical cumulative distribution function (ECDF) of historical daily portfolio profit/loss $\Delta V_t = V_0 \cdot r_{p,t}$:
$$\text{VaR}_{(1-\alpha)} = - \text{Quantile}_{\alpha}\left(\{ \Delta V_t \}_{t=1}^T\right) = - \inf\left\{ \ell \in \mathbb{R} : F_{\Delta V}(\ell) \ge \alpha \right\}$$

#### C. Monte Carlo Simulation
Under a standard Geometric Brownian Motion (GBM) discrete path model over $M = 10,000$ simulation trials:
$$S_{T}^{(k)} = S_0 \exp\left( \left(\mu - \frac{1}{2}\sigma^2\right)\Delta t + \sigma \sqrt{\Delta t} \cdot Z^{(k)} \right), \quad Z^{(k)} \sim \mathcal{N}(0,1)$$
$$\text{VaR}_{(1-\alpha)}^{\text{MC}} = - \text{Percentile}_{\alpha}\left( \Delta V_{\text{sim}}^{(1)}, \dots, \Delta V_{\text{sim}}^{(M)} \right)$$

---

### 3. Expected Shortfall (ES / Conditional VaR)

Expected Shortfall measures the conditional expectation of loss exceeding the VaR threshold:
$$\text{ES}_{(1-\alpha)} = \mathbb{E}\left[ -\Delta V \;\middle|\; -\Delta V \ge \text{VaR}_{(1-\alpha)} \right] = \frac{1}{\alpha} \int_0^{\alpha} \text{VaR}_{(1-u)} \, du$$

* **Historical ES**:
  $$\text{ES}_{(1-\alpha)}^{\text{His}} = -\frac{1}{K} \sum_{t=1}^T \Delta V_t \cdot \mathbb{I}_{\{\Delta V_t \le -\text{VaR}_{(1-\alpha)}\}}, \quad K = \sum_{t=1}^T \mathbb{I}_{\{\Delta V_t \le -\text{VaR}_{(1-\alpha)}\}}$$
* **Parametric ES**:
  $$\text{ES}_{(1-\alpha)}^{\text{Para}} = V_0 \left[ -\mu_p + \sigma_p \cdot \frac{\phi(z_{\alpha})}{\alpha} \right]$$
  where $\phi(\cdot)$ is the Standard Normal Probability Density Function.

---

### 4. Dynamic Conditional Heteroskedasticity Models

To capture volatility clustering ($\sigma_t^2 \neq \text{const}$), three time-varying conditional variance models were calibrated:

#### A. Exponentially Weighted Moving Average (EWMA / RiskMetrics)
$$\sigma_t^2 = \lambda \sigma_{t-1}^2 + (1 - \lambda) \epsilon_{t-1}^2, \quad \lambda = 0.94$$

#### B. Autoregressive Conditional Heteroskedasticity — ARCH(1)
$$\epsilon_t = \sigma_t z_t, \quad z_t \overset{i.i.d.}{\sim} \mathcal{N}(0,1)$$
$$\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2 \quad (\omega > 0, \; 0 \le \alpha < 1)$$

#### C. Generalized ARCH — GARCH(1,1)
$$\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2 + \beta \sigma_{t-1}^2 \quad (\omega > 0, \; \alpha \ge 0, \; \beta \ge 0, \; \alpha + \beta < 1)$$
* **Long-Term Unconditional Volatility**:
  $$V_L = \sqrt{\sigma_{\text{uncond}}^2} = \sqrt{\frac{\omega}{1 - \alpha - \beta}}$$
* **Maximum Likelihood Estimation (MLE)** via Excel Solver:
  $$\ln L(\theta) = -\frac{T}{2}\ln(2\pi) - \frac{1}{2}\sum_{t=1}^T \ln(\sigma_t^2) - \frac{1}{2}\sum_{t=1}^T \frac{\epsilon_t^2}{\sigma_t^2} \longrightarrow \max_{\{\omega, \alpha, \beta\}}$$

---

### 5. Statistical Backtesting Framework

Model adequacy is formally evaluated using the **Kupiec Proportion of Failures (POF) Likelihood Ratio Test**:

Define the violation sequence $I_t = \mathbb{I}_{\{r_{p,t} < -\text{VaR}_{\alpha,t}\}}$ with sample size $T$ and total exception count $N = \sum_{t=1}^T I_t$, giving an empirical failure rate $\hat{p} = N / T$. Under the null hypothesis $H_0: p = p_0 = \alpha$:

$$LR_{\text{POF}} = -2 \ln \left[ \frac{(1 - p_0)^{T-N} p_0^N}{(1 - \hat{p})^{T-N} \hat{p}^N} \right] = 2 \left[ (T-N)\ln(1-\hat{p}) + N\ln(\hat{p}) - (T-N)\ln(1-p_0) - N\ln(p_0) \right]$$

Under $H_0$, $LR_{\text{POF}} \overset{d}{\longrightarrow} \chi^2(1)$ (Critical value at $5\%$ significance is $\chi_{0.05}^2(1) = 3.841$).
The decision rule rejects model accuracy if $p\text{-value} = \Pr\left(\chi^2(1) \ge LR_{\text{POF}}\right) < 0.05$.

---

## Empirical Findings & Model Benchmark Tables

### 1. Descriptive Statistics of Banking Equities (2020–2026)
*Sample size: $T = 1,533$ continuous trading returns.*

| Asset Ticker | Mean Return ($\mu$) | Daily Volatility ($\sigma$) | Annualized Volatility ($\sigma \sqrt{252}$) | Minimum Loss | Sample Skewness | Sample Kurtosis | Distributional Trait |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **VCB** (Vietcombank) | $-0.0047\%$ | $1.988\%$ | $31.56\%$ | $-40.21\%$ | **$-5.758$** | **$112.610$** | Extreme Leptokurtic / Heavy Crash Tail |
| **BID** (BIDV) | $+0.0177\%$ | $2.160\%$ | $34.29\%$ | $-6.98\%$ | $-0.512$ | $3.511$ | Moderate Fat-Tailed ($\text{Kurt} > 3$) |
| **CTG** (VietinBank) | $+0.0886\%$ | $2.146\%$ | $34.07\%$ | $-7.00\%$ | $-0.274$ | $2.249$ | Moderate Volatility Clustering |
| **MBB** (MB Bank) | $+0.0977\%$ | $2.007\%$ | $31.86\%$ | $-6.99\%$ | $-0.424$ | $2.834$ | Consistent Growth with Regime Shifts |

> [!NOTE]
> **Key Insight**: VCB displays a colossal kurtosis of **112.61** and skewness of **-5.76**, driven by discrete price-shock discontinuities. This quantitatively refutes the validity of assuming a Gaussian distribution for Vietnamese financial assets.

---

### 2. Multi-Asset Covariance Structure & Portfolio Calibration
*Equal-weighted allocation ($w_i = 25\%$), initial portfolio capital $V_0 = 100,000,000\text{ VND}$.*

```
                           COVARIANCE MATRIX (Σ × 10^-4)
                    VCB          BID          CTG          MBB
          VCB ┌   3.9540       1.9660       1.9250      -0.0310   ┐
          BID │   1.9660       4.6640       3.1790      -0.0010   │
          CTG │   1.9250       3.1790       4.6040       0.1530   │
          MBB └  -0.0310      -0.0010       0.1530       4.0270   ┘
```

* **Portfolio Expected Return ($\mu_p$)**: $0.0498\%/\text{day}$ ($+12.55\%/\text{year}$)
* **Portfolio Variance ($\sigma_p^2$)**: $0.0001977$
* **Portfolio Daily Volatility ($\sigma_p$)**: $1.4060\%/\text{day}$ ($22.32\%/\text{year}$)
* **Diversification Benefit**: Equal-weighted portfolio standard deviation ($1.406\%$) is significantly below the average component asset volatility ($\approx 2.075\%$), achieving a **$\approx 32.2\%$ volatility reduction** via asset diversification.

---

### 3. Risk Benchmark: VaR & Expected Shortfall Comparison

| Estimation Methodology | Significance ($\alpha$) | VaR (% of Portfolio) | VaR Amount (Million VND) | ES Amount (Million VND) | ES / VaR Ratio |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Historical Simulation** | $5\%$ ($95\%$ Conf.) | $2.170\%$ | **$2.1699$** | **$3.6131$** | $1.665\times$ |
| | $1\%$ ($99\%$ Conf.) | $4.484\%$ | **$4.4845$** | **$5.8137$** | **$1.296\times$** |
| **Parametric (VCV)** | $5\%$ ($95\%$ Conf.) | $2.263\%$ | **$2.2629$** | **$2.8504$** | $1.260\times$ |
| | $1\%$ ($99\%$ Conf.) | $3.221\%$ | **$3.2211$** | **$3.6976$** | $1.148\times$ |
| **Monte Carlo (GBM 10k)** | $5\%$ ($95\%$ Conf.) | $2.309\%$ | **$2.3088$** | **$2.9398$** | $1.273\times$ |
| | $1\%$ ($99\%$ Conf.) | $3.315\%$ | **$3.3147$** | **$3.7547$** | $1.133\times$ |

---

### 4. Backtesting Verification: Kupiec POF Likelihood Ratio Test
*Sample size $T = 1,533$ observations. Expected exceptions: $N_{95\%} = 76.65$, $N_{99\%} = 15.33$.*

| Risk Model | Target $\alpha$ | Actual Exceptions ($N$) | Failure Rate ($\hat{p}$) | Kupiec $LR_{\text{POF}}$ | $p\text{-value}$ | Statistical Decision ($\alpha_{\text{test}} = 0.05$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Historical Simulation** | **$5\%$** | **$77$** | **$5.023\%$** | **$0.0016$** | **$0.9673$** | ✅ **Passed / Highly Reliable** |
| **Historical Simulation** | **$1\%$** | **$16$** | **$1.044\%$** | **$0.0290$** | **$0.8644$** | ✅ **Passed / Robust Tail Coverage** |
| **Parametric (VCV)** | $5\%$ | $70$ | $4.566\%$ | $0.6247$ | $0.4293$ | ✅ Passed |
| **Parametric (VCV)** | $1\%$ | **$39$** | **$2.544\%$** | **$22.146$** | **$2.49 \times 10^{-6}$** | ❌ **REJECTED (Severe Underestimation)** |
| **Monte Carlo (GBM)** | $5\%$ | $68$ | $4.436\%$ | $1.0664$ | $0.3018$ | ✅ Passed |
| **Monte Carlo (GBM)** | $1\%$ | **$36$** | **$2.348\%$** | **$18.681$** | **$1.53 \times 10^{-5}$** | ❌ **REJECTED (Model Inadequacy)** |

```
                       KUPIEC POF TEST P-VALUES (LOG SCALE)
                       1.0 ┼────────────────────────────────────
                           │   Historical 95% (0.967)
                       0.8 ┼─  Historical 99% (0.864)
                           │
                       0.4 ┼─  Parametric 95% (0.429)
                       0.3 ┼─  Monte Carlo 95% (0.302)
                           │
                     0.05 ─┼════════════════════════════════════ Critical Rejection Threshold
                           │
                     10^-5 ┼─  Monte Carlo 99% (1.5e-5) [FAILED]
                     10^-6 ┼─  Parametric 99% (2.5e-6)  [FAILED]
```

---

### 5. Econometric Volatility Parameter Estimates (Solver MLE Calibration)

#### A. GARCH(1,1) Dynamics: $\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2 + \beta \sigma_{t-1}^2$

| Bank Asset | Constant ($\mu$) | $\omega$ ($\times 10^{-4}$) | ARCH Coeff ($\alpha$) | GARCH Coeff ($\beta$) | Persistence ($\alpha + \beta$) | Long-Term Vol ($V_L$) | Max Log-Likelihood ($\text{LLF}$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **VCB** | $-0.0047\%$ | $1.8016$ | $0.0937$ | $0.4685$ | $0.5621$ | $2.028\%$ | **$3,849.71$** |
| **BID** | $+0.0670\%$ | $0.7910$ | $0.2005$ | $0.6510$ | **$0.8514$** | $2.307\%$ | **$3,817.01$** |
| **CTG** | $+0.0144\%$ | $0.4947$ | $0.1546$ | $0.7400$ | **$0.8946$** | $2.166\%$ | **$3,826.62$** |
| **MBB** | $+0.0977\%$ | $1.0000$ | $0.2138$ | $0.5502$ | **$0.7640$** | $2.059\%$ | **$3,925.06$** |

#### B. ARCH(1) Dynamics: $\sigma_t^2 = \omega + \alpha \epsilon_{t-1}^2$

| Bank Asset | $\omega$ ($\times 10^{-4}$) | ARCH Coeff ($\alpha$) | Long-Term Vol ($V_L$) | Max Log-Likelihood ($\text{LLF}$) | Likelihood Ratio ($\Delta \text{LLF}$) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **VCB** | $2.4786$ | $0.3426$ | $1.942\%$ | $1,403.73$ | $+2,445.98$ for GARCH |
| **BID** | $4.2735$ | $0.2054$ | $2.319\%$ | $1,281.60$ | $+2,535.41$ for GARCH |
| **CTG** | $5.1042$ | $0.1402$ | $2.436\%$ | $1,248.27$ | $+2,578.35$ for GARCH |
| **MBB** | $3.8535$ | $0.2041$ | $2.200\%$ | $1,310.34$ | $+2,614.72$ for GARCH |

> [!IMPORTANT]
> **Econometric Synthesis**:
> * **Superiority of GARCH over ARCH**: The Likelihood Ratio test statistic $LR = 2(\text{LLF}_{\text{GARCH}} - \text{LLF}_{\text{ARCH}}) > 4,800 \gg \chi^2_{0.001}(1) = 10.83$ demonstrates that lagged conditional variance ($\beta \sigma_{t-1}^2$) is indispensable for modeling financial equities.
> * **Memory Effects**: CTG exhibits the highest persistence ($\alpha + \beta = 0.8946$), indicating that market shocks subside slowly and require extended capital buffers.

---

## Visual Highlights & Empirical Plots

All analytical figures generated during the research are indexed below and available in the [`figures/`](file:///c:/Users/Asus/Downloads/Github2/figures) directory:

| Visual Representation | Figure Path | Econometric Phenomenon Captured |
| :--- | :---: | :--- |
| **Historical Price Series** | `figures/price_trends.png` | Co-movement across banking cycle (COVID-19 dip, 2021 bull run, 2022-2024 monetary tightening). |
| **Daily Log-Return Distributions** | `figures/log_returns.png` | Asymmetric leptokurtosis, negative skewness, and extreme downside outliers. |
| **EWMA VaR Dynamics (95% & 99%)** | `figures/ewma_var.png` | Fast exponential updates ($\lambda = 0.94$) during structural drawdown intervals. |
| **ARCH(1) VaR Spikes** | `figures/arch_var.png` | High sensitivity to immediate previous shocks with noisy, spiky tail boundaries. |
| **GARCH(1,1) Smooth VaR Dynamics** | `figures/garch_var.png` | Optimal balance of shock reaction and memory smoothing over time. |

---

## Repository Structure

```
├── TLMHĐLRRTC_Hồ Quỳnh My.xlsx     # Primary quantitative modeling workbook (15 sheets + Solver MLE)
├── DL_TLMHĐLRRTC_Hồ Quỳnh My.xlsx  # Processed raw market data & historical price feeds
├── TLMHĐLRRTC_Hồ Quỳnh My_2321000338.pdf # Comprehensive research defense paper (Academic Thesis)
├── TLMHĐLRRTC_Hồ Quỳnh My_2321000338.docx# Original editable thesis documentation
├── figures/                               # Directory containing all visual charts & empirical plots
│   └── image1.png                         # Extracted asset trajectory chart
├── .gitignore                             # Standard Microsoft Office, Windows & OS ignore rules
└── README.md                              # Quantitative Portfolio & Technical Documentation
```

---

## Excel Architecture & Reproduction Workflow

The accompanying Excel workbook [`TLMHĐLRRTC_Hồ Quỳnh My.xlsx`](file:///c:/Users/Asus/Downloads/Github2/TLMH%C4%90LRRTC_H%E1%BB%93%20Qu%E1%BB%B3nh%20My.xlsx) is built with 15 dedicated sheets:

```
┌─────────────────┬────────────────────────────────────────────────────────────────────────┐
│ Sheet Name      │ Quantitative Functionality & Core Excel Formulas                       │
├─────────────────┼────────────────────────────────────────────────────────────────────────┤
│ Last            │ Continuous log-return generation: =LN(P_t / P_{t-1})                  │
│ His             │ Non-parametric historical P&L and quantile search: =PERCENTILE.INC()  │
│ Para            │ Parametric Variance-Covariance Matrix (=MMULT, =NORM.S.INV)            │
│ Monte           │ 10,000-iteration Monte Carlo engine via Inverse Transform Sampling    │
│ Backtesting     │ Kupiec POF Likelihood Ratio test & p-values (=CHISQ.DIST.RT)           │
│ Garch [1 to 4]  │ VCB, BID, CTG, MBB GARCH(1,1) MLE Solver optimization                  │
│ Arch [1 to 4]   │ VCB, BID, CTG, MBB ARCH(1) MLE Solver calibration                      │
│ EWMA            │ RiskMetrics recursive volatility calculation: =0.94*σ_{t-1}^2 + 0.06*r^2│
│ Arch-Garch      │ Master benchmark summary of dynamic VaR, LLF, AIC, and BIC metrics     │
└─────────────────┴────────────────────────────────────────────────────────────────────────┘
```

### How to Re-Optimize GARCH Parameters using Excel Solver:
1. Navigate to the desired sheet (e.g., `Garch` for VCB, `Garch (2)` for BID).
2. Open **Data** $\rightarrow$ **Solver**.
3. **Set Objective**: Cell `H7` (Total Log-Likelihood) $\rightarrow$ **To: Max**.
4. **By Changing Variable Cells**: Cells `B6:B8` ($\omega, \alpha, \beta$).
5. **Subject to the Constraints**:
   - `B6 >= 0.000001` ($\omega > 0$)
   - `B7 >= 0` ($\alpha \ge 0$)
   - `B8 >= 0` ($\beta \ge 0$)
   - `B9 <= 0.9999` ($\alpha + \beta < 1$ for weak stationarity)
6. **Solving Method**: Select **GRG Nonlinear**.
7. Click **Solve**.

---

## Risk Management & Regulatory Policy Implications

### 1. For Portfolio Managers & Quantitative Traders
* **Reject Gaussian VaR for High-Confidence Decisions**: Using Parametric VaR at $99\%$ confidence severely understates capital exposure in Vietnam equities by up to **$28.2\%$** ($3.221\text{M}$ vs. $4.484\text{M VND}$). Historical Simulation and GARCH-filtered historical simulations should form the baseline.
* **Transition to Expected Shortfall (ES)**: To prevent moral hazard from sub-additive tail risks, allocation limits must be constrained by ES ($5.814\text{M VND}$ at $\alpha=1\%$), guarding against catastrophic black-swan drawdowns.

### 2. For Commercial Banking Institutions (Basel Framework)
* **Internal Models Approach (IMA)**: Under Basel II/III Market Risk guidelines, commercial banks must implement backtested conditional volatility models (GARCH-VaR) to dynamically calibrate minimum regulatory capital requirements (MRCR).
* **Capital Surcharges for Volatility Persistence**: Banks with high volatility persistence (such as CTG and BID) require larger Tier-1 market risk capital buffers during expansionary monetary regimes to absorb cyclical contractions.

### 3. For Financial Regulators (State Bank of Vietnam & SSC)
* **Macroprudential Tail Risk Monitoring**: Implement sector-wide VaR/ES stress-testing routines across the credit institution network to detect systemic risk spillover.
* **Development of Hedging Instruments**: Accelerate the rollout of interest-rate and index derivative contracts on the Vietnam stock market, enabling institutions to hedge uninsurable systematic risk.

---

## Academic References

1. **Bollerslev, T. (1986)**. *Generalized Autoregressive Conditional Heteroskedasticity*. Journal of Econometrics, 31(3), 307–327.
2. **Engle, R. F. (1982)**. *Autoregressive Conditional Heteroskedasticity with Estimates of the Variance of United Kingdom Inflation*. Econometrica, 50(4), 987–1007.
3. **Jorion, P. (2007)**. *Value at Risk: The New Benchmark for Managing Financial Risk* (3rd ed.). McGraw-Hill.
4. **Kupiec, P. H. (1995)**. *Techniques for Verifying the Accuracy of Risk Measurement Models*. The Journal of Derivatives, 3(2), 73–84.
5. **Christoffersen, P. F. (1998)**. *Evaluating Interval Forecasts*. International Economic Review, 39(4), 841–862.
6. **J.P. Morgan / Reuters (1996)**. *RiskMetrics — Technical Document* (4th ed.). New York.
7. **Basel Committee on Banking Supervision (2006)**. *International Convergence of Capital Measurement and Capital Standards: A Revised Framework (Basel II)*. Bank for International Settlements.
8. **Artzner, P., Delbaen, F., Eber, J. M., & Heath, D. (1999)**. *Coherent Measures of Risk*. Mathematical Finance, 9(3), 203–228.

---

## Author & Project Metadata

* **Author**: Ho Quynh My (Hồ Quỳnh My)
* **Student ID**: 2321000338
* **Major**: Quantitative Finance (*Tài chính định lượng*) — Class 23DTL02
* **Academic Supervisor**: M.Sc. Vu Anh Linh Duy (Th.S Vũ Anh Linh Duy)
* **Institution**: Faculty of Data Science, University of Finance – Marketing (UFM), Ho Chi Minh City, Vietnam
* **Date of Defense**: April 2026

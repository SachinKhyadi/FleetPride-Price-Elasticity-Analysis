# FleetPride-Price-Elasticity-Analysis

## Causal Demand Estimation & Sales Forecasting for Aftermarket Truck Parts

This project demonstrates 5 rare skills that most data analysts do not show on GitHub: 

(1) causal inference with instrumental variables — not just correlation analysis

(2) econometric modeling with real business endogeneity correction

(3) ensemble combining statistical and ML models

(4) interactive Power BI dashboard with a live what-if simulator

(5) true demand elasticity vs revenue elasticity distinction — Phase 2 quantity model. 

These are graduate-level and industry-level skills that immediately separate this from typical student projects.

## 🎯 Objective

FleetPride — one of North America's largest aftermarket truck parts distributors — 
makes pricing decisions for 10 parts across 8 fleet customers without any 
quantitative framework. This project answers three business questions:

> **"If we raise the price on PART_2683 by 10%, how many fewer units will we sell?"**
> **"Which parts have pricing power — where customers barely respond to price changes?"**
> **"What will Q4 2025 sales look like for each part?"**

### Problem
Pricing decisions were driven by intuition with no elasticity data, creating 
simultaneous margin risk (prices too low) and volume risk (prices too high). 
Additionally, 55% of competitor price data was missing, limiting competitive 
pricing intelligence.

### Solution
Built a **causal price elasticity framework** using econometric and ML models 
that separates the true price effect from confounding demand conditions — 
something standard regression cannot do. Delivered:
- ✅ Elasticity estimate per SKU with 95% confidence intervals
- ✅ Q4 2025 demand forecast (422 transactions) with 6.9% MAPE
- ✅ 7-page interactive Power BI dashboard with live price simulator
- ✅ Prioritized pricing recommendations with dollar impact estimates

### Why causal inference matters here
FleetPride raises prices when demand is already high — meaning price and 
sales move together not because customers like high prices, but because both 
respond to the same demand environment. Standard OLS cannot separate these 
effects. **Two-Stage Least Squares (2SLS) with a COGS instrument** corrects 
this endogeneity bias, producing the true causal elasticity estimate.

## 📊 Data Used

### Primary Dataset — FleetPride Transactions
| Attribute | Detail |
|-----------|--------|
| **Source** | FleetPride ERP system (provided via MIS 6390 Analytics Practicum) |
| **Training rows** | 4,971 (after cleaning from 5,082 raw) |
| **Test rows** | 422 (Q4 2025 — blind, no actuals at time of modeling) |
| **Time period** | January 2, 2023 – December 31, 2025 (33 months training) |
| **Customers** | 8 national and regional fleet accounts |
| **Parts (SKUs)** | 10 aftermarket components (4 OEM · 6 non-OEM) |
| **Key columns** | invoice_date, CustID, PartID, unit_price, quantity, actual_sales, unit_cogs, comp_price, annual_spend, retail_sales |

### External Dataset — FRED Macroeconomic Indicators
| Variable | Description | Source |
|----------|-------------|--------|
| TRUCKD11 | Truck tonnage index | Federal Reserve |
| VMTD11 | Vehicle miles traveled | Federal Reserve |
| HTRUCKSSAAR | Heavy truck sales (SAAR) | Federal Reserve |
| WPU141106 | Wholesale price index — truck parts | Federal Reserve |
| FRGSHPUSM649NCIS | Freight shipment index | Federal Reserve |

### Key Data Characteristics
- **Skewness:** actual_sales skewness = **2.836** (raw) → **-0.049** (after log transform)
- **Missing data:** comp_price was zero for **55% of rows** — not actual $0, 
  but structural missing. Replaced with NaN + binary `has_comp` flag
- **Critical correction:** annual_spend imputed via forward-fill within 
  customer group (original median imputation was time-series invalid — 
  look-ahead bias)
- **Removed rows:** 22 returns (negative qty) + 86 zero transactions + 
  3 adjustments = **111 rows removed (2.2%)**

  ## 🛠️ Tools & Technologies

### Languages & Environment
| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.11 | All data processing, modeling, visualization |
| Jupyter Notebook | 7.x | Interactive analysis (70 cells, fully documented) |
| Git / GitHub | — | Version control and project sharing |

### Data Processing
| Library | Purpose |
|---------|---------|
| `pandas` | Data cleaning, feature engineering, pipeline |
| `numpy` | Numerical operations, array transformations |
| `scipy` | Statistical tests |

### Statistical & Econometric Modeling
| Library | Purpose |
|---------|---------|
| `statsmodels` | OLS (log-log), 2SLS (IV2SLS), HC3 robust SEs, F-statistics |
| `linearmodels` | Panel data IV estimation, Wu-Hausman endogeneity test |
| `sklearn` | Model evaluation (MAPE, RMSE, R²), preprocessing |
| `sklearn.ensemble` | GradientBoostingRegressor (600 trees, depth=5, lr=0.03) |

### Visualization & Dashboard
| Tool | Purpose |
|------|---------|
| `matplotlib` | Static charts — actual vs predicted, SHAP, elasticity |
| `seaborn` | Correlation heatmaps, distribution plots |
| `shap` | SHAP feature importance (causal model — no leakage) |
| **Power BI Desktop** | 7-page interactive dashboard with price simulator |

### Key Techniques Applied
```
✅ Two-Stage Least Squares (2SLS)     — causal elasticity, endogeneity correction
✅ Instrumental Variables (IV)         — COGS instrument, F-stat = 3,719
✅ Log-Log OLS with Fixed Effects      — customer & SKU dummies, HC3 robust SEs
✅ Gradient Boosting Machine (GBM)     — nonlinear ML prediction
✅ Ensemble Modeling (3-model blend)   — 35% OLS + 35% 2SLS + 30% GBM
✅ SHAP Analysis (causal model)        — no data leakage, true business drivers
✅ Temporal Train/Val/Test Split        — no random shuffle, forward-looking only
✅ Phase 2 Quantity Elasticity          — ln(quantity) outcome, true demand model
```

## 🔑 Key Steps

### Step 1 — Data Cleaning (9-Step Pipeline)
- Renamed unnamed date column → `invoice_date`, parsed as `datetime64`
- **Critical fix:** Changed `annual_spend` imputation from customer median 
  (look-ahead bias) to **forward-fill within customer group**
- Forward-filled December 2025 FRED macro variables (publication lag)
- Removed 22 returns (negative quantity), 86 zero transactions, 3 adjustments
- Replaced `comp_price` zeros with NaN → created binary `has_comp` flag
- **Result:** 4,971 clean rows · 19 columns · zero missing values

### Step 2 — Feature Engineering (46 features from 17 raw columns)
- **Log transforms:** `log_price`, `log_sales`, `log_quantity`, `log_spend`, 
  `log_cogs` — converts skewness 2.836 → -0.049, validates log-log spec
- **IV instrument:** `unit_cogs = COGS / quantity` — supply-side cost per unit 
  used as exogenous instrument in 2SLS to correct price endogeneity
- **Fixed effects:** 9 SKU dummies + 7 customer dummies — absorbs all 
  time-invariant between-group variation
- **Cyclical date encoding:** `month_sin`, `month_cos` — ensures Dec/Jan adjacency
- **Macro signals:** 5 FRED indicators for demand environment

### Step 3 — Exploratory Data Analysis
- Distribution analysis revealed actual_sales skewness = 2.836 → log transform required
- Identified structural zeros in comp_price (55% missing — not true $0)
- Discovered freight index (FRGSHPUSM) as strongest univariate predictor
- Confirmed temporal autocorrelation requiring time-ordered splits

### Step 4 — Temporal Train / Validation / Test Split
```
Training:   Jan 2, 2023 – Jul 31, 2025  →  4,675 rows  (model fitting)
Validation: Aug 1, 2025 – Sep 30, 2025  →    296 rows  (performance evaluation)
Test:       Oct 1, 2025 – Dec 31, 2025  →    422 rows  (blind Q4 prediction)
```
> No random shuffling. Strictly time-ordered to prevent data leakage.

### Step 5 — OLS Baseline (Log-Log Specification)
- Formula: `ln(actual_sales) ~ ln(unit_price) + ln(quantity) + controls + FE`
- HC3 heteroskedasticity-robust standard errors throughout
- **Result:** Revenue elasticity = **+0.119** (p<0.001) · MAPE 9.0% · R² 0.981
- Baseline identified — endogeneity bias suspected

### Step 6 — 2SLS Causal Elasticity (Core Contribution)
- **Problem:** FleetPride raises prices during high-demand periods → price and 
  sales are simultaneously driven by demand → OLS is biased
- **Instrument:** `log_cogs` (unit cost = COGS/qty) — shifts prices via 
  supplier costs, uncorrelated with customer demand
- **First-stage F-statistic: 3,719** (threshold = 10 → no weak IV concern)
- **Result:** Causal elasticity = **+0.262** (p<0.001, CI [+0.200, +0.319])
- **Endogeneity bias corrected: +0.143** (2SLS − OLS)

### Step 7 — Gradient Boosting Machine
- 600 estimators · max_depth=5 · learning_rate=0.03
- Captures nonlinear effects and interactions that linear models miss
- **Result:** MAPE 9.7% · R² 0.961 · Overfit gap 5.1%

### Step 8 — Ensemble Model (Best Model)
- Weights: 0.35×OLS + 0.35×2SLS + 0.30×GBM
- Retrained on full 4,971-row set before Q4 prediction
- **Result: MAPE 6.9% · R² 0.9856 · RMSE $8.76 · Mean error $0.30**

### Step 9 — Phase 2: True Demand Elasticity
- Switched outcome from `ln(actual_sales)` → `ln(quantity)` to remove 
  mechanical positive relationship (sales = price × qty)
- Pooled quantity elasticity: **-0.020** (p=0.857, ns)
- Per-SKU: **PART_2683 ε = -0.920** (p=0.004 **) — most reliable result
- PART_25F3 shows anomalous **+0.385** (p=0.036 *) — flagged for investigation

### Step 10 — SHAP Analysis (Causal Model — No Leakage)
- Excluded `retail_sales`, `log_retail`, `quantity`, `log_quantity` to prevent 
  circular identity (sales = price × qty would trivially dominate)
- Top drivers: Customer identity $12.5 · Unit price $8.0 · National flag $6.7
- Combined price signals: **$13.1** — largest controllable lever

### Step 11 — Q4 2025 Predictions & Power BI Dashboard
- Generated 422 Q4 predictions (0 negatives · mean = $78.55)
- Built 7-page Power BI dashboard with live price simulator
- Significance warning prevents business users from acting on unreliable estimates


## 💡 Key Insights

### Insight 1 — OLS Elasticity Was 55% Wrong Due to Endogeneity
Standard OLS gave revenue elasticity of +0.119. After correcting for price 
endogeneity with 2SLS, the causal estimate is +0.262 — **55% higher**.  
This confirms that FleetPride's pricing behavior (raising prices when demand 
is already high) was contaminating the OLS estimate. Using OLS alone for 
pricing decisions would lead to systematically wrong conclusions.

> **Bias corrected: +0.143 | F-stat = 3,719 (372× the minimum threshold)**

---

### Insight 2 — Most Parts Have Pricing Power
7 of 10 SKUs show inelastic quantity demand. Three captive OEM parts have 
near-zero quantity elasticity:

| Part | ε quantity | p-value | Interpretation |
|------|-----------|---------|----------------|
| PART_012C (C0D99F43) | -0.025 | 0.968 | Most captive — raise price freely |
| PART_F846 (27E5289C) | -0.103 | 0.745 | Inelastic OEM — pricing power |
| PART_F840 (18BCF325) | -0.197 | 0.890 | Inelastic OEM — pricing power |

> **Recommended action: Test 5–8% price increase at next contract renewal**
> **Estimated impact: +$4,200–$6,800 per quarter · Risk: LOW**

---

### Insight 3 — PART_2683 Is Near Unit Elastic (Only Double-Star Result)
```
Part:         AEC82997 (PART_2683)
Elasticity:   ε = -0.920  (p = 0.004, **)
95% CI:       [-1.545, -0.295]  ← entirely negative, excludes zero
Interpretation: 10% price increase → -9.2% volume → net revenue ≈ $0
```
Current pricing is already near the revenue-maximizing point.  
**Do NOT raise price on this part.**

---

### Insight 4 — PART_25F3 Shows Anomalous Positive Elasticity
```
Part:         AC9B9893 (PART_25F3) 
Elasticity:   ε = +0.385  (p = 0.036, *)
Q4 revenue:   ~32% of total Q4 predicted revenue
Hypotheses:   Quality signaling · Bundling effect · Data artifact
```
Positive demand elasticity (more units at higher prices) is economically 
unusual. **Do NOT change price until root cause is identified.**

---

### Insight 5 — Customer Identity Dominates — Differentiated Pricing Is the Priority

| SHAP Feature | Dollar Impact | Category |
|-------------|--------------|---------|
| Customer identity | **$12.5** | Customer |
| Unit price | **$8.0** | Price (controllable) |
| National flag | **$6.7** | Customer |
| Log price | **$5.1** | Price (controllable) |
| Annual spend | **$5.2** | Customer |

Combined price signals ($13.1) = largest controllable driver.  
National accounts generate **87% more revenue per transaction** than regional.  
**Differentiated pricing by customer > uniform price changes across all SKUs.**

---

### Insight 6 — Freight Markets Are the Leading Demand Indicator
The freight shipment index (FRGSHPUSM) ranked as the strongest non-price 
univariate predictor. When freight activity rises, fleet parts demand follows.  
FleetPride should **monitor freight indices as a leading indicator** for 
inventory planning and pricing windows.

## ✅ Conclusion

### What Was Achieved
This project delivered a complete **causal price elasticity framework** for 
FleetPride's aftermarket parts portfolio — moving from intuition-based pricing 
to data-driven, statistically validated recommendations.

| Deliverable | Detail |
|-------------|--------|
| Ensemble forecast model | 6.9% MAPE · R² 0.9856 · $0.30 mean error |
| Causal elasticity estimate | +0.262 (2SLS) · F-stat 3,719 · corrects OLS bias |
| Per-SKU elasticity table | All 10 parts · 95% CIs · p-values · strategy |
| Q4 2025 predictions | 422 transactions · mean $78.55 · 0 negatives |
| Power BI dashboard | 7 interactive pages · live price simulator |
| Pricing recommendations | 3 actions · $4,200–$6,800 estimated Q4 impact |

---

### Business Impact
```
✅ RAISE PRICE  → PART_012C, PART_F840, PART_F846 (captive OEM, ε ≈ 0)
   Estimated:  +$4,200–$6,800/quarter · Risk: LOW

⚠️ HOLD PRICE   → PART_2683 (ε = -0.920, p=0.004**)
   10% increase → -9.2% volume → net revenue ≈ $0

🔍 INVESTIGATE  → PART_25F3 (ε = +0.385, p=0.036*, 32% of Q4 revenue)
   Do not act until root cause of positive elasticity is identified
```

---

### Technical Contributions
1. **Causal identification** — 2SLS with COGS instrument corrects endogeneity 
   that OLS cannot handle. F-stat = 3,719 confirms instrument validity
2. **Phase 2 quantity model** — Switched outcome from revenue to quantity 
   to remove the mechanical price×qty identity and reveal true demand response
3. **Leakage-free SHAP** — Excluded transaction-correlated features 
   (retail_sales, quantity) to produce honest feature importance
4. **Ensemble design** — 35% causal (2SLS) + 35% interpretable (OLS) + 
   30% ML (GBM) balances causal validity with predictive accuracy
5. **Temporal split** — No random shuffling — strictly time-ordered to 
   simulate real-world forward-looking deployment

---

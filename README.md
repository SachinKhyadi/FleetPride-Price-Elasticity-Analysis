# 🚚 FleetPride Price Elasticity Analysis
### Causal Demand Estimation & Sales Forecasting for Aftermarket Truck Parts

## 🎯 Objective

FleetPride manages pricing for 10 aftermarket truck parts across 8 fleet customers
without any quantitative demand framework. This project answers:

> *"If we raise the price on a specific part by 10%, how much volume do we lose and does revenue go up or down?"*

**The core challenge is causal, not predictive.** FleetPride raises prices when demand
is already high meaning price and sales are simultaneously driven by the same demand
environment. Standard OLS cannot separate these effects. We use **Two-Stage Least Squares
(2SLS) with a COGS instrument** to isolate the true causal price effect.

**Deliverables:**
- ✅ Causal elasticity estimate per SKU with 95% confidence intervals
- ✅ Q4 2025 forecast (422 transactions, 6.9% MAPE)
- ✅ 5-page interactive Power BI dashboard with live price simulator
- ✅ Prioritized pricing recommendations with dollar impact

---

## 📊 Data Used

| Dataset | Description |
|---------|-------------|
| FleetPride transactions | 5,082 raw rows · Jan 2023–Sep 2025 · 17 columns |
| FRED macro indicators | 5 variables: truck tonnage, VMT, freight index, WPI, heavy truck sales |
| Test set (blind) | 422 Q4 2025 transactions · no actuals at modeling time |

**Key data facts:**
- actual_sales skewness = **2.836** (raw) → **-0.049** (after log transform)
- comp_price zero for **55%** of rows structural missing, not $0
- annual_spend corrected from median to **forward-fill within customer** (time-series valid)
- **111 rows removed** (returns, zeros, adjustments) → 4,971 clean rows

---

## 🛠️ Tools & Technologies

| Category | Tool/Library |
|----------|-------------|
| Language | Python 3.11 · Jupyter Notebook |
| Data processing | pandas · numpy · scipy |
| Statistical modeling | statsmodels (OLS, IV2SLS, HC3) · linearmodels |
| Machine learning | scikit-learn (GBM, metrics) |
| Explainability | shap (causal model — no leakage) |
| Dashboard | Power BI Desktop (7 pages · what-if simulator) |
| Visualization | matplotlib · seaborn |

---

## 🔑 Key Steps

```
1. Data Cleaning       → 9-step pipeline · 111 rows removed · annual_spend fix
2. Feature Engineering → 46 features · log transforms · COGS IV · FE dummies
3. EDA                 → skewness discovery · missing comp_price · freight index
4. Temporal Split      → Train Jan23–Jul25 · Val Aug–Sep25 · Test Oct–Dec25
5. OLS Baseline        → log-log · HC3 SEs · ε = +0.119 · MAPE 9.0%
6. 2SLS Causal Model   → COGS instrument · F-stat 3,719 · ε = +0.262 · MAPE 8.5%
7. GBM Model           → 600 trees · depth=5 · MAPE 9.7% · overfit gap 5.1%
8. Ensemble            → 35%OLS + 35%2SLS + 30%GBM → MAPE 6.9% · R² 0.9856
9. Quantity Elasticity → ln(quantity) outcome · PART_2683 ε=-0.920 (p=0.004**)
10. SHAP Analysis      → causal model · customer identity $12.5 · price $13.1
11. Dashboard + Recs   → Power BI · 422 Q4 predictions · 3 pricing actions
```

---

## 💡 Key Insights

**1. OLS Was 55% Wrong** — Endogeneity bias caused OLS to underestimate elasticity
by 0.143. 2SLS corrects this: +0.262 vs OLS +0.119.

**2. Most Parts Have Pricing Power** — 7/10 SKUs show inelastic demand. Three
captive OEM parts (PART_012C, F840, F846) have ε ≈ 0 — customers cannot substitute.

**3. PART_2683 Is Near Unit Elastic** — Only double-star result: ε = -0.920
(p=0.004). A 10% price increase loses ~9.2% volume → net revenue ≈ $0.
**Do not raise this price.**

**4. PART_25F3 Is Anomalous** — Positive significant elasticity +0.385 (p=0.036)
and 32% of Q4 revenue. Do not act until root cause identified.

**5. Customer Identity > Price** — SHAP: customer identity $12.5 vs unit price $8.0.
National accounts generate 87% more revenue per transaction.
Differentiated pricing by customer > uniform changes.

**6. Freight Markets Are a Leading Indicator** — Freight shipment index is the
strongest non-price driver. Monitor as early signal for demand planning.

---

## ✅ Results & Recommendations

### Model Performance

| Model | Val MAPE | Val R² | RMSE | Overfit Gap |
|-------|----------|--------|------|-------------|
| OLS | 9.02% | 0.9809 | $9.82 | 2.39% |
| 2SLS | 8.50% | 0.9796 | $10.15 | 1.98% |
| GBM | 9.71% | 0.9606 | $14.10 | 5.07% |
| **Ensemble ★** | **6.90%** | **0.9856** | **$8.76** | **1.52%** |

### Pricing Recommendations

| Priority | Action | Part(s) | Evidence | Est. Impact |
|----------|--------|---------|---------|------------|
| 🟢 HIGH | Raise price 5–8% | PART_012C, F840, F846 | ε ≈ 0, captive OEM | +$4,200–$6,800/qtr |
| 🟠 HIGH | Hold price | PART_2683 | ε = -0.920** | ~$0 net if raised |
| 🔵 MEDIUM | Investigate | PART_25F3 | ε = +0.385* | Unknown |

---

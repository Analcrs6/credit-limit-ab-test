# Credit Limit Increase A/B Test: LTV-Driven Segmentation Strategy

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![pandas](https://img.shields.io/badge/pandas-2.0+-lightgrey)
![scipy](https://img.shields.io/badge/scipy-1.11+-lightgrey)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

## Overview

This project simulates a randomized A/B experiment on real LendingClub loan 
data (2007–2018) to evaluate whether proactively offering credit limit increases 
to eligible borrowers improves 24-month customer lifetime value (LTV).

The analysis goes beyond a simple significance test — it builds a full 
revenue/risk LTV model, identifies a segment where the offer destroys value, 
and delivers a segmented rollout recommendation grounded in both statistical 
evidence and business logic.

---

## Key Results

| Test | Δ LTV | 95% CI | P-value | Decision |
|---|---|---|---|---|
| All grades | +$151 | [-$31, +$332] | 0.104 | Inconclusive |
| Excl. Grade D | +$225 | [+$36, +$414] | **0.019** | **Roll out** |

**Finding:** The offer is net positive for Grades A, B, C, E, F, G (+$172 to 
+$441 LTV per customer) but destroys value for Grade D (-$274), which drags 
the overall test to non-significance. A segmented rollout excluding Grade D 
yields a statistically significant and commercially meaningful result.

---

## Project Structure
```
├── notebook/
│   └── credit_limit_ab_test.ipynb   # Full analysis notebook
├── figures/
│   └── ab_test_ltv_analysis.png     # Key results visualization
└── README.md
```

---

## Methodology

### 1. Data
- **Source:** [LendingClub Loan Data 2007–2018](https://www.kaggle.com/datasets/wordsforthewise/lending-club) via Kaggle
- **Size:** 2.2M loans, 145 columns — loaded with `usecols` (29 columns) and downsampled to 500k rows for memory efficiency
- **Cleaning:** Removed outlier DTI (>60) and revolving utilization (>100%), filtered to terminal loan statuses only (Fully Paid / Charged Off), constructed binary default flag

### 2. Experiment Design

**Eligibility criteria:**

| Criterion | Threshold |
|---|---|
| FICO score | 620–720 |
| Revolving utilization | < 60% |
| Debt-to-income ratio | < 35% |
| Delinquencies (2yr) | 0 |

**Eligible pool:** 99,662 borrowers (33.4% of clean sample)

**Sample size:** Two-proportion z-test, MDE = 3pp, α = 0.05, power = 80% → 4,340 per arm

**Randomization check:** Chi-square SRM test on grade distribution → p = 0.082, no imbalance detected

### 3. LTV Model
```
LTV = Gross Interest Revenue + Fee Revenue
    − Credit Loss − Cost of Funds − Servicing Cost
```

| Parameter | Value |
|---|---|
| Limit increase (treatment) | +30% |
| Drawdown rate (control / treatment) | 45% / 48% |
| Loss given default (LGD) | 60% |
| Origination fee | 1% |
| Cost of funds | 4% annual |
| Servicing cost | $8/month |
| LTV horizon | 24 months |

### 4. Statistical Testing
- Primary test: Welch's two-sample t-test on LTV
- Guardrail: Default rate must not exceed control + 2pp
- Segmented test: Repeat analysis excluding Grade D

---

## Visualizations

Four panels produced by the analysis:

- **LTV delta by grade** — bar chart showing which segments win/lose
- **LTV distribution** — treatment vs control density overlay
- **Revenue vs cost breakdown** — grouped bar chart per variant
- **LTV delta vs default rate increase** — bubble chart by grade (rollout decision map)

---

## Limitations

- Drawdown behavior is simulated, not observed from a live experiment
- LGD and cost of funds are industry benchmarks, not lender-specific
- No novelty effect correction applied
- Data covers 2007–2018; current credit environment may differ
- `Current` loans excluded — final outcomes unknown at analysis time

---

## Next Steps

1. Re-test Grade D with a smaller limit increase (+15%) to find the risk-adjusted optimum
2. Monitor default rate monthly post-launch against the +2pp guardrail
3. Further segment Grade B/C by income band and customer tenure
4. Apply propensity score matching to strengthen causal claims

---

## Tech Stack

- **Python 3.10+**
- `pandas` — data manipulation
- `numpy` — numerical simulation
- `scipy` — statistical testing
- `matplotlib` — visualization
- `kagglehub` — dataset download

---

## How to Run
```bash
# 1. Clone the repo
git clone https://github.com/Anaislcrs/credit-limit-ab-test
cd credit-limit-ab-test

# 2. Install dependencies
pip install pandas numpy scipy matplotlib kagglehub

# 3. Download the dataset
python -c "import kagglehub; kagglehub.dataset_download('wordsforthewise/lending-club')"

# 4. Open the notebook
jupyter notebook notebook/credit_limit_ab_test.ipynb
```

> **Note:** The dataset is ~1.5GB. The notebook uses `usecols` and downsampling 
> to keep memory under 300MB. A Kaggle account is required for dataset download.

---

## Author

**Anaïs Lacreuse** · [GitHub](https://github.com/Anaislcrs)  
MS Data Science — Pace University, 2025

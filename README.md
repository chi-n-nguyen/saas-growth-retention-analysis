# RavenStack SaaS Growth & Retention Analysis

A multi-table subscription analytics project analysing 500 accounts, 5,000 subscription records, and 600 churn events from the RavenStack synthetic SaaS dataset.

**Tools:** Python (pandas, matplotlib, seaborn) · Excel (openpyxl) · Jupyter Notebooks
**Dataset:** [RavenStack by River @ Rivalytics](https://rivalytics.medium.com) (MIT-like licence — fully synthetic, no PII)

---

## Project Structure

```
ravenstack-saas-analysis/
├── data/
│   ├── accounts.csv
│   ├── subscriptions.csv
│   ├── churn_events.csv
│   ├── feature_usage.csv
│   └── support_tickets.csv
├── notebooks/
│   ├── 01_cohort_retention.ipynb
│   ├── 02_mrr_bridge.ipynb
│   └── 03_unit_economics.ipynb
├── outputs/
│   ├── cohort_heatmap.png
│   ├── retention_by_tier.png
│   ├── mrr_bridge.png
│   ├── unit_economics_charts.png
│   └── unit_economics_table.xlsx
└── README.md
```

---

## Analyses

### 1 · Cohort Retention (`01_cohort_retention.ipynb`)

Joined `accounts.csv` + `subscriptions.csv` on `account_id`. Grouped accounts by signup month (cohort) and tracked what percentage remained active at Month 1, 3, 6, and 12 checkpoints.

**Output:** Cohort heatmap (all cohorts × 4 time checkpoints) + retention curves by plan tier.

![Cohort Heatmap](outputs/cohort_heatmap.png)

![Retention by Tier](outputs/retention_by_tier.png)

---

### 2 · MRR Bridge (`02_mrr_bridge.ipynb`)

Decomposed month-over-month MRR movement from `subscriptions.csv` into:

- **New MRR** — paid subscriptions started this month (non-upgrade)
- **Expansion MRR** — upgrade-flagged subscriptions started this month
- **Churned MRR** — subscriptions ended with `churn_flag = True`
- **Net New MRR** — New + Expansion − Churned

**Output:** Stacked bar + Net MRR overlay + cumulative growth chart.

![MRR Bridge](outputs/mrr_bridge.png)

---

### 3 · Unit Economics & Churn Drivers (`03_unit_economics.ipynb`)

Calculated per-tier unit economics (Avg MRR, Avg Lifespan, LTV, Monthly Churn Rate, LTV:MRR Ratio) and cross-tabulated churn reason codes by plan tier from `churn_events.csv`.

**Output:** Formatted Excel workbook (3 sheets: Unit Economics · Churn Drivers · Strategic Recommendations) + charts.

![Unit Economics](outputs/unit_economics_charts.png)

---

## Key Findings

- **Month 6–12 is the critical churn window.** Average retention across all cohorts drops from 91.3% at Month 6 to 78.8% at Month 12 — a 12.5 percentage point fall, steeper than any earlier interval. Retention holds relatively stable in the first 6 months (Month 1: 97.4%, Month 3: 94.9%), making the second half of year one the highest-risk period for churn.

- **Enterprise LTV ($15,775) is 8.9× Basic LTV ($1,765).** The tier differential is driven by MRR rather than lifespan — Enterprise accounts pay $5,787/month vs $560/month for Basic, with comparable average subscription lengths across tiers. This confirms a strong upsell ROI: converting one Basic account to Enterprise replaces the LTV of ~9 Basic accounts.

- **Support issues drive Basic and Pro churn (21% and 19% respectively); Enterprise churns primarily on missing features (20%).** This points to two separate retention strategies: self-serve support improvements for lower tiers, and roadmap communication / feature delivery prioritisation for Enterprise accounts.

---

## How to Run

```bash
# 1. Install dependencies
pip install pandas numpy matplotlib seaborn openpyxl jupyter python-dateutil

# 2. Run notebooks (from project root)
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/01_cohort_retention.ipynb
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/02_mrr_bridge.ipynb
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/03_unit_economics.ipynb

# Or open interactively
jupyter notebook
```

Outputs are written to `outputs/`. All notebooks are self-contained and read from `data/`.

---

## Data Attribution

Dataset: **RavenStack** synthetic SaaS dataset by River @ Rivalytics.
Licence: MIT-like (educational/portfolio use permitted with credit).
[https://rivalytics.medium.com](https://rivalytics.medium.com)

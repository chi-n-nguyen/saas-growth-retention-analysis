# RavenStack SaaS Growth & Retention Analysis

Subscription analytics project covering 500 accounts, 5,000 records, and 600 churn events across 24 months. Four notebooks: SQL extraction layer, cohort retention, MRR bridge, and unit economics with NRR.

**Tools:** Python (pandas, matplotlib, seaborn) · SQL (SQLite) · Excel (openpyxl) · Jupyter Notebooks

**Dataset:** [SaaS Subscription & Churn Analytics Dataset](https://www.kaggle.com/datasets/rivalytics/saas-subscription-and-churn-analytics-dataset)

**Full analysis:** [REPORT.md](REPORT.md)

---

## Headline Numbers

| Metric | Value |
|---|---|
| Month-12 retention (avg) | 78.8% |
| Steepest churn window | Month 6-12 (-12.5pp) |
| Net MRR (24 months) | $10,159,608 |
| Enterprise LTV | $15,775 (8.9x Basic) |
| Enterprise NRR | 329.5% |
| Top churn driver, Basic/Pro | support (21%, 19%) |
| Top churn driver, Enterprise | features (20%) |

---

## Project Structure

```
ravenstack-saas-analysis/
├── data/
│   ├── accounts.csv
│   ├── subscriptions.csv
│   └── churn_events.csv
├── notebooks/
│   ├── 00_sql_queries.ipynb
│   ├── 01_cohort_retention.ipynb
│   ├── 02_mrr_bridge.ipynb
│   └── 03_unit_economics.ipynb
├── outputs/
│   ├── cohort_heatmap.png
│   ├── retention_by_tier.png
│   ├── retention_by_industry.png
│   ├── retention_by_referral.png
│   ├── mrr_bridge.png
│   ├── nrr_trend.png
│   ├── unit_economics_charts.png
│   └── unit_economics_table.xlsx
├── REPORT.md
└── README.md
```

---

## SQL Layer

Raw CSVs are loaded into a local SQLite database. Core aggregations run as SQL queries before results are pulled into pandas. See [`00_sql_queries.ipynb`](notebooks/00_sql_queries.ipynb).

SQL features demonstrated: `CASE WHEN` aggregation, window functions (`PARTITION BY`), CTEs (`WITH`), `JULIANDAY` date arithmetic, multi-table `JOIN`.

---

## How to Run

```bash
# 1. Install dependencies
pip install pandas numpy matplotlib seaborn openpyxl jupyter python-dateutil

# 2. Run notebooks (from project root)
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/00_sql_queries.ipynb
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/01_cohort_retention.ipynb
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/02_mrr_bridge.ipynb
jupyter nbconvert --to notebook --execute --output-dir=notebooks notebooks/03_unit_economics.ipynb

# Or open interactively
jupyter notebook
```

Outputs are written to `outputs/`. All notebooks are self-contained and read from `data/`.

---



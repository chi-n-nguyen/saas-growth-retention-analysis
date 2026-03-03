# RavenStack SaaS: Growth & Retention Analysis Report

**Dataset:** RavenStack synthetic SaaS dataset (Kaggle, by River @ Rivalytics)
**Period:** January 2023 - January 2025 (24 months)
**Scope:** 500 accounts · 5,000 subscriptions · 600 churn events

---

## 1. Executive Summary

RavenStack generated $10.16M in net MRR over 24 months, with every month net-positive. Average Month-12 retention is 78.8%, with a 12.5pp drop concentrated in the Month 6-12 window. Enterprise accounts carry 8.9x the LTV of Basic accounts and the highest NRR (330%), making them the primary expansion lever. Basic and Pro churn is driven by support issues (21%, 19%); Enterprise churn is driven by feature gaps (20%).

| Metric | Value |
|---|---|
| Month-12 retention (avg) | 78.8% |
| Steepest churn window | Month 6-12 (-12.5pp) |
| Net MRR generated (24 mo) | $10,159,608 |
| Enterprise LTV | $15,775 (8.9x Basic) |
| Enterprise NRR | 329.5% |
| Top churn driver, Basic/Pro | support (21%, 19%) |
| Top churn driver, Enterprise | features (20%) |

---

## 2. Dataset

| File | Rows | Key Fields |
|---|---|---|
| accounts.csv | 500 | account_id, plan_tier, industry, referral_source, signup_date |
| subscriptions.csv | 5,000 | subscription_id, account_id, plan_tier, mrr_amount, start_date, end_date, churn_flag, upgrade_flag, downgrade_flag, is_trial |
| churn_events.csv | 600 | account_id, churn_date, reason_code |

**Plan tiers:** Basic, Pro, Enterprise
**Industries:** FinTech, Cybersecurity, HealthTech, EdTech, DevTools
**Referral sources:** ads, organic, referral, partner
**Churn reason codes:** pricing, support, budget, unknown, features, competitor

---

## 3. SQL Layer

All core aggregations are first run as SQL queries against a local SQLite database before results are pulled into pandas for visualisation. See `notebooks/00_sql_queries.ipynb`.

**Stack:** CSVs loaded via `pandas.to_sql()` into SQLite; queries executed with `pd.read_sql_query()`.

| Query | SQL techniques |
|---|---|
| Monthly MRR by tier | `SUM(CASE WHEN ...)`, `strftime` date functions, `GROUP BY` |
| Churn drivers by tier | Window function: `COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY plan_tier)` |
| Unit economics | `AVG(JULIANDAY(end_date) - JULIANDAY(start_date))` date arithmetic |
| Cohort active rate | CTE (`WITH latest_status AS ...`), `LEFT JOIN`, conditional aggregation |
| Industry retention | Multi-table `LEFT JOIN`, `COUNT(DISTINCT ...)` |

---

## 4. Cohort Retention Analysis

### Methodology

Accounts are grouped by signup month. A cohort member is "active" at checkpoint M if their `last_active` date exceeds the checkpoint date (`signup_date + M months`). Only cohorts with enough elapsed time to reach a given checkpoint are included in that calculation. Analysis date: 1 Jan 2025.

### Overall Retention

| Checkpoint | Avg Retention | Interval Drop |
|---|---|---|
| Month 1 | 97.4% | -2.6pp from Month 0 |
| Month 3 | 94.9% | -2.5pp from Month 1 |
| Month 6 | 91.3% | -3.6pp from Month 3 |
| Month 12 | 78.8% | -12.5pp from Month 6 |

The first 6 months lose 8.7pp in total. The following 6 months lose 12.5pp - 44% more than the entire first half-year. The Month 6-12 window is the highest-risk period across every segmentation cut.

![Cohort Heatmap](outputs/cohort_heatmap.png)

### Retention by Plan Tier

| Tier | Month-12 Retention |
|---|---|
| Pro | 80.0% |
| Basic | 79.4% |
| Enterprise | 74.7% |

Enterprise underperforms at Month 12 despite the highest contract values. The churn driver analysis (Section 6) explains why: Enterprise churns on feature gaps, a structural product issue rather than the operational support failures that drive lower-tier churn.

![Retention by Tier](outputs/retention_by_tier.png)

### Retention by Industry

| Industry | Month-12 Retention |
|---|---|
| FinTech | 90.9% |
| Cybersecurity | ~85% |
| HealthTech | ~82% |
| EdTech | ~78% |
| DevTools | 66.7% |

FinTech leads DevTools by 24.2pp at Month 12. This is the largest segmentation gap in the dataset. FinTech accounts are long-term, compliance-driven buyers; DevTools customers churn faster as tooling needs change. This gap has direct implications for CSM resource allocation and upsell targeting.

![Retention by Industry](outputs/retention_by_industry.png)

### Retention by Referral Source

| Referral Source | Month-12 Retention |
|---|---|
| ads | 83.3% |
| referral | ~79% |
| partner | ~77% |
| organic | 74.4% |

Ads-acquired accounts retain 8.9pp higher than organic at Month 12. Counterintuitive but consistent: paid acquisition likely selects for higher purchase intent and clearer use cases at signup, reducing early churn from poor fit. Organic signups may have more exploratory intent and lower switching costs.

![Retention by Referral Source](outputs/retention_by_referral.png)

---

## 5. MRR Bridge Analysis

### Methodology

Five-component bridge built from `subscriptions.csv`:

1. **New MRR**: paid subscriptions with no upgrade or downgrade flag, starting this month
2. **Expansion MRR**: subscriptions where `upgrade_flag = True`, starting this month
3. **Contraction MRR**: per downgrade pair, `max(prev_mrr - new_mrr, 0)`, where the prior subscription is the most recent ended subscription for the same account
4. **Churned MRR**: subscriptions where `churn_flag = True`, ending this month, excluding accounts that merely downgraded
5. **Net New MRR**: New + Expansion - Contraction - Churned

### 24-Month Results

| Component | Total |
|---|---|
| New MRR | $10,075,750 |
| Expansion MRR | $1,262,997 |
| Contraction MRR | $22,700 |
| Churned MRR | $1,179,139 |
| Net MRR | $10,159,608 |

All 24 months were net-positive. Peak new MRR: December 2024 ($2,113,247).

Contraction MRR ($22,700) represents 2% of Churned MRR ($1,179,139). Revenue loss is almost entirely from full churn, not downgrades: 188 downgrade events generated minimal MRR reduction. Retention investment should target full churn prevention, not downgrade mitigation.

![MRR Bridge](outputs/mrr_bridge.png)

---

## 6. Unit Economics & NRR

### LTV by Plan Tier

| Tier | Avg MRR | Avg Lifespan* | LTV | Monthly Churn | LTV:MRR |
|---|---|---|---|---|---|
| Basic | $560 | 3.2 mo | $1,765 | 11.1% | 3.2x |
| Pro | $1,501 | 2.5 mo | $3,758 | 11.1% | 2.5x |
| Enterprise | $5,787 | 2.7 mo | $15,775 | 11.2% | 2.7x |

*Avg Lifespan from churned subscriptions only. Active subscriptions are right-censored and excluded; reported values understate true lifespan across all tiers.

Enterprise LTV is 8.9x Basic. The gap is entirely MRR-driven: all three tiers have comparable subscription lifespans (2.5-3.2 months) and near-identical monthly churn rates (~11%). Upgrading one Basic account to Enterprise replaces the LTV of approximately 9 Basic accounts.

![Unit Economics](outputs/unit_economics_charts.png)

### Net Revenue Retention (NRR)

**Methodology:** Trailing 12M NRR measures revenue retained and expanded from existing customers. Cohort: all accounts with active paid MRR on 1 Jan 2024. NRR = sum(MRR of cohort on 1 Jan 2025) / sum(MRR of cohort on 1 Jan 2024). Churned accounts contribute $0 at the end date. NRR above 100% means existing customers generate more revenue than they did 12 months prior.

| Tier | Trailing 12M NRR |
|---|---|
| Basic | 248.3% |
| Pro | 298.5% |
| Enterprise | 329.5% |
| Overall | 290.4% |

All tiers exceed 100%. Enterprise at 329.5% means the Jan 2024 Enterprise cohort generated 3.3x its starting MRR by Jan 2025 through subscription expansion alone - no new logos required.

Monthly trailing 12M NRR fell from 588.7% (Feb 2024) to 290.4% (Jan 2025) as the customer base grew and expansion normalised relative to a larger denominator. The downward trend reflects maturation, not deterioration.

![NRR Trend](outputs/nrr_trend.png)

---

## 7. Churn Driver Analysis

### Cross-Tab: Reason Code by Plan Tier (% of tier churns)

| Reason | Basic | Pro | Enterprise |
|---|---|---|---|
| support | 20.6% | 19.1% | 12.3% |
| features | 18.9% | 18.2% | **20.0%** |
| budget | 16.7% | 17.8% | 17.4% |
| unknown | 13.9% | 15.6% | 17.9% |
| pricing | 16.1% | 13.3% | 16.4% |
| competitor | 13.9% | 16.0% | 15.9% |

### Dominant Reason by Tier

| Tier | Top Reason | Share |
|---|---|---|
| Basic | support | 21% |
| Pro | support | 19% |
| Enterprise | features | 20% |

Support is the dominant churn driver for both self-serve tiers. Enterprise churn is led by missing features at 20%, well above the Enterprise support share of 12.3%. This divergence points to a product-fit gap at the high end: Enterprise accounts have more complex requirements that the current feature set does not fully address.

---

## 8. Strategic Recommendations

### Priority 1: Build Month-6 churn-risk alerts

The 6-12 month window loses 12.5pp - more than the first 6 months combined. Accounts hitting 5-month tenure with below-median feature usage are the highest-risk cohort.

**Immediate action:** Build a dashboard flagging accounts with tenure of 4-6 months and feature usage below the 50th percentile. Route these accounts to CSM outreach before they enter the high-churn window.

**Targeting layer:** Organic-acquired accounts retain 8.9pp lower than ads-acquired at Month 12 (74.4% vs 83.3%). Within the 5-7 month tenure band, weight intervention resources toward organic signups.

### Priority 2: Treat Enterprise NRR as the primary growth lever

Enterprise NRR is 329.5%. Existing Enterprise accounts triple revenue in 12 months without new logos. With an 8.9x LTV differential and the highest NRR tier, deepening one Enterprise relationship outperforms equivalent spend on Basic acquisition, where 11% monthly churn offsets CAC within months.

**Immediate action:** Set an internal target for Enterprise account expansion MRR as a standalone KPI - separate from new logo ARR. Assign dedicated expansion-focused CSMs to accounts in their first 12 months.

**Conversion path:** Prioritise direct Basic-to-Enterprise conversion over the incremental Basic-to-Pro-to-Enterprise ladder. The LTV jump at Enterprise justifies a higher-touch sales motion at conversion.

### Priority 3: Segment retention playbooks and weight CSM toward FinTech

**Basic and Pro:** Support-driven churn (21%, 19%). Fixes: reduce ticket SLAs, invest in self-serve documentation and in-app guides, deploy automated 30/60/90-day onboarding sequences.

**Enterprise:** Feature-driven churn (20%). Fixes: publish a public roadmap with delivery dates, run quarterly business reviews that tie roadmap milestones to Enterprise customer needs, create a structured feature request pipeline visible to customers.

**Industry weighting:** FinTech retains at 90.9% vs DevTools 66.7% at Month 12 (24.2pp gap). FinTech accounts are the highest-retention and highest-LTV cohort in the dataset. CSM renewal effort, upsell programs, and case study investment should be weighted toward FinTech customers first.

---

## 9. Methodology Notes

- **Right-censored lifespans:** Avg Lifespan in the unit economics table is computed from churned subscriptions only. Active subscriptions have unknown end dates and are excluded. Reported values understate true average lifespan for all tiers.
- **NRR cohort definition:** The Jan 2024 cohort includes all accounts with at least one active, paid (non-trial) subscription on 1 Jan 2024. The monthly NRR trend applies the same definition to each start month.
- **Contraction MRR:** Computed by matching each `downgrade_flag=True` subscription to the most recent prior subscription for the same account. Contraction = `max(prev_mrr - new_mrr, 0)`. Churned MRR excludes accounts that appear in the downgrade set.
- **Cohort retention checkpoints:** A cohort member is "active" at month M if `last_active > signup_date + M months`. Only cohorts where the checkpoint date falls on or before the analysis date (1 Jan 2025) are included in that checkpoint's average.
- **Industry/referral segmentation caveat:** Some segments (e.g. DevTools at Month 12) have small eligible cohort sizes. Treat segment-level percentages as directional rather than statistically precise.

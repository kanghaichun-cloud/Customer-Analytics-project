# Customer Analytics: Segmentation → Retention Diagnosis → A/B Test Validation

A three-notebook analytics project that starts with a business problem — *this retailer is losing customers after their first purchase; what should we do about it?* — and builds a complete, data-grounded answer.

The project uses the [UCI Online Retail Dataset](https://archive.ics.uci.edu/dataset/352/online+retail) (541k transactions, Dec 2010 – Dec 2011, UK-based online gift retailer).

---

## The Story in 60 Seconds

Most customer analytics projects stop at segmentation. This one asks **what to do with the segments** — and whether the recommendation actually works.

The analysis follows a single thread across three notebooks:

1. **Who are our customers?** → RFM clustering reveals 4 segments: a large mid-tier (C0), a lapsed group (C1), a handful of wholesale whales (C2), and a small but high-value loyal core (C3).

2. **Where do we lose them?** → Cohort retention shows 70–85% of customers never return after their first purchase. The purchase funnel confirms it: the 1→2 transition loses 34.4% of customers — nearly 10pp worse than any later step. Revenue is concentrated (26% of customers → 80% of revenue), so even small retention improvements at this step have outsized impact.

3. **What specifically should we do — and does it work?** → SQL gap analysis reveals that C0 and C1 have *identical* purchase rhythms (median gap ~43–48 days) but radically different recency (44d vs 248d). This means the same coupon window can't serve both. The A/B simulation validates segment-specific coupons (52-day for C0, 45-day for C1) with statistically significant lift in both cases.

**The key insight that connects everything:** C0 and C1 are not separated by how often they *used to* buy — they are separated by whether they are *still* buying. The intervention logic follows directly from this distinction.

---

## Project Structure

```
├── 01_customer_analytics.ipynb      # Segmentation, retention, revenue concentration
├── 02_sql_business_analysis.ipynb   # SQL deep-dive: funnel, timing, segment-level gaps
├── 03_ab_test_simulation.ipynb      # A/B test simulation + business impact estimation
├── rfm_clusters.csv                 # Linking key between notebooks (CustomerID → Cluster)
└── README.md
```

### How the notebooks connect

```
01 (Python)                    02 (SQL via DuckDB)              03 (Python + scipy)
─────────────                  ──────────────────               ───────────────────
RFM segmentation               Loads rfm_clusters.csv           Loads rfm_clusters.csv
  → 4 clusters (C0–C3)          → Purchase funnel analysis       → Empirical gap baselines
  → rfm_clusters.csv            → Inter-purchase gap by cluster  → Coupon window design
Cohort retention                → Segment-specific tactics       → A/B simulation
  → "when do we lose them?"                                      → t-test + 95% CI
Pareto analysis                                                  → Revenue impact
  → "who drives revenue?"
```

Each notebook is self-contained and runnable, but the analytical logic flows left to right. `rfm_clusters.csv` is the shared key.

---

## Notebook Details

### 01 — Customer Analytics: Segmentation, Retention & Revenue

**What it does:** Builds the analytical foundation — cleans the data, segments customers via KMeans on RFM features, analyses cohort retention, and quantifies revenue concentration.

**Key decisions and trade-offs:**
- **StandardScaler over log-transform:** Monetary and Frequency are right-skewed, so a log-transform before scaling is a defensible alternative. StandardScaler was chosen because the resulting clusters are interpretable and business-meaningful — but this is a modelling choice, not a settled answer.
- **K=4 over K=2:** Silhouette score peaks at K=2 (0.896), but two clusters can't distinguish whale, at-risk, regular, and loyal customers. K=4 balances statistical separation (silhouette 0.616) with business interpretability.
- **Sensitivity analysis on Pareto:** C2 (13 whale/B2B customers) distorts the revenue concentration figure. The Pareto analysis is run with and without C2 to confirm the conclusion holds for the retail base (26% → 31% — same story).

**Key outputs:**
- `rfm_clusters.csv` — the linking key for Notebooks 02 and 03
- Cohort retention heatmap — establishes the early drop-off pattern
- Pareto curve — quantifies revenue concentration

---

### 02 — SQL Business Analysis (DuckDB)

**What it does:** Translates the segmentation into actionable business intelligence using SQL. Four cases build on each other:

| Case | Question | Finding |
|------|----------|---------|
| 1 | How many customers ever return? | 64.3% — but this is a lifetime metric, not a timing signal |
| 2 | When do they return? | The 31–60 day window captures +14.9pp incremental return — the largest single jump |
| 3 | Where in the funnel do we lose them? | 1→2 purchase transition: 34.4% drop-off (10pp worse than any later step) |
| 4 | Does every segment need the same intervention? | No — C0 and C1 have identical gap distributions but radically different recency |

**The most important finding in this notebook:** C0 (median gap 43d, recency 44d) and C1 (median gap 48d, recency 248d) had virtually the same purchase rhythm when they were active. The difference is entirely in recency — C0 is mid-cycle, C1 has structurally disengaged. This means:
- C0 needs a **nudge at the right moment** (coupon arrives when the purchase decision becomes active)
- C1 needs a **reason to restart** (win-back with a deadline)

---

### 03 — A/B Test Simulation

**What it does:** Takes the segment-specific coupon windows from Notebook 02 and validates them through simulated A/B experiments.

| Cluster | Coupon Window | Control Rate | Treatment Rate | Lift | p-value | Significant? |
|---------|-------------|-------------|----------------|------|---------|-------------|
| C0 — Regular | 52-day | ~60% | ~69% | +8–9pp | <0.001 | ✅ |
| C1 — Lapsed | 45-day | ~15% | ~25% | +8–10pp | <0.001 | ✅ |

**Key design decisions:**
- **C1 control rate set at 15%, not 47.8%.** The raw gap analysis shows 47.8% return-within-45d — but this is computed only from C1 customers with 2+ purchases (n=504). With avg frequency of 1.6, most C1 customers have a single purchase and no gap history. 15% is a conservative estimate consistent with single-purchase win-back benchmarks.
- **Power analysis before simulation.** Both experiments are verified as adequately powered before running.
- **Margin sensitivity included.** Gross lift (~£204k) is adjusted for 10–20% coupon depth to give a realistic net revenue range.

**Limitations acknowledged:**
- This is a simulation, not a real experiment — the +10pp lift assumption is from industry benchmarks
- C1 gap estimates are directional (derived from the minority with 2+ purchases)
- Single 12-month observation window — some "lapsed" customers may be seasonal
- No campaign cost data in the dataset

---

## Tools Used

- **Python:** pandas, numpy, scikit-learn (KMeans, PCA, StandardScaler, silhouette_score), scipy (t-test), matplotlib, seaborn
- **SQL:** DuckDB (in-process analytical SQL engine)
- **Visualisation:** Tableau Public (dashboard, linked from notebooks)

---

## How to Run

```bash
# 1. Download the dataset
#    UCI Online Retail: https://archive.ics.uci.edu/dataset/352/online+retail
#    Save as online_retail.csv in the project root

# 2. Install dependencies
pip install pandas numpy scikit-learn scipy matplotlib seaborn duckdb

# 3. Run notebooks in order
#    01 → produces rfm_clusters.csv
#    02 → reads rfm_clusters.csv, produces segment analysis
#    03 → reads rfm_clusters.csv, runs A/B simulation
```

---

## What I Would Do Differently With More Data

- **Run a real A/B test** instead of simulating — the simulation provides power and effect size priors
- **Add product category** to the retention model — high-AOV categories may have different churn dynamics
- **Extend the observation window** to separate seasonal absence from structural churn
- **Add real campaign delivery cost** (email/print/design) to the margin sensitivity model — the current analysis accounts for coupon discount depth but not operational cost per contact
- **Test a longer coupon window for C0** (52–60 days) based on the segment-level gap analysis vs. the 30-day population-level baseline used in the original simulation

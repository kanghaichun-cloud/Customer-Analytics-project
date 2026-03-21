# Customer Analytics — UCI Online Retail (2010–2011)

End-to-end customer analysis combining Python-based segmentation with SQL-based behavioral analysis. Built on the UCI Online Retail dataset covering ~4,300 customers and 18,500 orders across December 2010 – December 2011.

---

## Project Structure

| File | Description |
|------|-------------|
| `customer_analytics.ipynb` | RFM segmentation (KMeans), cohort retention, revenue concentration |
| `sql_business_analysis.ipynb` | Retention funnel, purchase gap analysis, segment targeting (DuckDB) |
| `rfm_clusters.csv` | Cluster labels exported from Python notebook, consumed by SQL notebook |

> `online_retail.csv` is not included. Download from [UCI Repository](https://archive.ics.uci.edu/dataset/352/online+retail) and place in the project root before running.

---

## Key Findings

**Segmentation (Python)**
- KMeans (K=4) identifies four distinct tiers: ~13 whale customers (likely B2B), ~200 high-value loyals, ~3,000 regular buyers, and ~1,000 at-risk/lapsed users
- 26% of customers contribute 80% of revenue — concentrated but with a meaningful mid-tier that can be upgraded through frequency and basket-size strategies

**Retention Funnel (SQL)**
- 34.4% of customers never return after their first purchase — the 1→2 transition is the single largest drop-off in the lifecycle
- Once past the second purchase, retention stabilizes at ~70–75%
- Inter-purchase mean = 45.7 days; median = 28 days (right-skewed distribution)

**Recommendation**
- A 45-day post-first-purchase coupon targets the peak return window (day 31–60), converting passive re-engagers before the gap becomes churn
- Cluster 3 (AOV £1,080) is the highest-ROI target for this campaign
- Cluster 2 whale customers warrant dedicated account management — retention here is critical regardless of discount strategy

---

## How to Run

1. Download `online_retail.csv` from the UCI link above and place it in the project root
2. Run `customer_analytics.ipynb` — generates `rfm_clusters.csv`
3. Run `sql_business_analysis.ipynb` — reads `rfm_clusters.csv` for segment-level analysis

---

## Stack

Python · DuckDB · pandas · scikit-learn · Jupyter
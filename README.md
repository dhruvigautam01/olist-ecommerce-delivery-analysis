# Does Delivery Speed Affect Customer Satisfaction? — Olist E-Commerce Analysis

> **Status: Core Analysis Complete** — Python/EDA, SQL, and Power BI dashboard phases done. Final polish and write-up in progress.

## Overview

An end-to-end analysis of the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), covering 99,000+ orders across customers, products, sellers, payments, and reviews. This project investigates a core business question: **does how fast an order is delivered actually affect how customers rate their experience?**

The full pipeline: Python for data cleaning, feature engineering, and statistical testing → SQL for business-question-driven querying → Power BI for an executive dashboard → this README tying it all into a business narrative.

## Business Question

Late deliveries are assumed to hurt customer satisfaction — but is that assumption actually true in the data, or just intuition? This matters because it tells a business whether investing in faster logistics is likely to move customer satisfaction metrics, or whether the two are unrelated.

## Data

- **Source:** Olist Brazilian E-Commerce Public Dataset (Kaggle)
- **Scope:** 99,441 orders (2016–2018), filtered to 96,478 orders with status `delivered` for delivery-time analysis
- **Tables used:** `orders`, `order_reviews`, `customers`, `order_items`, `products`

## Data Cleaning Notes

- Orders were filtered to `order_status == 'delivered'` before any delivery-time analysis, since undelivered orders (canceled, shipped, unavailable, etc.) have no delivery date to measure.
- Date columns were converted from text to proper datetime type.
- The reviews table contained duplicate `order_id` entries; deduplicated to one review per order before merging, to avoid inflating order counts.
- A `delivery_days` metric was engineered as `order_delivered_customer_date − order_purchase_timestamp`.

## Analysis: Delivery Speed vs. Review Score

Orders were split into two groups at the median delivery time (10 days): **fast** (≤10 days) and **slow** (>10 days). An independent two-sample t-test compared average review scores between the groups.

| Group | Mean Review Score | Sample Size |
|-------|-------------------|-------------|
| Fast (≤10 days) | 4.38 | 51,839 |
| Slow (>10 days) | 3.94 | 49,565 |

**Result:** t = 55.4, p < 0.001

### Finding

Orders delivered in 10 days or fewer receive significantly higher review scores (4.38) than slower orders (3.94). With a p-value effectively at zero and sample sizes over 49,000 per group, this difference is not due to random chance — **delivery speed has a real, measurable relationship with customer satisfaction.**

### Business Implication

Investment in faster fulfillment and logistics is likely to have a direct, measurable payoff in customer satisfaction scores, not just anecdotal goodwill. This provides a data-backed case for prioritizing delivery-time improvements over other potential satisfaction levers, or for setting a delivery-time SLA target around the 10-day mark.

## What Drives Slow Delivery? — Geography

Average delivery time was calculated by customer state, filtering out states with very small order counts (e.g. under 100 orders) to avoid outliers distorting the picture — an early look showed a state with a 29-day average driven by only 41 orders, which wasn't reliable enough to draw conclusions from.

Among states with sufficient order volume, delivery time varies substantially:

| State | Avg. Delivery Days | Order Count |
|-------|--------------------|--------------|
| AM (Amazonas) | ~25 | 145 |
| AL (Alagoas) | ~24 | 397 |
| SP (São Paulo) | ~8.3 | 40,501 |

### Finding

States far from Olist's primary seller hub in São Paulo see meaningfully longer delivery times — Amazonas and Alagoas both average over three weeks, compared to São Paulo's 8.3-day average. This pattern holds even after filtering out low-volume states, and lines up with these states' physical distance from the main commercial and logistics hub.

### Business Implication

Slow delivery is largely a **geography and logistics problem**, not a fulfillment-process failure across the board. Efforts to improve delivery speed would likely be best targeted regionally (e.g. regional distribution hubs or carrier partnerships for the North/Northeast) rather than applied uniformly nationwide.

## What Drives Slow Delivery? — Product Category

Average delivery time was also calculated by product category, again filtering for categories with meaningful order volume.

| Category | Avg. Delivery Days | Order Count |
|----------|--------------------|--------------|
| moveis_escritorio (office furniture) | 20.4 | 1,668 |
| fashion_calcados (footwear) | 14.9 | 257 |
| Most other categories | ~10 (near median) | — |

### Finding

Product category has a secondary effect: office furniture nearly doubles the overall median delivery time, plausibly due to size and handling complexity. Most other categories cluster close to the overall median, suggesting category is a smaller factor than geography.

### Business Implication

Bulky/heavy categories like furniture may warrant separate logistics handling or clearer delivery-time expectations set at checkout, but category-driven delays are not the primary lever for improving delivery speed overall — geography remains the dominant factor.

## SQL Analysis: Revenue and Customer Retention

Queries were run using DuckDB against the pandas DataFrames, allowing standard SQL (joins, GROUP BY, HAVING) directly on the cleaned data.

### Revenue by State

```sql
SELECT c.customer_state, SUM(oi.price) AS total_revenue
FROM order_items oi
INNER JOIN orders o ON oi.order_id = o.order_id
INNER JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_state
ORDER BY total_revenue DESC
```

| State | Total Revenue |
|-------|---------------|
| SP (São Paulo) | ~R$5.2M |
| RJ (Rio de Janeiro) | ~R$1.82M |
| MG (Minas Gerais) | ~R$1.58M |

**Finding:** Revenue is heavily concentrated in São Paulo, generating nearly 3x the revenue of the next closest state. This mirrors the earlier delivery-speed finding — SP is both the fastest-shipping and highest-revenue state, consistent with it being Olist's commercial hub.

### Repeat Customer Rate

```sql
SELECT c.customer_unique_id, COUNT(o.order_id) AS order_count
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
GROUP BY c.customer_unique_id
HAVING COUNT(o.order_id) > 1
```

- **Repeat customers:** 2,997
- **Total unique customers:** 96,096
- **Repeat rate:** ~3.1%

**Finding:** Only about 3.1% of customers placed more than one order. This is low relative to typical e-commerce benchmarks (often 20-30%+), indicating the business is currently driven almost entirely by one-time purchases.

### Business Implication

Combined, these findings suggest two distinct opportunities: (1) revenue is concentrated in a single region, so growth efforts targeting other high-population states could diversify revenue away from SP dependency, and (2) the very low repeat rate points to a significant retention gap — even a modest improvement here would likely have an outsized impact on revenue, since acquiring one-time buyers is already working at scale.

## Power BI Dashboard

A dashboard was built in Power BI Desktop on top of the cleaned data (exported from the notebook), using a star schema with `delivered_orders` as the fact table and `customers`, `products` as dimension tables joined through `order_items`.

![Dashboard](<img width="749" height="422" alt="Screenshot 2026-09-20 124142" src="https://github.com/user-attachments/assets/64771901-1d90-4ec8-9e03-6005c2642bfb" />
)

**KPI Cards:**
- Average delivery time: 12.09 days
- Average review score: ~4.15
- Repeat customer rate: 3.1%
- Total revenue

**Visuals:**
- **Average delivery days by state** — filtered to states with 100+ orders to avoid small-sample distortion, confirming Amazonas and Alagoas as the genuinely slowest states
- **Revenue by state** — confirms São Paulo's dominant share of both revenue and delivery speed
- **Review score by delivery speed** — a `Delivery Speed` calculated column (Fast ≤10 days / Slow >10 days) visualizes the core t-test finding: fast deliveries average a noticeably higher review score than slow ones

All dashboard figures were cross-checked against the Python (pandas/scipy) and SQL (DuckDB) analysis earlier in this project, and match exactly — confirming the pipeline is consistent end-to-end from raw data through to the final visuals.

## Tools Used

- **Python** (pandas, scipy) — data cleaning, feature engineering, hypothesis testing
- **SQL** (via DuckDB) — business-question-driven querying: revenue by state, repeat customer rate
- **Power BI** — executive dashboard with DAX measures (repeat rate, order counts) and a calculated column (delivery speed grouping)

## Next Steps

- [x] Investigate *why* some orders are slow — by state and product category
- [ ] Investigate seller-level effects on delivery time
- [x] SQL phase: revenue by state, repeat customer rate
- [ ] SQL: late-delivery revenue impact, average order value
- [x] Power BI dashboard summarizing key findings
- [ ] Final write-up with recommendations

## How to Reproduce

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) or run this notebook directly on Kaggle (dataset pre-attached).
2. Run the notebook (`.ipynb`) — cells are organized by pipeline stage (load → clean → engineer features → analyze).
3. Requires: `pandas`, `scipy`, `duckdb`.
4. To reproduce the Power BI dashboard: export the cleaned tables from the notebook to CSV (`delivered_orders`, `customers`, `order_items`, `products`), then open `dashboard.pbix` in Power BI Desktop and refresh the data source paths.

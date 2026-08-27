# olist-ecommerce-delivery-analysis
# Does Delivery Speed Affect Customer Satisfaction? — Olist E-Commerce Analysis

> **Status: In Progress** — Python/EDA phase complete. SQL and Power BI dashboard phases coming next.

## Overview

An end-to-end analysis of the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), covering 99,000+ orders across customers, products, sellers, payments, and reviews. This project investigates a core business question: **does how fast an order is delivered actually affect how customers rate their experience?**

The full pipeline (planned): SQL for data cleaning and business queries → Python for statistical testing → Power BI for an executive dashboard → this README tying it all into a business narrative.

## Business Question

Late deliveries are assumed to hurt customer satisfaction — but is that assumption actually true in the data, or just intuition? This matters because it tells a business whether investing in faster logistics is likely to move customer satisfaction metrics, or whether the two are unrelated.

## Data

- **Source:** Olist Brazilian E-Commerce Public Dataset (Kaggle)
- **Scope:** 99,441 orders (2016–2018), filtered to 96,478 orders with status `delivered` for delivery-time analysis
- **Tables used:** `orders`, `order_reviews` (customers, order_items, payments, products, and sellers to be incorporated in later phases)

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

## Tools Used

- **Python** (pandas, scipy) — data cleaning, feature engineering, hypothesis testing
- **SQL** — *(planned)* business-question-driven querying across tables
- **Power BI** — *(planned)* executive dashboard with DAX measures

## Next Steps

- [ ] Investigate *why* some orders are slow — by state, seller, or product category
- [ ] SQL phase: revenue by category/state, repeat customer rate, late-delivery impact
- [ ] Power BI dashboard summarizing key findings
- [ ] Final write-up with recommendations

## How to Reproduce

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) or run this notebook directly on Kaggle (dataset pre-attached).
2. Run `notebook.ipynb` — cells are organized by pipeline stage (load → clean → engineer features → analyze).
3. Requires: `pandas`, `scipy`.

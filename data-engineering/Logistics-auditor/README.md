# The "Last Mile" Logistics Auditor
**Client:** Veridi Logistics | **Dataset:** Olist Brazilian E-Commerce | **Author:** Manyara Bonface Baraka

---

## A. Executive Summary

Analysis of 96,470 delivered orders reveals that 8.11% arrive later than promised, with the Nordeste region disproportionately affected at 15.19% late rate — nearly double every other region. Contrary to the assumption that remoteness drives delays, the Norte region (most geographically remote) recorded only 7.51% late rate, indicating a regional operational gap in the Northeast rather than a distance problem. Late deliveries have a severe impact on customer satisfaction — Super Late orders (>5 days late) average a review score of 1.78/5 compared to 4.29/5 for On Time orders, a 58% drop. Most critically, Super Late deliveries put an estimated R$499,145 in revenue at risk annually due to customer churn, with Rio de Janeiro (R$121,885) and São Paulo (R$113,623) representing the highest financial exposure — not because of worst late rates, but because of highest order volumes.

---

## B. Project Links

- **Notebook:** [Google Colab — Amaliteh_DE_Logistic.ipynb](https://colab.research.google.com/drive/1yD7HfnDkY-sQZNPSDaPZIt2oF0MdQvxD?usp=sharing)
- **Dashboard:** [Tableau Public — Veridi Logistics Delivery Performance Audit](https://public.tableau.com/app/profile/bonface.manyara/viz/VeridiLogisticsDeliveryPerformanceAudit_17806025621570/VeridiLogisticsDeliveryAudit)
- **Presentation:** [Google Slides — Delivery Performance Audit](https://docs.google.com/presentation/d/1MtIa_RTVGQI6ZvRR1SDGnB8sJxVN_BYeyXq-agQUaRg/edit?usp=sharing)

---

## C. Technical Explanation

### Data Sources
Six CSV files from the Olist Brazilian E-Commerce dataset (Kaggle):
- `olist_orders_dataset.csv` — central fact table (99,441 orders)
- `olist_order_reviews_dataset.csv` — customer sentiment (99,224 reviews)
- `olist_customers_dataset.csv` — geographic data
- `olist_order_items_dataset.csv` — bridge table linking orders to products
- `olist_products_dataset.csv` — product categories (Portuguese)
- `product_category_name_translation.csv` — English category mapping
- `olist_order_payments_dataset.csv` — payment values for Revenue at Risk analysis

### Data Cleaning
- **Master table join:** 6 CSV files joined into a single master dataset of 99,441 rows. Deduplication assertion confirmed row count integrity throughout — 99,441 in, 99,441 out.
- **Review deduplication:** 551 orders had multiple reviews. Resolved by keeping the most recent review per order, preserving the customer's final considered sentiment rather than averaging scores.
- **Undelivered orders excluded:** 2,963 orders with statuses other than "delivered" (canceled, unavailable, shipped, processing, invoiced, created, approved) were excluded from delay analysis. Only delivered orders have an actual delivery date to compare against estimates.
- **Missing delivery dates:** 8 orders marked "delivered" with no recorded delivery date were dropped — confirmed data entry errors in the source system. Impact: 0.008% of delivered orders.
- **Category threshold:** Product categories with fewer than 100 orders were excluded from category analysis to ensure statistical reliability of late rate percentages.
- **Date parsing:** All date columns converted from string to `datetime64` before calculations.

### Delay Classification
```
days_difference = order_estimated_delivery_date - order_delivered_customer_date

Positive value = delivered early or on time
Negative value = delivered late

On Time:    days_difference >= 0
Late:       days_difference between -5 and -1 (1-5 days late)
Super Late: days_difference < -5 (more than 5 days late)
```

### Regional Classification
Brazilian state regions were sourced directly from the GeoJSON properties (`regiao_id` field) used to build the choropleth map — no external geographic assumptions were made. São Paulo identified as distribution hub based on order volume data (40,494 orders = 42% of total).

### Candidate's Choice — Revenue at Risk Analysis

**Why this feature:** The CEO's question was about over-promising and under-delivering. Stories 1–4 proved *where* and *how badly* this happens. The Revenue at Risk analysis answers the follow-up question every CEO actually cares about: *"What is this costing us?"*

**Methodology:**
- Isolated Super Late orders (>5 days late) as the highest customer churn risk
- Churn probability (64.4%) derived directly from the data: Super Late average review score = 1.78/5 = 35.6% satisfaction rate, meaning 64.4% dissatisfaction rate used as churn proxy
- Revenue at Risk = `total_payment_value × churn_probability` per state
- Payment data sourced from `olist_order_payments_dataset.csv`, aggregated by `order_id` to handle multiple payment methods per order

**Key finding:** R$499,145 in revenue at risk annually. RJ and SP lead not because of worst late rates — they don't even appear in the top 10 worst states — but because their high order volumes amplify even moderate late rates into large financial exposure. This reveals two distinct problems requiring two different interventions: Nordeste needs operational logistics fixes, while RJ/SP need delivery promise recalibration.

---

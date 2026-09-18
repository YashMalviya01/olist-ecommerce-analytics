# 🛒 Olist E-Commerce Analytics

<p align="center">
  <img src="screenshots/project_banner.png" width="100%">
</p>

<p align="center">

![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Warehouse-blue) 
![AWS](https://img.shields.io/badge/AWS-S3-orange)
![Python](https://img.shields.io/badge/Python-Analytics-yellow)
![SQL](https://img.shields.io/badge/SQL-Analysis-green)
![Tableau](https://img.shields.io/badge/Tableau-Visualization-red)

</p>

---

## What this project is

An end-to-end analytics build on Olist's Brazilian e-commerce data (99,441 customers, 96,096 unique buyers) — from raw files in **S3**, through a **Snowflake** warehouse, to three **Tableau** dashboards a business team could actually use.

The short version of what I found:

- **Late delivery is the single biggest driver of bad reviews** — delivery delay correlates more strongly with low review scores than any other factor I tested.
- **São Paulo dominates** — highest revenue of any state, and by city too.
- **Revenue is concentrated in a small group of sellers**, clustered in just a few states — a supply-side concentration risk, not just a demand-side story.
- **Credit card is the default payment method** (76,795 of ~104K orders), and most customers choose fewer installments over more.

Everything below is how I got to those findings.

---

## Business questions I set out to answer

- Which states/cities drive the most revenue, and is that growing over time?
- Which product categories drive revenue vs. volume (they're not the same)?
- How much does delivery delay actually hurt customer satisfaction?
- Who are the highest-value customers, and what does repeat behavior look like?
- Is seller revenue broad-based or concentrated?

---

## Architecture

```text
                    ┌─────────────────┐
                    │    Amazon S3    │
                    │   Raw Datasets  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Snowflake     │
                    │ Staging Layer   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Data Cleaning   │
                    │ Transformation  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Data Mart Layer │
                    │ Star Schema     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Business Layer  │
                    │ SQL Analytics   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Tableau      │
                    │ Dashboards      │
                    └─────────────────┘
```

**Star schema:**

```text
                      DIM_CUSTOMERS
                              │
DIM_PRODUCTS ───── FACT_ORDERS ───── DIM_SELLERS
                              │
                         DIM_DATE

FACT_PAYMENTS
FACT_REVIEWS
```

**Stack:** AWS S3 (storage) · AWS Glue + Athena (validation, automation) · Snowflake (warehouse) · Python · SQL · Tableau

---

## Data quality, before the analysis

Before trusting any of the numbers above, I profiled and validated the raw data:

| Metric | Result |
|---|---|
| Total customers | 99,441 |
| Unique customers | 96,096 |
| Missing product category/description fields | 610 rows |

I applied a 9-category data quality framework across all 10 staging tables in Athena (90 checks total) — schema integrity, completeness, content validity, and ETL load accuracy — before promoting anything to Snowflake.

**Payment method breakdown (from profiling):**

| Method | Orders |
|---|---|
| Credit Card | 76,795 |
| Boleto | 19,784 |
| Voucher | 5,775 |
| Debit Card | 1,529 |

**Review score distribution:**

| Score | Count |
|---|---|
| 5 | 57,328 |
| 4 | 19,142 |
| 3 | 8,179 |
| 2 | 3,151 |
| 1 | 11,424 |

Cleaning steps: null handling, duplicate detection, type conversion, date standardization, category normalization, and review integrity checks.

---

## A few of the SQL metrics behind the dashboards

**Delivery days** (feeds the delay → review-score finding):
```sql
SELECT
    ORDER_ID,
    DATEDIFF(DAY, ORDER_PURCHASE_TIMESTAMP, ORDER_DELIVERED_CUSTOMER_DATE) AS DELIVERY_DAYS
FROM FACT_ORDERS;
```

**Customer lifetime value:**
```sql
SELECT
    CUSTOMER_UNIQUE_ID,
    COUNT(DISTINCT ORDER_ID) AS TOTAL_ORDERS,
    SUM(REVENUE) AS CUSTOMER_LIFETIME_VALUE
FROM FACT_ORDERS
GROUP BY CUSTOMER_UNIQUE_ID;
```

Full SQL (staging → cleaning → modeling → business analysis) is in [`/sql`](./sql).

---

## Dashboards

**1. Executive Performance** — revenue, orders, AOV, review score, and delivery time at a glance, with monthly revenue trend and state/city breakdowns.

<img width="3198" height="1798" alt="Dashboard 1" src="https://github.com/user-attachments/assets/db567647-d38c-4929-95e3-75a02b85ca7b" />

**2. Products & Operations** — review distribution, the delivery-delay-to-review-score relationship, category revenue vs. volume, top sellers.

<img width="3198" height="1798" alt="Dashboard 2" src="https://github.com/user-attachments/assets/b9fa6f52-4f9b-4318-b256-c3a3713819d2" />

**3. Customer & Geographic Insights** — CLV, repeat customer behavior, satisfaction by region, top revenue/seller states.

<img width="2730" height="1534" alt="Dashboard 3" src="https://github.com/user-attachments/assets/877fbf11-c80d-4f2a-89ae-be404029bddf" />

---

## Cloud layer (S3 + Snowflake)

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/8fa61ae1-9cea-496c-8dd6-78dfa0e15c9a" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/9f05cebf-9794-4abf-a6a2-a27e275df5da" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/cf35ba21-1308-4214-b4e3-3fe6c081aad2" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/d383f889-684b-4c5d-a23a-eeffdcb03241" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/198a0584-1bce-465a-9d28-fc38064ee296" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/33224184-78f6-410e-bf1c-2ad3c46efe69" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/f826f730-2139-4546-aef1-cd021fba3e7b" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/f0cac66d-566e-445b-b414-98829090fc3d" />
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/f1fe83f9-7b53-4e14-922f-46f1cb5e4f71" />

---

## Repo structure

```text
Olist-Ecommerce-Analytics/
├── data/{raw, cleaned, business_outputs}/
├── sql/01_staging.sql … 06_business_analysis.sql
├── tableau/Dashboard_1-3.twbx
├── screenshots/
└── README.md
```

---

## What I'd add next

Real-time pipeline refresh, churn prediction, and a product recommendation engine are the natural next steps — the CLV and repeat-purchase groundwork here would feed directly into either.

---

## Author

**Yash Malviya** — Data Analyst | SQL · Python · Snowflake · AWS · Tableau
[LinkedIn](https://www.linkedin.com/in/yash-malviya-03433b258/) · [Tableau Public](https://public.tableau.com/app/profile/yash.malviya6387/vizzes) · [GitHub](https://github.com/YashMalviya01)

⭐ If this was useful, a star helps others find it.

# Customer Retention Strategy — D2C Fashion Brand
**Consulting & Analytics Club, IIT Guwahati**

Analysis of 3,900 customers to find out: is this brand building real loyalty, or just buying attention with discounts?

## The Answer
Mostly discounts. **43% of customers only buy on promo**, and just **15.4% (600 customers)** are genuinely loyal. The subscription program (1,053 members) isn't helping — it shows **zero loyalty lift**. The strongest loyalty signals are being **female (1.65x lift)** and **not subscribed (1.37x lift)**.

## Key Numbers

| Metric | Value |
|---|---|
| Customers analyzed | 3,900 |
| Genuinely loyal | 600 (15.4%) |
| Annual margin lost to discounts | $19,882 |
| Subscription loyalty lift | 0.00 |
| Projected 6-month recovery | $9,338 |

## What's in Here

```
Deliverable_1_Python_Notebook.ipynb      # Feature engineering
Deliverable_1_engineered_dataset.csv     # Cleaned dataset (31 columns)
Deliverable_2_SQL_Queries.sql            # 8 PostgreSQL segmentation queries
Deliverable_3_Power_BI_Template.pbix     # 4-panel dashboard
Deliverable_4_Retention_Playbook.docx    # 4-phase action plan
Deliverable_5_Executive_Summary.pdf      # Board-ready summary
```

## How to Run

1. **Python** — open the notebook in Colab or Jupyter, run it on the raw dataset to get `engineered_dataset.csv`.
2. **SQL** — load `engineered_dataset.csv` into PostgreSQL, run the 8 queries in `Deliverable_2_SQL_Queries.sql`.
3. **Power BI** — open the `.pbix` file, re-point the data source to your local CSV if visuals are blank.
4. **Read the playbook** — `Deliverable_4_Retention_Playbook.docx` has the 4-phase plan; `Deliverable_5_Executive_Summary.pdf` is the short version.

## The Playbook (Promo Sunset)

1. Stop discounting Promo Hunters → **+$3,738**
2. Slowly cut discounts for At-Risk customers → **+$5,600**
3. Protect Organic Loyalists and High-Value Retainers → **$59,376 safeguarded**
4. Fix the subscription program before spending more on it

## Loyalty Definitions

- **Loyal_A (behavioral):** no discounts used + 25+ past purchases + above-average review rating
- **Loyal_B (revenue):** top 40% spend + top 40% frequency + no discounts

Loyal_A is the main metric. Loyal_B exists as a stress test — high spenders aren't always behaviorally loyal.

## Segments

| Segment | Meaning |
|---|---|
| Organic Loyalist | Loyal, no promo use |
| High-Value Retainer | High value, some promo use |
| At-Risk | Mid value, promo-dependent |
| Promo Hunter | Low value, only buys on discount |
| Dormant | Low frequency and tenure |

## Tools Used
Python (Pandas, NumPy) · PostgreSQL · Power BI · Google Colab

---
*Synthetic dataset. Independent project — Consulting & Analytics Club, IIT Guwahati.*
<img width="1895" height="948" alt="image" src="https://github.com/user-attachments/assets/cd908754-6cca-4a63-b480-f4f585191ad0" />

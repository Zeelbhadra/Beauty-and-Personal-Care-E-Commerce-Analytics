# Beauty-and-Personal-Care-E-Commerce-Analytics

> **Why is revenue flat despite 18% user growth?**  
> An end-to-end data analysis project spanning Excel, PostgreSQL, and Power BI.

---

## Project Overview

A beauty & personal care e-commerce platform was growing its monthly active user base by 18% YoY, but net revenue remained flat. Marketing spend was rising, discounts were increasing, and return rates were climbing. This project investigates the root causes across 50,000 orders and 5 data tables spanning 18 months.

**Tools used:** Excel · PostgreSQL · Power BI · Python (data generation)  
**Dataset:** 5 tables · 148,000+ rows · 18 months of synthetic e-commerce data  
**Output:** Executive memo + Power BI dashboard + 8 advanced SQL queries

---

## Repository Structure

```
Beauty and Personal Care E-commerce Analytcis/
├── Beauty & Personal Care E-Commerce_dashboard.pbix       # Power BI dashboard file
│
├── Executive_Memo_Revenue_Leakage.pdf
│
├── query_category_returns.csv
│
├── query_city_tier.csv
│
├── query_discount_buckets.csv
│
├── query_ltv_cac.csv
│
├── query_monthly_trend.csv
│
└── README.md

---

## Business Questions Answered

| # | Question | Tool |
|---|----------|------|
| Q2 | Are expensive acquisition channels justified by LTV? | SQL + Power BI |
| Q3 | Which city tiers show highest return rates? | SQL + Power BI |
| Q4 | Do high discounts drive disproportionate returns? | SQL + Power BI |
| Q6 | What is actual net revenue vs GMV after discounts and refunds? | Excel + SQL |
| Q7 | Which return reasons are fixable vs structural? | SQL |
| Q8 | Do city tiers correlate with discount levels and returns? | SQL + Power BI |
| Q9 | Which product categories have the highest return rates? | SQL + Power BI |
| Q11 | Which acquisition channel has the best LTV:CAC ratio? | SQL + Power BI |
| Q12 | Does festive marketing spend generate proportional revenue? | SQL + Power BI |
| Q13 | Are we acquiring cheaper customers or are existing ones buying less? | SQL |

---

## Key Findings

**Finding 1 — Discounting destroys margin faster than it drives growth**  
Revenue leakage from discounts alone = Rs. 1.32 crore (18.96% of GMV). Return rate jumps from 5.86% on low-discount orders to 32.05% on very high discount orders — a 5.5x gap.

**Finding 2 — Medium discount is the silent killer**  
Not the worst margin bucket, but the highest volume: 20,544 orders with Rs. 29.6L in refunds. Fixing medium-discount return behavior has higher total impact than addressing extreme discounting.

**Finding 3 — 55% of customers acquired at a net loss**  
Influencer (LTV:CAC 0.52x) and Paid Social (0.58x) together bring in 55% of customers but both are Loss Making. Blended platform LTV:CAC = 0.94x. No channel currently hits the Healthy 3x threshold.

**Finding 4 — Returns are fixable**  
Top reasons: Did Not Like It (1,244), Quality Not As Expected (1,124), Changed Mind (933). Uniform return rates across all categories (13.7–14.4%) confirm this is a platform-wide behavioral and operational problem, not a product-specific one.

---

## Dashboard Preview

The Power BI dashboard includes:
- 4 KPI cards: Total GMV · Net Revenue After Refunds · LTV:CAC Ratio · Overall Return Rate
- Revenue leakage line chart (GMV vs Net Revenue vs Net After Refunds — 18 months)
- Effective margin % by discount bucket (color coded green → red)
- LTV:CAC ratio by acquisition channel (with 3x healthy threshold reference line)
- Return rate % by product category
- City tier — order value vs return rate
- City tier slicer for interactive filtering

---

## SQL Highlights

This project uses advanced PostgreSQL including:

```sql
-- CTE example: LTV vs CAC by acquisition channel
WITH customer_ltv AS (
    SELECT customer_id,
           COUNT(order_id)      AS total_orders,
           SUM(net_revenue)     AS total_net_revenue
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY customer_id
),
channel_ltv AS (
    SELECT c.acquisition_channel,
           COUNT(cl.customer_id)          AS total_customers,
           ROUND(AVG(cl.total_net_revenue), 2) AS avg_ltv
    FROM customer_ltv cl
    JOIN customers c ON cl.customer_id = c.customer_id
    GROUP BY c.acquisition_channel
),
channel_cac AS (
    SELECT channel AS acquisition_channel,
           ROUND(AVG(cac), 2) AS avg_cac
    FROM marketing_spends
    GROUP BY channel
)
SELECT cl.acquisition_channel,
       cl.avg_ltv,
       cc.avg_cac,
       ROUND(cl.avg_ltv / NULLIF(cc.avg_cac, 0), 2) AS ltv_to_cac_ratio
FROM channel_ltv cl
JOIN channel_cac cc ON cl.acquisition_channel = cc.acquisition_channel
ORDER BY ltv_to_cac_ratio DESC;
```

**SQL concepts demonstrated:** Star schema design · CTEs · LEFT JOIN · CASE bucketing · COALESCE / NULLIF · COUNT DISTINCT · Window aggregation · TO_CHAR date formatting · Subquery joins

---

## Excel Skills Demonstrated

- EDA across 5 tables (148,000+ rows)
- AVERAGEIF to quantify revenue leakage by discount tier
- Pivot tables for CAC trend analysis by channel and month
- Basket size calculation and industry benchmark comparison
- Data quality flag identification (timestamp cleanup, cancelled order revenue)

---

## Power BI Skills Demonstrated

- Star schema data model across 5 CSV sources
- DAX measures: Total GMV, Net Revenue After Refunds, Overall Return Rate %, Blended LTV:CAC
- Color-coded KPI cards with business-meaning backgrounds
- Interactive city tier slicer
- Reference line at 3x LTV:CAC healthy threshold
- Custom color coding per bar (verdict-based: green / amber / red)

---

## How To Reproduce

**PostgreSQL setup:**
```bash
# 1. Create database
createdb nykaa_analysis

# 2. Run setup script
psql -d nykaa_analysis -f sql/01_nykaa_db_setup.sql

# 3. Verify row counts
psql -d nykaa_analysis -c "SELECT 'customers', COUNT(*) FROM customers UNION ALL SELECT 'orders', COUNT(*) FROM orders;"
```

**Power BI:**
1. Open Power BI Desktop
2. Get Data → Text/CSV → load all 5 CSV files from `data/` folder
3. Open `dashboard/Nykaa_Revenue_Leakage.pbix`

---

## Author

Data Science undergraduate with hands-on experience in SQL, Power BI, and Python.  
Focused on analytics problems where the answer changes how a team operates.

---

*Dataset is synthetic and generated for analytical purposes. All business insights are derived from simulated data modeled on real e-commerce patterns.*

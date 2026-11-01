# Marts & Fact Modeling

The Marts layer houses business-facing models, including Fact tables and Dimensions designed for analytics reporting.

---

## 🏛️ Models Overview
- **`fct_orders.sql`**: Captures order-level summaries, pricing metrics, and discount totals.
- **`dim_customers.sql`**: Provides customer demographic and segmentation attributes.
- **`monthly_sales_summary.sql`**: Aggregate reporting mart for executive dashboards.

---

## 💾 Materialization Strategy
Unlike staging models (views), marts are materialized as **`table`** or **`incremental`** to ensure fast query response times for BI tools.

# 🛒 Olist E-Commerce SQL Analytics Project

> A SQL-based recreation of the Olist Power BI dashboard, covering revenue, orders, customers, products, and seller performance insights.

---

## 📊 Dashboard Overview

| Metric | Value |
|---|---|
| Total Revenue | 16.01M |
| Total Orders | 113K |
| Total Customers | 99K |
| Total Products | 33K |
| Average Review Score | 4.09 |

---

## 🗃️ Database Schema (Olist Dataset)

```sql
-- Core tables used across queries
-- olist_orders_dataset
-- olist_order_items_dataset
-- olist_order_payments_dataset
-- olist_order_reviews_dataset
-- olist_customers_dataset
-- olist_products_dataset
-- olist_sellers_dataset
-- olist_geolocation_dataset
-- product_category_name_translation
```

---

## 📌 KPI Queries

### 1. 💰 Total Revenue

```sql
SELECT 
    ROUND(SUM(payment_value), 2) AS total_revenue
FROM olist_order_payments_dataset op
JOIN olist_orders_dataset o 
    ON op.order_id = o.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable');
```

---

### 2. 📦 Total Orders

```sql
SELECT 
    COUNT(DISTINCT order_id) AS total_orders
FROM olist_orders_dataset
WHERE order_status NOT IN ('canceled', 'unavailable');
```

---

### 3. 👥 Total Unique Customers

```sql
SELECT 
    COUNT(DISTINCT customer_unique_id) AS total_customers
FROM olist_customers_dataset;
```

---

### 4. 🛍️ Total Products

```sql
SELECT 
    COUNT(DISTINCT product_id) AS total_products
FROM olist_products_dataset;
```

---

### 5. ⭐ Average Review Score

```sql
SELECT 
    ROUND(AVG(review_score), 2) AS avg_review_score
FROM olist_order_reviews_dataset;
```

---

## 📈 Orders by Category

### 6. Top Categories by Number of Orders

```sql
SELECT 
    t.product_category_name_english AS category,
    COUNT(DISTINCT oi.order_id) AS total_orders
FROM olist_order_items_dataset oi
JOIN olist_products_dataset p 
    ON oi.product_id = p.product_id
JOIN product_category_name_translation t 
    ON p.product_category_name = t.product_category_name
JOIN olist_orders_dataset o 
    ON oi.order_id = o.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY t.product_category_name_english
ORDER BY total_orders DESC
LIMIT 10;
```

---

### 7. Revenue by Category

```sql
SELECT 
    t.product_category_name_english AS category,
    ROUND(SUM(op.payment_value), 2) AS total_revenue
FROM olist_order_items_dataset oi
JOIN olist_products_dataset p 
    ON oi.product_id = p.product_id
JOIN product_category_name_translation t 
    ON p.product_category_name = t.product_category_name
JOIN olist_orders_dataset o 
    ON oi.order_id = o.order_id
JOIN olist_order_payments_dataset op 
    ON oi.order_id = op.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY t.product_category_name_english
ORDER BY total_revenue DESC
LIMIT 10;
```

---

## 📅 Orders Trend Over Time

### 8. Monthly Orders Trend

```sql
SELECT 
    DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS order_month,
    COUNT(DISTINCT order_id) AS total_orders
FROM olist_orders_dataset
WHERE order_status NOT IN ('canceled', 'unavailable')
GROUP BY order_month
ORDER BY order_month ASC;
```

---

### 9. Daily Orders Trend (for time-series chart)

```sql
SELECT 
    DATE(order_purchase_timestamp) AS order_date,
    COUNT(DISTINCT order_id) AS total_orders
FROM olist_orders_dataset
WHERE order_status NOT IN ('canceled', 'unavailable')
GROUP BY order_date
ORDER BY order_date ASC;
```

---

## 🕐 Pending Orders by Status

### 10. Orders Count by Order Status

```sql
SELECT 
    order_status,
    COUNT(DISTINCT order_id) AS total_orders
FROM olist_orders_dataset
GROUP BY order_status
ORDER BY total_orders DESC;
```

---

### 11. Pending Orders Breakdown (Shipped, Canceled, Processing, Invoiced, Approved)

```sql
SELECT 
    order_status,
    COUNT(*) AS order_count
FROM olist_orders_dataset
WHERE order_status IN ('shipped', 'canceled', 'processing', 'invoiced', 'approved', 'unavailable')
GROUP BY order_status
ORDER BY order_count DESC;
```

---

## 🗺️ Orders by State (Geolocation)

### 12. Orders by Seller State

```sql
SELECT 
    s.seller_state,
    COUNT(DISTINCT oi.order_id) AS total_orders
FROM olist_order_items_dataset oi
JOIN olist_sellers_dataset s 
    ON oi.seller_id = s.seller_id
JOIN olist_orders_dataset o 
    ON oi.order_id = o.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY s.seller_state
ORDER BY total_orders DESC;
```

---

### 13. Orders by Customer State

```sql
SELECT 
    c.customer_state,
    COUNT(DISTINCT o.order_id) AS total_orders
FROM olist_orders_dataset o
JOIN olist_customers_dataset c 
    ON o.customer_id = c.customer_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY c.customer_state
ORDER BY total_orders DESC;
```

---

### 14. Revenue by Customer State

```sql
SELECT 
    c.customer_state,
    ROUND(SUM(op.payment_value), 2) AS total_revenue
FROM olist_orders_dataset o
JOIN olist_customers_dataset c 
    ON o.customer_id = c.customer_id
JOIN olist_order_payments_dataset op 
    ON o.order_id = op.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY c.customer_state
ORDER BY total_revenue DESC;
```

---

## 🔍 Additional Business Insights

### 15. Average Delivery Time per State

```sql
SELECT 
    c.customer_state,
    ROUND(AVG(DATEDIFF(o.order_delivered_customer_date, o.order_purchase_timestamp)), 1) AS avg_delivery_days
FROM olist_orders_dataset o
JOIN olist_customers_dataset c 
    ON o.customer_id = c.customer_id
WHERE o.order_delivered_customer_date IS NOT NULL
GROUP BY c.customer_state
ORDER BY avg_delivery_days ASC;
```

---

### 16. Top 10 Sellers by Revenue

```sql
SELECT 
    oi.seller_id,
    s.seller_city,
    s.seller_state,
    ROUND(SUM(op.payment_value), 2) AS total_revenue,
    COUNT(DISTINCT oi.order_id) AS total_orders
FROM olist_order_items_dataset oi
JOIN olist_sellers_dataset s 
    ON oi.seller_id = s.seller_id
JOIN olist_orders_dataset o 
    ON oi.order_id = o.order_id
JOIN olist_order_payments_dataset op 
    ON oi.order_id = op.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY oi.seller_id, s.seller_city, s.seller_state
ORDER BY total_revenue DESC
LIMIT 10;
```

---

### 17. Customer Retention — Repeat Buyers

```sql
SELECT 
    order_count_group,
    COUNT(*) AS customer_count
FROM (
    SELECT 
        c.customer_unique_id,
        COUNT(DISTINCT o.order_id) AS num_orders,
        CASE 
            WHEN COUNT(DISTINCT o.order_id) = 1 THEN 'One-time Buyer'
            WHEN COUNT(DISTINCT o.order_id) BETWEEN 2 AND 3 THEN 'Repeat Buyer (2-3)'
            ELSE 'Loyal Buyer (4+)'
        END AS order_count_group
    FROM olist_orders_dataset o
    JOIN olist_customers_dataset c 
        ON o.customer_id = c.customer_id
    WHERE o.order_status NOT IN ('canceled', 'unavailable')
    GROUP BY c.customer_unique_id
) customer_orders
GROUP BY order_count_group
ORDER BY customer_count DESC;
```

---

### 18. Payment Method Distribution

```sql
SELECT 
    payment_type,
    COUNT(*) AS total_transactions,
    ROUND(SUM(payment_value), 2) AS total_amount,
    ROUND(AVG(payment_value), 2) AS avg_payment
FROM olist_order_payments_dataset
GROUP BY payment_type
ORDER BY total_transactions DESC;
```

---

### 19. Average Order Value (AOV) by Month

```sql
SELECT 
    DATE_FORMAT(o.order_purchase_timestamp, '%Y-%m') AS order_month,
    ROUND(SUM(op.payment_value) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM olist_orders_dataset o
JOIN olist_order_payments_dataset op 
    ON o.order_id = op.order_id
WHERE o.order_status NOT IN ('canceled', 'unavailable')
GROUP BY order_month
ORDER BY order_month;
```

---

### 20. Review Score Distribution

```sql
SELECT 
    review_score,
    COUNT(*) AS total_reviews,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM olist_order_reviews_dataset
GROUP BY review_score
ORDER BY review_score DESC;
```

---

## 🏗️ How to Use This Project

1. **Download** the [Olist dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
2. **Import** all CSV files into your SQL database (MySQL / PostgreSQL / SQLite)
3. **Run** each query in sequence or use them to power your own dashboard
4. **Extend** the queries to add date filters, drill-downs, or export to BI tools

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| MySQL / PostgreSQL | Query execution |
| Olist Dataset (Kaggle) | Source data |
| Power BI / Tableau *(optional)* | Visualization layer |
| Git & GitHub | Version control |

---

## 📁 Project Structure

```
olist-sql-project/
│
├── README.md                  ← You are here
├── queries/
│   ├── 01_kpis.sql
│   ├── 02_orders_by_category.sql
│   ├── 03_orders_trend.sql
│   ├── 04_pending_orders.sql
│   ├── 05_orders_by_state.sql
│   └── 06_advanced_insights.sql
└── data/
    └── (place Kaggle CSV files here)
```

---

## 📜 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙌 Acknowledgements

- Dataset by [Olist](https://olist.com/) on [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Dashboard design inspired by Power BI Olist project

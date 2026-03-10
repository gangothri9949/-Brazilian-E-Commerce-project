1️⃣ Create Database
CREATE DATABASE olist_db;
USE olist_db;
2️⃣ Create Tables

Example structure for the main tables.

Orders Table
CREATE TABLE orders (
    order_id VARCHAR(50) PRIMARY KEY,
    customer_id VARCHAR(50),
    order_status VARCHAR(20),
    order_purchase_timestamp DATETIME,
    order_approved_at DATETIME,
    order_delivered_carrier_date DATETIME,
    order_delivered_customer_date DATETIME,
    order_estimated_delivery_date DATETIME
);
Customers Table
CREATE TABLE customers (
    customer_id VARCHAR(50) PRIMARY KEY,
    customer_unique_id VARCHAR(50),
    customer_city VARCHAR(50),
    customer_state VARCHAR(5)
);
Order Items Table
CREATE TABLE order_items (
    order_id VARCHAR(50),
    order_item_id INT,
    product_id VARCHAR(50),
    seller_id VARCHAR(50),
    price DECIMAL(10,2),
    freight_value DECIMAL(10,2)
);
Products Table
CREATE TABLE products (
    product_id VARCHAR(50) PRIMARY KEY,
    product_category_name VARCHAR(100)
);
Reviews Table
CREATE TABLE reviews (
    review_id VARCHAR(50),
    order_id VARCHAR(50),
    review_score INT
);
3️⃣ Data Cleaning
Remove Null Category Names
UPDATE products
SET product_category_name = 'Unknown'
WHERE product_category_name IS NULL;
Check Missing Values
SELECT *
FROM orders
WHERE order_purchase_timestamp IS NULL;
4️⃣ Create Revenue Calculation
SELECT 
    SUM(price + freight_value) AS total_revenue
FROM order_items;
5️⃣ Total Orders
SELECT COUNT(DISTINCT order_id) AS total_orders
FROM orders;
6️⃣ Total Customers
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM customers;
7️⃣ Total Products
SELECT COUNT(DISTINCT product_id) AS total_products
FROM products;
8️⃣ Average Review Score
SELECT 
    ROUND(AVG(review_score),2) AS avg_review
FROM reviews;
9️⃣ Orders by Category
SELECT 
    p.product_category_name,
    COUNT(oi.order_id) AS total_orders
FROM order_items oi
JOIN products p
ON oi.product_id = p.product_id
GROUP BY p.product_category_name
ORDER BY total_orders DESC;
🔟 Orders Trend Over Time
SELECT 
    DATE(order_purchase_timestamp) AS order_date,
    COUNT(order_id) AS total_orders
FROM orders
GROUP BY order_date
ORDER BY order_date;
1️⃣1️⃣ Pending Orders Analysis
SELECT 
    order_status,
    COUNT(order_id) AS total_orders
FROM orders
GROUP BY order_status
ORDER BY total_orders DESC;
1️⃣2️⃣ Orders by State
SELECT 
    c.customer_state,
    COUNT(o.order_id) AS total_orders
FROM orders o
JOIN customers c
ON o.customer_id = c.customer_id
GROUP BY c.customer_state
ORDER BY total_orders DESC;
1️⃣3️⃣ Create a View for Dashboard

This makes it easier to connect Power BI.

CREATE VIEW sales_summary AS
SELECT 
    o.order_id,
    o.order_status,
    o.order_purchase_timestamp,
    c.customer_state,
    p.product_category_name,
    oi.price,
    oi.freight_value
FROM orders o
JOIN customers c 
ON o.customer_id = c.customer_id
JOIN order_items oi 
ON o.order_id = oi.order_id
JOIN products p 
ON oi.product_id = p.product_id;
1️⃣4️⃣ Query for Power BI Dashboard
SELECT 
    product_category_name,
    COUNT(order_id) AS total_orders
FROM sales_summary
GROUP BY product_category_name
ORDER BY total_orders DESC;

1️⃣ Monthly Revenue Trend

Analyzing revenue growth over time.

SELECT 
    DATE_FORMAT(o.order_purchase_timestamp, '%Y-%m') AS order_month,
    ROUND(SUM(oi.price + oi.freight_value),2) AS total_revenue
FROM orders o
JOIN order_items oi
ON o.order_id = oi.order_id
GROUP BY order_month
ORDER BY order_month;
2️⃣ Top 10 Product Categories by Revenue
SELECT 
    p.product_category_name,
    ROUND(SUM(oi.price),2) AS revenue
FROM order_items oi
JOIN products p
ON oi.product_id = p.product_id
GROUP BY p.product_category_name
ORDER BY revenue DESC
LIMIT 10;
3️⃣ Top 10 Customers by Total Spending
SELECT 
    c.customer_unique_id,
    ROUND(SUM(oi.price + oi.freight_value),2) AS total_spent
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
JOIN order_items oi
ON o.order_id = oi.order_id
GROUP BY c.customer_unique_id
ORDER BY total_spent DESC
LIMIT 10;
4️⃣ Customer Order Frequency

Understanding how often customers place orders.

SELECT 
    c.customer_unique_id,
    COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_unique_id
ORDER BY total_orders DESC;
5️⃣ Revenue by State
SELECT 
    c.customer_state,
    ROUND(SUM(oi.price + oi.freight_value),2) AS total_revenue
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
JOIN order_items oi
ON o.order_id = oi.order_id
GROUP BY c.customer_state
ORDER BY total_revenue DESC;
6️⃣ Top 5 States Using Window Functions

Example of ROW_NUMBER().

SELECT *
FROM (
    SELECT 
        c.customer_state,
        ROUND(SUM(oi.price + oi.freight_value),2) AS revenue,
        ROW_NUMBER() OVER (ORDER BY SUM(oi.price + oi.freight_value) DESC) AS rank_position
    FROM customers c
    JOIN orders o
    ON c.customer_id = o.customer_id
    JOIN order_items oi
    ON o.order_id = oi.order_id
    GROUP BY c.customer_state
) ranked_states
WHERE rank_position <= 5;
7️⃣ Average Delivery Time Analysis
SELECT 
    AVG(DATEDIFF(order_delivered_customer_date, order_purchase_timestamp)) 
    AS avg_delivery_days
FROM orders
WHERE order_delivered_customer_date IS NOT NULL;
8️⃣ Order Status Distribution
SELECT 
    order_status,
    COUNT(*) AS total_orders
FROM orders
GROUP BY order_status
ORDER BY total_orders DESC;
9️⃣ Customer Retention Analysis

Customers with more than one order.

SELECT 
    customer_unique_id,
    COUNT(o.order_id) AS total_orders
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY customer_unique_id
HAVING COUNT(o.order_id) > 1
ORDER BY total_orders DESC;
🔟 Running Revenue Total (Window Function)
SELECT 
    DATE(o.order_purchase_timestamp) AS order_date,
    SUM(oi.price + oi.freight_value) AS daily_revenue,
    SUM(SUM(oi.price + oi.freight_value)) 
        OVER (ORDER BY DATE(o.order_purchase_timestamp)) 
        AS running_revenue
FROM orders o
JOIN order_items oi
ON o.order_id = oi.order_id
GROUP BY order_date;

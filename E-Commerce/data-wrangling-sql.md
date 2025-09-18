
# 🛠️ Data Wrangling in PostgreSQL — E-commerce Dataset  

This document presents the data wrangling and cleaning steps performed on the **E-commerce database** stored in PostgreSQL.  
The database consists of multiple related tables: `customers`, `orders`, `order_items`, `products`, `payments`, `reviews`, `sellers`, and `geolocation`.

---

## 1. Checking for Duplicates

### Customers
```sql
SELECT customer_id, COUNT(*)
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```
➡️ **Finding:** `customer_id` is unique.

### Orders
```sql
SELECT order_id, COUNT(*)
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```
➡️ **Finding:** No duplicate `order_id`.

### Products
```sql
SELECT product_id, COUNT(*)
FROM products
GROUP BY product_id
HAVING COUNT(*) > 1;
```
➡️ **Finding:** No duplicate `product_id`.

---

## 2. Handling Missing Values

### Orders not delivered
```sql
SELECT *
FROM orders
WHERE order_delivered_customer_date IS NULL;
```
➡️ 2900 of missing values, some orders were never delivered.

### Products with missing dimensions
```sql
SELECT *
FROM products
WHERE product_weight_g IS NULL
   OR product_length_cm IS NULL
   OR product_height_cm IS NULL
   OR product_width_cm IS NULL;
```
➡️ 2 missing or invalid product attributes detected.

---

## 3. Validating Foreign Keys

### Orders without matching customers
```sql
SELECT o.order_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```
➡️ There are no orders without customers.

### Order items without products
```sql
SELECT oi.order_id, oi.product_id
FROM order_items oi
LEFT JOIN products p ON oi.product_id = p.product_id
WHERE p.product_id IS NULL;
```
➡️ There are no orders with empty product_id.

---

## 4. Outlier Detection

### Product dimensions
```sql
SELECT *
FROM products
WHERE product_weight_g < 0
   OR product_length_cm < 0
   OR product_height_cm < 0
   OR product_width_cm < 0;
```
➡️ There are no negative dimensions.

### Payments with zero or negative values
```sql
SELECT *
FROM payments
WHERE payment_value <= 0;
```
➡️ 9 zero values detected.
---

## 5. Creating Cleaned Views

Best practice: keep raw tables intact and create **views** for analysis.

```sql
CREATE VIEW clean_orders AS
SELECT *
FROM orders
WHERE order_status = 'delivered'
  AND order_delivered_customer_date IS NOT NULL;
```

```sql
CREATE VIEW clean_payments AS
SELECT *
FROM payments
WHERE payment_value > 0;
```

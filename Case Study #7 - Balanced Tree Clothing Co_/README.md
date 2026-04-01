
## 🌲 Case Study #7: Balanced Tree Analysis

## 📚 Table of Contents

* [Project Overview](#project-overview)
* [Database Schema](#database-schema)
* [Analysis Questions and Solutions](#analysis-questions-and-solutions)

All data for this case study has been referenced from [here](https://8weeksqlchallenge.com/case-study-7/).

---

## Project Overview

Balanced Tree Clothing specializes in providing an optimized collection of apparel and lifestyle products tailored for modern adventurers.

Danny, the company’s CEO, requested assistance in **evaluating sales performance and generating a basic financial summary** that the merchandising team can present to the company.

---

## Database Schema

<img width="932" alt="Database Schema" src="https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/2ce4df84-2b05-4fe9-a50c-47c903b392d5">

**Table: `product_details`**

| product_id | price | product_name                  | category_id | segment_id | style_id | category_name | segment_name | style_name     |
| :--------- | :---- | :---------------------------- | :---------- | :--------- | :------- | :------------ | :----------- | :------------- |
| c4a632     | 13    | Navy Oversized Jeans - Womens | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized |
| e83aa3     | 32    | Black Straight Jeans - Womens | 1           | 3          | 8        | Womens        | Jeans        | Black Straight |
| e31d39     | 10    | Cream Relaxed Jeans - Womens  | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed  |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens    | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit     |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens   | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain    |
| 9ec847     | 54    | Grey Fashion Jacket - Womens  | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion   |
| 5d267b     | 40    | White Tee Shirt - Mens        | 2           | 5          | 13       | Mens          | Shirt        | White Tee      |
| c8d436     | 10    | Teal Button Up Shirt - Mens   | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up |
| 2a2353     | 57    | Blue Polo Shirt - Mens        | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo      |
| f084eb     | 36    | Navy Solid Socks - Mens       | 2           | 6          | 16       | Mens          | Socks        | Navy Solid     |

**Table: `sales`**

| prod_id | qty | price | discount | member | txn_id | start_txn_time           |
| :------ | :-- | :---- | :------- | :----- | :----- | :----------------------- |
| c4a632  | 4   | 13    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 5d267b  | 4   | 40    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| b9a74d  | 4   | 17    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 2feb6b  | 2   | 29    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| c4a632  | 5   | 13    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| e31d39  | 2   | 10    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| 72f5d4  | 3   | 19    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| 2a2353  | 3   | 57    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| f084eb  | 3   | 36    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
| c4a632  | 1   | 13    | 21       | false  | ef648d | 2021-01-27T02:18:17.164Z |

**Table: `product_hierarchy`**

| id | parent_id | level_text     | level_name |
| :- | :-------- | :------------- | :--------- |
| 1  | null      | Womens         | Category   |
| 2  | null      | Mens           | Category   |
| 3  | 1         | Jeans          | Segment    |
| 4  | 1         | Jacket         | Segment    |
| 5  | 2         | Shirt          | Segment    |
| 6  | 2         | Socks          | Segment    |
| 7  | 3         | Navy Oversized | Style      |
| 8  | 3         | Black Straight | Style      |
| 9  | 3         | Cream Relaxed  | Style      |
| 10 | 4         | Khaki Suit     | Style      |

**Table: `product_prices`**

| id | product_id | price |
| :- | :--------- | :---- |
| 7  | c4a632     | 13    |
| 8  | e83aa3     | 32    |
| 9  | e31d39     | 10    |
| 10 | d5e9a6     | 23    |
| 11 | 72f5d4     | 19    |
| 12 | 9ec847     | 54    |
| 13 | 5d267b     | 40    |
| 14 | c8d436     | 10    |
| 15 | 2a2353     | 57    |
| 16 | f084eb     | 36    |

---

## Analysis Questions and Solutions

You can practice these queries in PostgreSQL using [DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8). For questions, feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/katiehuangx/).

---

### 📈 A. High-Level Sales Overview

**1. Total quantity sold for all products**

```sql
SELECT product.product_name, SUM(sales.qty) AS total_qty
FROM balanced_tree.sales
JOIN balanced_tree.product_details AS product
  ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```

**2. Total revenue before discounts**

```sql
SELECT product.product_name, SUM(sales.qty * sales.price) AS total_revenue
FROM balanced_tree.sales
JOIN balanced_tree.product_details AS product
  ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```

**3. Total discount amount for all products**

```sql
SELECT product.product_name, SUM(sales.qty * sales.price * sales.discount/100) AS total_discount
FROM balanced_tree.sales
JOIN balanced_tree.product_details AS product
  ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```

---

### 🧾 B. Transaction-Level Insights

**1. Count of unique transactions**

```sql
SELECT COUNT(DISTINCT txn_id) AS total_transactions
FROM balanced_tree.sales;
```

**2. Average number of unique products per transaction**

```sql
SELECT ROUND(AVG(total_qty)) AS avg_products
FROM (
  SELECT txn_id, SUM(qty) AS total_qty
  FROM balanced_tree.sales
  GROUP BY txn_id
) AS subquery;
```

**3. Revenue percentiles per transaction (25th, 50th, 75th)**

```sql
WITH revenue_cte AS (
  SELECT txn_id, SUM(price * qty) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)
SELECT
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) AS p25,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS p50,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) AS p75
FROM revenue_cte;
```

**4. Average discount per transaction**

```sql
SELECT ROUND(AVG(discount_amt)) AS avg_discount
FROM (
  SELECT txn_id, SUM(qty * price * discount/100) AS discount_amt
  FROM balanced_tree.sales
  GROUP BY txn_id
) AS sub;
```

**5. Member vs non-member transaction percentage**

```sql
WITH txn_cte AS (
  SELECT member, COUNT(DISTINCT txn_id) AS txn_count
  FROM balanced_tree.sales
  GROUP BY member
)
SELECT member, txn_count, ROUND(100*txn_count/SUM(txn_count) OVER()) AS pct
FROM txn_cte;
```

**6. Average revenue by membership status**

```sql
WITH rev_cte AS (
  SELECT member, txn_id, SUM(price*qty) AS revenue
  FROM balanced_tree.sales
  GROUP BY member, txn_id
)
SELECT member, ROUND(AVG(revenue),2) AS avg_revenue
FROM rev_cte
GROUP BY member;
```

---

### 👚 C. Product & Category Insights

**1. Top 3 products by revenue**

**2. Segment-level quantity, revenue, discount totals**

**3. Top-selling product per segment**

**4. Category-level totals**

**5. Top-selling product per category**

**6-10. Revenue splits, penetration, and product combinations** (similar analysis can be performed with SQL queries using window functions and joins)

---

### 📝 Reporting Challenge

Create a consolidated SQL script combining all questions above into a monthly report. The CFO requests these analytics for January, but it should be easy to run for February with minimal adjustments.

---

### 💡 Bonus Task

Transform `product_hierarchy` and `product_prices` into the `product_details` table using a **single SQL query**—a recursive CTE may be useful here.

---

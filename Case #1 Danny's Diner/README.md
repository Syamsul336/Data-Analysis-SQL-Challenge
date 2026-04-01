# 🍜 Case Study #1: Danny's Diner (SQL Analysis)


## 📚 Table of Contents
- [Overview](#overview)
- [Business Understanding](#business-understanding)
- [Data Model](#data-model)
- [Analysis & Solutions](#analysis--solutions)
- [Key Takeaways](#key-takeaways)

> All dataset and case study materials are sourced from:  
> https://8weeksqlchallenge.com/case-study-1/

---

## 📌 Overview
In this case study, I performed an exploratory data analysis using SQL on transactional data from **Danny's Diner**.  
The main objective is to uncover insights about customer behavior, purchasing patterns, and menu preferences.

---

## 🧠 Business Understanding
Danny aims to better understand his customers by analyzing available data.  
The key questions focus on:

- Customer visit frequency  
- Total spending per customer  
- Most popular menu items  
- Individual customer preferences  

These insights can support better decision-making, especially in marketing strategies and customer loyalty programs.

---

## 🗂️ Data Model

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

The dataset consists of three main tables:
- `sales` → transaction records  
- `menu` → product details and pricing  
- `members` → customer membership information  

---

## 📊 Analysis & Solutions

---

### 1. Total Amount Spent by Each Customer

```sql
SELECT 
  s.customer_id, 
  SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
````

**Result:**

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

➡️ Customer A has the highest total spending.

---

### 2. Number of Visit Days per Customer

```sql
SELECT 
  customer_id,
  COUNT(DISTINCT order_date) AS total_visits
FROM dannys_diner.sales
GROUP BY customer_id;
```

**Result:**

| customer_id | total_visits |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

➡️ Customer B is the most frequent visitor.

---

### 3. First Item Purchased by Each Customer

```sql
WITH ranked_orders AS (
  SELECT 
    s.customer_id,
    s.order_date,
    m.product_name,
    DENSE_RANK() OVER (
      PARTITION BY s.customer_id
      ORDER BY s.order_date
    ) AS order_rank
  FROM dannys_diner.sales s
  JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM ranked_orders
WHERE order_rank = 1;
```

**Result:**

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

➡️ Due to the absence of timestamps, multiple items may share the same "first purchase" date.

---

### 4. Most Purchased Item Overall

```sql
SELECT 
  m.product_name,
  COUNT(*) AS total_orders
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY total_orders DESC
LIMIT 1;
```

**Result:**

| product_name | total_orders |
| ------------ | ------------ |
| ramen        | 8            |

➡️ Ramen is the most popular item across all customers.

---

### 5. Most Popular Item for Each Customer

```sql
WITH fav_menu AS (
  SELECT 
    s.customer_id,
    m.product_name,
    COUNT(*) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY s.customer_id
      ORDER BY COUNT(*) DESC
    ) AS rank
  FROM dannys_diner.sales s
  JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM fav_menu
WHERE rank = 1;
```

**Insight:**

* Customers A and C prefer ramen
* Customer B shows equal preference for all items

---

### 6. First Purchase After Becoming a Member

```sql
WITH member_orders AS (
  SELECT
    m.customer_id,
    s.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY m.customer_id
      ORDER BY s.order_date
    ) AS rn
  FROM dannys_diner.members m
  JOIN dannys_diner.sales s
    ON m.customer_id = s.customer_id
   AND s.order_date > m.join_date
)

SELECT 
  mo.customer_id,
  me.product_name
FROM member_orders mo
JOIN dannys_diner.menu me
  ON mo.product_id = me.product_id
WHERE rn = 1;
```

➡️ Customer A → ramen
➡️ Customer B → sushi

---

### 7. Last Purchase Before Membership

```sql
WITH last_before_member AS (
  SELECT 
    m.customer_id,
    s.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY m.customer_id
      ORDER BY s.order_date DESC
    ) AS rn
  FROM dannys_diner.members m
  JOIN dannys_diner.sales s
    ON m.customer_id = s.customer_id
   AND s.order_date < m.join_date
)

SELECT 
  lbm.customer_id,
  me.product_name
FROM last_before_member lbm
JOIN dannys_diner.menu me
  ON lbm.product_id = me.product_id
WHERE rn = 1;
```

➡️ Both customers (A & B) purchased sushi right before becoming members.

---

### 8. Total Items and Spending Before Membership

```sql
SELECT 
  s.customer_id,
  COUNT(*) AS total_items,
  SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.members mb
  ON s.customer_id = mb.customer_id
 AND s.order_date < mb.join_date
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

**Insight:**

* Customer A → 2 items ($25)
* Customer B → 3 items ($40)

---

### 9. Loyalty Points Calculation

```sql
WITH points AS (
  SELECT 
    product_id,
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10
    END AS pts
  FROM dannys_diner.menu
)

SELECT 
  s.customer_id,
  SUM(p.pts) AS total_points
FROM dannys_diner.sales s
JOIN points p
  ON s.product_id = p.product_id
GROUP BY s.customer_id;
```

**Result:**

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

### 10. Points with First-Week Bonus

```sql
WITH period AS (
  SELECT 
    customer_id,
    join_date,
    join_date + 6 AS bonus_end
  FROM dannys_diner.members
)

SELECT 
  s.customer_id,
  SUM(
    CASE
      WHEN m.product_name = 'sushi' THEN m.price * 20
      WHEN s.order_date BETWEEN p.join_date AND p.bonus_end THEN m.price * 20
      ELSE m.price * 10
    END
  ) AS total_points
FROM dannys_diner.sales s
JOIN period p
  ON s.customer_id = p.customer_id
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

**Insight:**

* Customer A → 1020 points
* Customer B → 320 points

---

## 🎯 Key Takeaways

* Ramen is the most dominant item overall
* Customer B visits most frequently but has no strong preference
* Membership appears to influence purchasing behavior
* Loyalty programs can effectively drive sales, especially for targeted items like sushi

---

## 🚀 Skills Demonstrated

* SQL Joins (INNER JOIN, LEFT JOIN)
* Aggregations (SUM, COUNT)
* Window Functions (ROW_NUMBER, DENSE_RANK)
* Conditional Logic (CASE WHEN)
* Common Table Expressions (CTE)

---

# Case Study #6: Clique Bait

## 📚 Table of Contents

* [Business Task](#business-task)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Question and Solution](#question-and-solution)

All case study materials are referenced from the official source: [here](https://8weeksqlchallenge.com/case-study-6/).

---

## Business Task

Clique Bait is an e-commerce platform specializing in seafood products.

In this case study, the objective is to assist Danny (Founder & CEO) in analyzing user behavior across the platform. The main focus is to understand user journeys and calculate funnel conversion and drop-off rates.

The insights derived from this analysis will help improve marketing strategies, optimize user experience, and increase overall conversion rates.

---

## Entity Relationship Diagram

<img width="825" alt="image" src="https://user-images.githubusercontent.com/81607668/134619326-f560a7b0-23b2-42ba-964b-95b3c8d55c76.png">

### Tables Overview

* `users`: Contains user and cookie-level data
* `events`: Tracks all user interactions
* `event_identifier`: Maps event types to descriptions
* `page_hierarchy`: Contains page and product details
* `campaign_identifier`: Contains marketing campaign periods

---

## Question and Solution

You can run all queries using PostgreSQL via [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17).

---

## 👩🏻‍💻 A. Digital Analysis

### 1. Total number of users

```sql
SELECT 
  COUNT(DISTINCT user_id) AS user_count
FROM clique_bait.users;
```

---

### 2. Average number of cookies per user

```sql
WITH cookie AS (
  SELECT 
    user_id, 
    COUNT(cookie_id) AS cookie_id_count
  FROM clique_bait.users
  GROUP BY user_id
)

SELECT 
  ROUND(AVG(cookie_id_count),0) AS avg_cookie_id
FROM cookie;
```

---

### 3. Monthly unique visits

```sql
SELECT 
  EXTRACT(MONTH FROM event_time) AS month, 
  COUNT(DISTINCT visit_id) AS unique_visit_count
FROM clique_bait.events
GROUP BY EXTRACT(MONTH FROM event_time);
```

---

### 4. Event count by type

```sql
SELECT 
  event_type, 
  COUNT(*) AS event_count
FROM clique_bait.events
GROUP BY event_type
ORDER BY event_type;
```

---

### 5. Percentage of visits with a purchase

```sql
SELECT 
  100 * COUNT(DISTINCT e.visit_id) /
    (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events) 
    AS percentage_purchase
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei
  ON e.event_type = ei.event_type
WHERE ei.event_name = 'Purchase';
```

---

### 6. Percentage of checkout views without purchase

```sql
WITH checkout_purchase AS (
SELECT 
  visit_id,
  MAX(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
  MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
FROM clique_bait.events
GROUP BY visit_id)

SELECT 
  ROUND(100 * (1 - (SUM(purchase)::numeric / SUM(checkout))),2) 
  AS percentage_checkout_view_with_no_purchase
FROM checkout_purchase;
```

---

### 7. Top 3 most viewed pages

```sql
SELECT 
  ph.page_name, 
  COUNT(*) AS page_views
FROM clique_bait.events AS e
JOIN clique_bait.page_hierarchy AS ph
  ON e.page_id = ph.page_id
WHERE e.event_type = 1
GROUP BY ph.page_name
ORDER BY page_views DESC
LIMIT 3;
```

---

### 8. Views and cart adds per product category

```sql
SELECT 
  ph.product_category, 
  SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM clique_bait.events AS e
JOIN clique_bait.page_hierarchy AS ph
  ON e.page_id = ph.page_id
WHERE ph.product_category IS NOT NULL
GROUP BY ph.product_category
ORDER BY page_views DESC;
```

---

## 👩🏻‍💻 B. Product Funnel Analysis

### Build product funnel dataset

```sql
WITH product_page_events AS (
  SELECT 
    e.visit_id,
    ph.product_id,
    ph.page_name AS product_name,
    ph.product_category,
    SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_view,
    SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_add
  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph
    ON e.page_id = ph.page_id
  WHERE product_id IS NOT NULL
  GROUP BY e.visit_id, ph.product_id, ph.page_name, ph.product_category
),
purchase_events AS (
  SELECT DISTINCT visit_id
  FROM clique_bait.events
  WHERE event_type = 3
),
combined_table AS (
  SELECT 
    ppe.visit_id, 
    ppe.product_id, 
    ppe.product_name, 
    ppe.product_category, 
    ppe.page_view, 
    ppe.cart_add,
    CASE WHEN pe.visit_id IS NOT NULL THEN 1 ELSE 0 END AS purchase
  FROM product_page_events AS ppe
  LEFT JOIN purchase_events AS pe
    ON ppe.visit_id = pe.visit_id
),
product_info AS (
  SELECT 
    product_name, 
    product_category, 
    SUM(page_view) AS views,
    SUM(cart_add) AS cart_adds, 
    SUM(CASE WHEN cart_add = 1 AND purchase = 0 THEN 1 ELSE 0 END) AS abandoned,
    SUM(CASE WHEN cart_add = 1 AND purchase = 1 THEN 1 ELSE 0 END) AS purchases
  FROM combined_table
  GROUP BY product_id, product_name, product_category
)

SELECT *
FROM product_info
ORDER BY product_id;
```

---

### Key Insights

* Oyster recorded the highest number of views
* Lobster generated the most cart additions and purchases
* Russian Caviar showed the highest abandonment rate

---

### Highest conversion (view → purchase)

```sql
SELECT 
  product_name, 
  product_category, 
  ROUND(100 * purchases/views,2) AS purchase_per_view_percentage
FROM product_info
ORDER BY purchase_per_view_percentage DESC;
```

---

### Conversion rates

```sql
SELECT 
  ROUND(100 * AVG(cart_adds/views),2) AS avg_view_to_cart_add_conversion,
  ROUND(100 * AVG(purchases/cart_adds),2) AS avg_cart_add_to_purchase_conversion
FROM product_info;
```

---

## 👩🏻‍💻 C. Campaign Analysis

### Build visit-level dataset

```sql
SELECT 
  u.user_id, 
  e.visit_id, 
  MIN(e.event_time) AS visit_start_time,
  SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
  SUM(CASE WHEN e.event_type = 3 THEN 1 ELSE 0 END) AS purchase,
  c.campaign_name,
  SUM(CASE WHEN e.event_type = 4 THEN 1 ELSE 0 END) AS impression, 
  SUM(CASE WHEN e.event_type = 5 THEN 1 ELSE 0 END) AS click, 
  STRING_AGG(
    CASE 
      WHEN p.product_id IS NOT NULL AND e.event_type = 2 
      THEN p.page_name 
      ELSE NULL 
    END, ', ' ORDER BY e.sequence_number
  ) AS cart_products
FROM clique_bait.users AS u
INNER JOIN clique_bait.events AS e
  ON u.cookie_id = e.cookie_id
LEFT JOIN clique_bait.campaign_identifier AS c
  ON e.event_time BETWEEN c.start_date AND c.end_date
LEFT JOIN clique_bait.page_hierarchy AS p
  ON e.page_id = p.page_id
GROUP BY u.user_id, e.visit_id, c.campaign_name;
```

---

## 📊 Key Business Insights

* Campaign impressions alone do not guarantee purchases
* Click-through behavior is strongly correlated with higher conversion rates
* Some products attract high interest but fail to convert (high abandonment)
* Lobster stands out as the most efficient product in the funnel
* Checkout abandonment indicates friction in the purchase process

---

## 💡 Recommendations

* Improve checkout experience to reduce drop-offs
* Optimize campaigns to increase click-through rates
* Focus marketing efforts on high-conversion products
* Investigate reasons behind high abandonment products
* Use behavioral segmentation for targeted campaigns

---

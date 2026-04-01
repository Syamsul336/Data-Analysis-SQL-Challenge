
# 🍕 Case Study #2: Pizza Runner (SQL Analysis)

## 📚 Section A: Pizza Metrics

In this section, I analyze key operational metrics related to pizza orders, deliveries, and customer preferences using SQL.

---

### 1. Total Number of Pizzas Ordered

```sql
SELECT COUNT(*) AS total_pizzas
FROM #customer_orders;
````

**Result:**

* A total of **14 pizzas** were ordered.

---

### 2. Number of Unique Customer Orders

```sql
SELECT 
  COUNT(DISTINCT order_id) AS unique_orders
FROM #customer_orders;
```

**Result:**

* There were **10 distinct orders** placed by customers.

---

### 3. Successful Deliveries by Each Runner

```sql
SELECT 
  runner_id, 
  COUNT(order_id) AS successful_deliveries
FROM #runner_orders
WHERE distance != 0
GROUP BY runner_id;
```

**Result:**

* Runner 1 → 4 deliveries
* Runner 2 → 3 deliveries
* Runner 3 → 1 delivery

➡️ Runner 1 handled the highest number of successful deliveries.

---

### 4. Delivered Pizza Count by Type

```sql
SELECT 
  p.pizza_name, 
  COUNT(c.pizza_id) AS total_delivered
FROM #customer_orders c
JOIN #runner_orders r
  ON c.order_id = r.order_id
JOIN pizza_names p
  ON c.pizza_id = p.pizza_id
WHERE r.distance != 0
GROUP BY p.pizza_name;
```

**Result:**

* Meatlovers → 9 pizzas
* Vegetarian → 3 pizzas

➡️ Meatlovers is significantly more popular among delivered orders.

---

### 5. Pizza Preferences by Customer

```sql
SELECT 
  c.customer_id, 
  p.pizza_name, 
  COUNT(*) AS order_count
FROM #customer_orders c
JOIN pizza_names p
  ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```

**Insights:**

* Customer 101 → 2 Meatlovers, 1 Vegetarian
* Customer 102 → 2 Meatlovers, 2 Vegetarian
* Customer 103 → 3 Meatlovers, 1 Vegetarian
* Customer 104 → 1 Meatlovers
* Customer 105 → 1 Vegetarian

➡️ Most customers show a preference toward Meatlovers pizza.

---

### 6. Maximum Pizzas Delivered in a Single Order

```sql
WITH order_summary AS (
  SELECT 
    c.order_id, 
    COUNT(c.pizza_id) AS pizzas_per_order
  FROM #customer_orders c
  JOIN #runner_orders r
    ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.order_id
)

SELECT 
  MAX(pizzas_per_order) AS max_pizzas
FROM order_summary;
```

**Result:**

* The maximum number of pizzas delivered in a single order is **3 pizzas**.

---

### 7. Orders with Changes vs No Changes

```sql
SELECT 
  c.customer_id,
  SUM(
    CASE 
      WHEN c.exclusions <> ' ' OR c.extras <> ' ' THEN 1
      ELSE 0
    END
  ) AS orders_with_changes,
  SUM(
    CASE 
      WHEN c.exclusions = ' ' AND c.extras = ' ' THEN 1
      ELSE 0
    END
  ) AS orders_without_changes
FROM #customer_orders c
JOIN #runner_orders r
  ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

**Insights:**

* Customers 101 & 102 → prefer standard recipes
* Customers 103, 104, 105 → prefer customization (extras/exclusions)

➡️ Indicates different customer segments: standard vs customized preferences.

---

### 8. Delivered Pizzas with Both Extras and Exclusions

```sql
SELECT  
  COUNT(*) AS pizzas_with_both_changes
FROM #customer_orders c
JOIN #runner_orders r
  ON c.order_id = r.order_id
WHERE r.distance > 0 
  AND c.exclusions <> ' ' 
  AND c.extras <> ' ';
```

**Result:**

* Only **1 pizza** had both extras and exclusions.

➡️ Very few customers make complex customizations.

---

### 9. Hourly Order Volume

```sql
SELECT 
  DATEPART(HOUR, order_time) AS hour,
  COUNT(order_id) AS total_orders
FROM #customer_orders
GROUP BY DATEPART(HOUR, order_time)
ORDER BY hour;
```

**Insights:**

* Peak hours → 13:00, 18:00, 21:00
* Low activity → 11:00, 19:00, 23:00

➡️ Orders spike during lunch and dinner times.

---

### 10. Orders by Day of the Week

```sql
SELECT 
  FORMAT(DATEADD(DAY, 2, order_time),'dddd') AS day_of_week,
  COUNT(order_id) AS total_orders
FROM #customer_orders
GROUP BY FORMAT(DATEADD(DAY, 2, order_time),'dddd');
```

**Insights:**

* Monday & Friday → 5 orders
* Saturday → 3 orders
* Sunday → 1 order

➡️ Higher demand observed at the beginning and end of the workweek.

---

## 🎯 Key Takeaways

* Meatlovers pizza dominates overall demand
* Peak ordering times align with lunch and dinner hours
* Customer behavior varies between standard and customized orders
* Delivery performance differs across runners, with Runner 1 leading

---

## 🚀 Skills Demonstrated

* SQL Aggregation (COUNT, MAX)
* JOIN Operations
* Common Table Expressions (CTE)
* Conditional Logic (CASE WHEN)
* Time-based Analysis (DATEPART, FORMAT)

---

## 🔗 Next Section

For further analysis on delivery performance and customer experience, check out:

👉 Part B
---
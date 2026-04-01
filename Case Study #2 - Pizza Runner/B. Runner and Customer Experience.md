# 🍕 Case Study #2: Pizza Runner (SQL Analysis)

## 📚 Section B: Runner and Customer Experience

This section focuses on analyzing runner performance and overall customer delivery experience using SQL.

---

### 1. Runner Signups by Weekly Period

```sql
SELECT 
  DATEPART(WEEK, registration_date) AS registration_week,
  COUNT(runner_id) AS total_signups
FROM runners
GROUP BY DATEPART(WEEK, registration_date);
````

**Insight:**

* Week 1 (Jan 2021) → 2 runners signed up
* Week 2 & 3 → 1 runner each

➡️ Initial onboarding was strong but slowed down in the following weeks.

---

### 2. Average Pickup Time (in Minutes)

```sql
WITH pickup_time_cte AS (
  SELECT 
    c.order_id,
    c.order_time,
    r.pickup_time,
    DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS pickup_duration
  FROM #customer_orders c
  JOIN #runner_orders r
    ON c.order_id = r.order_id
  WHERE r.distance != 0
)

SELECT 
  AVG(pickup_duration) AS avg_pickup_minutes
FROM pickup_time_cte
WHERE pickup_duration > 1;
```

**Result:**

* Average pickup time ≈ **15 minutes**

➡️ This reflects the average preparation + dispatch time at the kitchen.

---

### 3. Relationship Between Order Size and Preparation Time

```sql
WITH prep_time_cte AS (
  SELECT 
    c.order_id,
    COUNT(*) AS total_pizzas,
    DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS prep_time
  FROM #customer_orders c
  JOIN #runner_orders r
    ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT 
  total_pizzas,
  AVG(prep_time) AS avg_prep_time
FROM prep_time_cte
WHERE prep_time > 1
GROUP BY total_pizzas;
```

**Insights:**

* 1 pizza → ~12 minutes
* 2 pizzas → ~16 minutes (~8 min/pizza)
* 3 pizzas → ~30 minutes (~10 min/pizza)

➡️ Preparing 2 pizzas in one order appears to be the most efficient.

---

### 4. Average Delivery Distance per Customer

```sql
SELECT 
  c.customer_id,
  AVG(r.distance) AS avg_distance_km
FROM #customer_orders c
JOIN #runner_orders r
  ON c.order_id = r.order_id
WHERE r.duration != 0
GROUP BY c.customer_id;
```

**Insights:**

* Customer 104 → closest (~10 km)
* Customer 105 → furthest (~25 km)

➡️ Delivery distance varies significantly and may impact delivery time and cost.

---

### 5. Difference Between Longest and Shortest Delivery Time

```sql
SELECT 
  MAX(duration::NUMERIC) - MIN(duration::NUMERIC) AS delivery_time_gap
FROM runner_orders2
WHERE duration NOT LIKE ' ';
```

**Result:**

* Delivery time difference = **30 minutes**
  (Longest: 40 mins | Shortest: 10 mins)

➡️ Indicates variability in delivery performance.

---

### 6. Average Speed per Runner per Delivery

```sql
SELECT 
  r.runner_id,
  c.customer_id,
  c.order_id,
  COUNT(*) AS pizza_count,
  r.distance,
  (r.duration / 60) AS duration_hours,
  ROUND((r.distance / r.duration * 60), 2) AS avg_speed_kmh
FROM #runner_orders r
JOIN #customer_orders c
  ON r.order_id = c.order_id
WHERE r.distance != 0
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY c.order_id;
```

**Insights:**

* Runner 1 → 37.5 – 60 km/h
* Runner 2 → 35.1 – 93.6 km/h ⚠️
* Runner 3 → ~40 km/h

➡️ Runner 2 shows unusually high variability and may require further investigation.

---

### 7. Successful Delivery Rate per Runner

```sql
SELECT 
  runner_id,
  ROUND(
    100.0 * SUM(CASE WHEN distance = 0 THEN 0 ELSE 1 END) 
    / COUNT(*), 
  0) AS success_rate
FROM #runner_orders
GROUP BY runner_id;
```

**Result:**

* Runner 1 → 100%
* Runner 2 → 75%
* Runner 3 → 50%

➡️ Runner 1 demonstrates the most consistent performance.

> ⚠️ Note: Delivery success may also be influenced by factors outside the runner’s control (e.g., order cancellations).

---

## 🎯 Key Takeaways

* Runner performance varies significantly across individuals
* Delivery time inconsistency suggests operational inefficiencies
* Moderate order sizes (2 pizzas) yield optimal preparation efficiency
* Distance plays a key role in delivery performance and customer experience

---

## 🚀 Skills Demonstrated

* SQL Joins & Filtering
* Aggregation & Metrics Calculation
* CTE (Common Table Expressions)
* Time Analysis (DATEDIFF, DATEPART)
* Performance Analysis (speed, success rate)

---

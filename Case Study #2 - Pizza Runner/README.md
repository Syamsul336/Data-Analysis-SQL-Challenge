# 🍕 Case Study #2: Pizza Runner (SQL End-to-End Analysis)

## 📚 Table of Contents
- [Overview](#overview)
- [Business Problem](#business-problem)
- [Data Model](#data-model)
- [Data Cleaning](#-data-cleaning--transformation)
- [Analysis](#analysis)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner Performance](#b-runner-performance)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing & Profitability](#d-pricing--profitability)
- [Key Insights](#key-insights)
- [Business Recommendations](#business-recommendations)

> Dataset source: https://8weeksqlchallenge.com/case-study-2/

---

## 📌 Overview
This project analyzes the operations of **Pizza Runner**, a fictional pizza delivery startup.  
The analysis covers the full pipeline: from raw data cleaning to extracting business insights related to customers, delivery performance, and profitability.

---

## 🧠 Business Problem
Danny is scaling his pizza business into a delivery-based model by introducing **Pizza Runner**.

Key challenges include:
- Understanding customer ordering behavior  
- Evaluating runner (delivery) performance  
- Managing ingredient usage and customization  
- Calculating revenue and operational efficiency  

---

## 🗂️ Data Model

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

Main tables:
- `customer_orders`
- `runner_orders`
- `pizza_names`
- `pizza_recipes`
- `pizza_toppings`
- `runners`

---

## 🧼 Data Cleaning & Transformation

### Issues Identified:
- NULL and inconsistent values (`'null'`, `' '`)
- Mixed data formats (e.g., `"10km"`, `"20 minutes"`)
- Incorrect data types (text instead of numeric)

---

### Cleaning `customer_orders`

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
    WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ' '
    ELSE exclusions
  END AS exclusions,
  CASE
    WHEN extras IS NULL OR extras LIKE 'null' THEN ' '
    ELSE extras
  END AS extras,
  order_time
FROM pizza_runner.customer_orders;
````

---

### Cleaning `runner_orders`

```sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id,  
  CASE
    WHEN pickup_time LIKE 'null' THEN ' '
    ELSE pickup_time
  END AS pickup_time,
  CASE
    WHEN distance LIKE 'null' THEN ' '
    WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
    ELSE distance 
  END AS distance,
  CASE
    WHEN duration LIKE 'null' THEN ' '
    WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
    ELSE duration
  END AS duration,
  CASE
    WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ' '
    ELSE cancellation
  END AS cancellation
FROM pizza_runner.runner_orders;
```

```sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time DATETIME,
ALTER COLUMN distance FLOAT,
ALTER COLUMN duration INT;
```

---

## 📊 Analysis

---

## A. Pizza Metrics

### Key Findings:

* Total pizzas ordered → **14**
* Unique orders → **10**
* Most popular item → **Meatlovers**
* Peak order times → lunch & dinner hours
* Maximum pizzas in one order → **3**

➡️ Demand is concentrated during meal times and dominated by meat-based options.

---

## B. Runner Performance

### Key Findings:

* Average pickup time → **~15 minutes**
* Runner 1 → best performance (100% success rate)
* Runner 2 → inconsistent speed (potential anomaly)
* Optimal prep efficiency → **2 pizzas per order**

➡️ Delivery performance varies significantly and impacts customer experience.

---

## C. Ingredient Optimisation

### Key Findings:

* Customers frequently customize orders (extras & exclusions)
* Certain toppings are consistently preferred
* Some ingredients are repeatedly excluded

➡️ Ingredient-level insights can improve inventory planning and upselling strategies.

---

## D. Pricing & Profitability

### Key Considerations:

* Base pricing:

  * Meatlovers → $12
  * Vegetarian → $10
* Additional cost:

  * Extras → +$1
* Delivery cost:

  * $0.30 per km

➡️ Profitability depends on balancing:

* Ingredient cost
* Delivery distance
* Order volume

---

## 🎯 Key Insights

* 🍕 Meatlovers dominates customer preference
* ⏰ Orders peak during lunch and dinner
* 🚴 Runner performance is inconsistent across individuals
* ⚙️ Medium-sized orders (2 pizzas) are most efficient
* 🧀 Customization plays a major role in customer behavior

---

## 💡 Business Recommendations

### 1. Optimize Delivery Assignment

Assign runners based on distance and past performance to reduce delivery time variability.

---

### 2. Standardize Kitchen Workflow

Focus on optimizing preparation for 2–3 pizza orders to maximize efficiency.

---

### 3. Investigate Runner Anomalies

Review extreme speed variations (e.g., Runner 2) for potential data or operational issues.

---

### 4. Leverage Popular Items

Promote best-selling pizzas (e.g., Meatlovers) through bundles or discounts.

---

### 5. Improve Inventory Planning

Track frequently added and removed ingredients to optimize stock management.

---

### 6. Introduce Customer Rating System

Collect feedback on delivery experience to monitor runner performance.

---

## 🚀 Skills Demonstrated

* Data Cleaning & Transformation
* SQL Joins & Aggregations
* Window Functions
* Time-based Analysis
* Business Insight Generation
* End-to-End Data Analysis Workflow

---


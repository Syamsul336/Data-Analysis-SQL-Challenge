# 🍕 Case Study #2: Pizza Runner (SQL Analysis)

## 🧼 Data Cleaning & Transformation

Before performing any analysis, it is crucial to clean and standardize the dataset to ensure accuracy and consistency.  
In this section, I focus on handling missing values, inconsistent formats, and incorrect data types.

---

## 🔨 Table: `customer_orders`

### 📌 Data Issues Identified
From the `customer_orders` table, the following problems were observed:

- The `exclusions` column contains:
  - NULL values  
  - String values like `'null'`  
  - Blank spaces `' '`  

- The `extras` column has similar inconsistencies:
  - NULL values  
  - `'null'` strings  
  - Empty spaces  

These inconsistencies can lead to incorrect analysis if not properly handled.

---

### 🛠️ Data Cleaning Approach
To resolve these issues:
- Create a temporary table for cleaned data  
- Standardize missing values by replacing NULL and `'null'` with blank space `' '`  

---

### 💻 SQL Implementation

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

### ✅ Result

The cleaned `customer_orders_temp` table now has:

* Consistent formatting for missing values
* No ambiguity between NULL and empty strings

➡️ This table will be used for all subsequent analysis.

---

## 🔨 Table: `runner_orders`

### 📌 Data Issues Identified

From the `runner_orders` table, several inconsistencies were found:

* `pickup_time`:

  * Contains `'null'` values

* `distance`:

  * Includes unit suffix `"km"`
  * Contains NULL and `'null'` values

* `duration`:

  * Includes inconsistent formats such as `"mins"`, `"minute"`, `"minutes"`
  * Contains NULL values

* `cancellation`:

  * Contains NULL and `'null'` values

---

### 🛠️ Data Cleaning Approach

To standardize the dataset:

* Replace NULL and `'null'` values with blank space `' '`
* Remove text units from numeric columns:

  * `"km"` from `distance`
  * `"mins"`, `"minute"`, `"minutes"` from `duration`
* Ensure all values are in a consistent numeric format
* Convert columns into appropriate data types

---

### 💻 SQL Implementation

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

---

### 🔄 Data Type Conversion

After cleaning, the columns are converted into appropriate data types:

```sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time DATETIME,
ALTER COLUMN distance FLOAT,
ALTER COLUMN duration INT;
```

---

### ✅ Result

The cleaned `runner_orders_temp` table now provides:

* Proper numeric values for `distance` and `duration`
* Standardized timestamps for `pickup_time`
* Consistent handling of missing values

➡️ This ensures reliable analysis for delivery performance and operational metrics.

---

## 🎯 Key Takeaways

* Data cleaning is a critical first step before any analysis
* Handling NULL and inconsistent formats prevents misleading insights
* Standardizing units (km, minutes) improves data usability
* Proper data types enable more accurate calculations and aggregations

---

## 🚀 Skills Demonstrated

* Data Cleaning in SQL
* Handling NULL and inconsistent string values
* String manipulation (TRIM, LIKE)
* Data type conversion (CAST / ALTER)
* Data preprocessing for analytics

---

## 🔗 Next Section

Continue the analysis here:

👉 A. Pizza Metrics
---


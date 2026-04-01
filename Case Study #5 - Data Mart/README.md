# Case Study #5: Data Mart

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

All information used in this case study is adapted from the official source: https://8weeksqlchallenge.com/case-study-5/

***

## Business Task

Data Mart is an online grocery platform specializing in fresh produce.

In June 2020, the company implemented a major operational change by introducing sustainable packaging across all stages—from suppliers to end customers.

The goal of this analysis is to measure how this change impacted sales performance across different dimensions of the business.

Key business questions:
- What was the measurable impact after June 2020?
- Which segments (platform, region, customer type) were most affected?
- What improvements can be made for future sustainability initiatives?

---

## Entity Relationship Diagram

Dataset used: `data_mart.weekly_sales`

<img width="287" alt="image" src="https://user-images.githubusercontent.com/81607668/131438278-45e6a4e8-7cf5-468a-937b-2c306a792782.png">

---

## 🧼 A. Data Cleaning

```sql
DROP TABLE IF EXISTS clean_weekly_sales;

CREATE TEMP TABLE clean_weekly_sales AS (
SELECT
  TO_DATE(week_date, 'DD/MM/YY') AS week_date,
  DATE_PART('week', TO_DATE(week_date, 'DD/MM/YY')) AS week_number,
  DATE_PART('month', TO_DATE(week_date, 'DD/MM/YY')) AS month_number,
  DATE_PART('year', TO_DATE(week_date, 'DD/MM/YY')) AS calendar_year,
  region, 
  platform, 
  segment,
  CASE 
    WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
    WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
    WHEN RIGHT(segment,1) IN ('3','4') THEN 'Retirees'
    ELSE 'unknown' 
  END AS age_band,
  CASE 
    WHEN LEFT(segment,1) = 'C' THEN 'Couples'
    WHEN LEFT(segment,1) = 'F' THEN 'Families'
    ELSE 'unknown' 
  END AS demographic,
  transactions,
  ROUND((sales::NUMERIC / transactions),2) AS avg_transaction,
  sales
FROM data_mart.weekly_sales
);
````

---

## 🛍 B. Data Exploration

### 1. Day of the week

```sql
SELECT DISTINCT TO_CHAR(week_date, 'day') AS week_day
FROM clean_weekly_sales;
```

👉 Result: All weeks start on **Monday**

---

### 2. Missing week numbers

```sql
WITH week_number_cte AS (
  SELECT GENERATE_SERIES(1,52) AS week_number
)

SELECT DISTINCT week_no.week_number
FROM week_number_cte AS week_no
LEFT JOIN clean_weekly_sales AS sales
  ON week_no.week_number = sales.week_number
WHERE sales.week_number IS NULL;
```

👉 Result: **28 weeks are missing**

---

### 3. Total transactions per year

```sql
SELECT 
  calendar_year, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```

---

### 4. Monthly sales by region

```sql
SELECT 
  month_number, 
  region, 
  SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY month_number, region
ORDER BY month_number, region;
```

---

### 5. Transactions per platform

```sql
SELECT 
  platform, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform;
```

---

### 6. Percentage sales by platform

```sql
WITH monthly_transactions AS (
  SELECT 
    calendar_year, 
    month_number, 
    platform, 
    SUM(sales) AS monthly_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year, month_number, platform
)

SELECT 
  calendar_year, 
  month_number,
  ROUND(100 * MAX(CASE WHEN platform = 'Retail' THEN monthly_sales END) 
        / SUM(monthly_sales),2) AS retail_percentage,
  ROUND(100 * MAX(CASE WHEN platform = 'Shopify' THEN monthly_sales END) 
        / SUM(monthly_sales),2) AS shopify_percentage
FROM monthly_transactions
GROUP BY calendar_year, month_number
ORDER BY calendar_year, month_number;
```

---

### 7. Sales percentage by demographic

```sql
WITH demographic_sales AS (
  SELECT 
    calendar_year, 
    demographic, 
    SUM(sales) AS yearly_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year, demographic
)

SELECT 
  calendar_year,
  ROUND(100 * MAX(CASE WHEN demographic = 'Couples' THEN yearly_sales END) 
        / SUM(yearly_sales),2) AS couples_percentage,
  ROUND(100 * MAX(CASE WHEN demographic = 'Families' THEN yearly_sales END) 
        / SUM(yearly_sales),2) AS families_percentage,
  ROUND(100 * MAX(CASE WHEN demographic = 'unknown' THEN yearly_sales END) 
        / SUM(yearly_sales),2) AS unknown_percentage
FROM demographic_sales
GROUP BY calendar_year;
```

---

### 8. Contribution by age_band & demographic (Retail only)

```sql
SELECT 
  age_band, 
  demographic, 
  SUM(sales) AS retail_sales,
  ROUND(100 * SUM(sales)::NUMERIC 
        / SUM(SUM(sales)) OVER (),1) AS contribution_percentage
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY age_band, demographic
ORDER BY retail_sales DESC;
```

---

### 9. Average transaction size comparison

```sql
SELECT 
  calendar_year, 
  platform, 
  ROUND(AVG(avg_transaction),0) AS avg_transaction_row, 
  SUM(sales) / SUM(transactions) AS avg_transaction_group
FROM clean_weekly_sales
GROUP BY calendar_year, platform
ORDER BY calendar_year, platform;
```

---

## 🧼 C. Before & After Analysis

### Step: Identify baseline week

```sql
SELECT DISTINCT week_number
FROM clean_weekly_sales
WHERE week_date = '2020-06-15' 
  AND calendar_year = 2020;
```

👉 Result: Week 25

---

### 1. 4 Weeks Before vs After

```sql
WITH packaging_sales AS (
  SELECT 
    week_date, 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE week_number BETWEEN 21 AND 28
    AND calendar_year = 2020
  GROUP BY week_date, week_number
),
before_after_changes AS (
  SELECT 
    SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS before_sales,
    SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS after_sales
  FROM packaging_sales
)

SELECT 
  after_sales - before_sales AS sales_variance,
  ROUND(100 * (after_sales - before_sales) / before_sales,2) AS variance_percentage
FROM before_after_changes;
```

---

### 2. 12 Weeks Before vs After

```sql
WITH packaging_sales AS (
  SELECT 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE week_number BETWEEN 13 AND 37
    AND calendar_year = 2020
  GROUP BY week_number
),
before_after_changes AS (
  SELECT 
    SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before_sales,
    SUM(CASE WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) AS after_sales
  FROM packaging_sales
)

SELECT 
  after_sales - before_sales AS sales_variance,
  ROUND(100 * (after_sales - before_sales) / before_sales,2) AS variance_percentage
FROM before_after_changes;
```

---

### 3. Comparison with previous years

```sql
WITH changes AS (
  SELECT 
    calendar_year, 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE week_number BETWEEN 13 AND 37
  GROUP BY calendar_year, week_number
),
before_after_changes AS (
  SELECT 
    calendar_year,
    SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before_sales,
    SUM(CASE WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) AS after_sales
  FROM changes
  GROUP BY calendar_year
)

SELECT 
  calendar_year,
  after_sales - before_sales AS sales_variance,
  ROUND(100 * (after_sales - before_sales) / before_sales,2) AS variance_percentage
FROM before_after_changes;
```

---

## 🎯 Key Insights

* Sales declined after the sustainable packaging implementation
* The negative impact becomes stronger over a longer time horizon
* Retail dominates the business, while Shopify contribution is minimal
* A large portion of customer data is labeled as "unknown"

---

## 💡 Recommendations

* Improve product visibility after packaging changes
* Conduct gradual rollout instead of full implementation
* Enhance customer data quality for better segmentation
* Perform A/B testing before major operational changes

---

⭐ If you found this project useful, feel free to give it a star!


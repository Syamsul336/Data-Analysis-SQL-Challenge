
# 🥑 Case Study #3: Foodie-Fi

---

## 📚 Table of Contents

* [Business Overview](#business-overview)
* [Data Model](#data-model)
* [Analysis & Solutions](#analysis--solutions)

> 📌 All case study materials are sourced from: [https://8weeksqlchallenge.com/case-study-3/](https://8weeksqlchallenge.com/case-study-3/)

---

## 🧾 Business Overview

Foodie-Fi is a subscription-based streaming platform founded by Danny and his partners. The platform offers exclusive food-related video content from around the world through monthly and annual subscription plans.

The main objective of this case study is to analyze customer behavior, subscription transitions, and business performance using transactional subscription data.

---

## 🗂️ Data Model

### 📌 Table: `plans`

This table defines the available subscription plans:

* **Trial Plan**

  * Free 7-day trial for new users
  * Automatically converts to a paid plan unless changed or canceled

* **Basic Monthly**

  * Limited access
  * Monthly fee: $9.90

* **Pro Monthly**

  * Unlimited streaming + downloads
  * Monthly fee: $19.90

* **Pro Annual**

  * Discounted yearly plan
  * Annual fee: $199

* **Churn**

  * Indicates canceled subscriptions
  * Access continues until billing cycle ends

---

### 📌 Table: `subscriptions`

This table records when each customer starts a specific plan.

Important rules:

* Plan changes only take effect at the `start_date`
* Upgrades → effective immediately
* Downgrades/cancellations → effective at end of billing cycle
* Churn reflects cancellation decision date, not access end date

---

## 📊 Analysis & Solutions

---

## 🎬 A. Customer Journey

To understand how users move between plans, we extract subscription timelines:

```sql
SELECT
  sub.customer_id,
  p.plan_id,
  p.plan_name,
  sub.start_date
FROM foodie_fi.subscriptions AS sub
JOIN foodie_fi.plans AS p
  ON sub.plan_id = p.plan_id
WHERE sub.customer_id IN (1,2,11,13,15,16,18,19);
```

### 🔍 Example Customer Journeys

* **Customer 1**

  * Started trial: 1 Aug 2020
  * Converted to basic: 8 Aug 2020

* **Customer 13**

  * Trial: 15 Dec 2020
  * Basic: 22 Dec 2020
  * Upgraded to Pro: 29 Mar 2021

* **Customer 15**

  * Trial: 17 Mar 2020
  * Pro: 24 Mar 2020
  * Churned: 29 Apr 2020

---

## 📈 B. Data Analysis

---

### 1. Total Number of Customers

```sql
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM foodie_fi.subscriptions;
```

✅ **Result:** 1,000 customers

---

### 2. Monthly Trial Distribution

```sql
SELECT
  DATE_PART('month', start_date) AS month,
  COUNT(customer_id) AS total_trials
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY month
ORDER BY month;
```

📌 Insight:

* March has the highest trial sign-ups
* February has the lowest

---

### 3. Plans After 2020

```sql
SELECT
  p.plan_id,
  p.plan_name,
  COUNT(s.customer_id) AS total_events
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;
```

---

### 4. Churn Rate

```sql
SELECT
  COUNT(DISTINCT s.customer_id) AS churned_customers,
  ROUND(
    100.0 * COUNT(DISTINCT s.customer_id) /
    (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),
  1) AS churn_percentage
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id = 4;
```

📌 Insight:

* 307 customers churned
* Churn rate: **30.7%**

---

### 5. Immediate Churn After Trial

```sql
WITH ranked AS (
  SELECT
    customer_id,
    plan_id,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS rn
  FROM foodie_fi.subscriptions
)
SELECT
  COUNT(*) AS churned_after_trial,
  ROUND(
    100.0 * COUNT(*) /
    (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)
  ) AS percentage
FROM ranked
WHERE rn = 2 AND plan_id = 4;
```

📌 Insight:

* 92 customers churn immediately after trial (~9%)

---

### 6. Plan Conversion After Trial

```sql
WITH next_plan AS (
  SELECT
    customer_id,
    plan_id,
    LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_plan
  FROM foodie_fi.subscriptions
)
SELECT
  next_plan,
  COUNT(customer_id) AS customers,
  ROUND(
    100.0 * COUNT(customer_id) /
    (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),
  1) AS percentage
FROM next_plan
WHERE plan_id = 0
GROUP BY next_plan
ORDER BY next_plan;
```

📌 Insight:

* Majority convert to Basic or Pro plans
* Annual plan adoption is relatively low

---

### 7. Plan Distribution (End of 2020)

```sql
WITH latest_plan AS (
  SELECT
    customer_id,
    plan_id,
    start_date,
    LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_date
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
)
SELECT
  plan_id,
  COUNT(DISTINCT customer_id) AS customers,
  ROUND(
    100.0 * COUNT(DISTINCT customer_id) /
    (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),
  1) AS percentage
FROM latest_plan
WHERE next_date IS NULL
GROUP BY plan_id;
```

---

### 8. Annual Plan Upgrades in 2020

```sql
SELECT COUNT(DISTINCT customer_id) AS customers
FROM foodie_fi.subscriptions
WHERE plan_id = 3
  AND start_date <= '2020-12-31';
```

✅ **Result:** 196 customers

---

### 9. Average Time to Upgrade (Trial → Annual)

```sql
WITH trial AS (
  SELECT customer_id, start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
),
annual AS (
  SELECT customer_id, start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
)
SELECT
  ROUND(AVG(annual_date - trial_date), 0) AS avg_days
FROM trial
JOIN annual USING (customer_id);
```

📌 Insight:

* Average upgrade time: **~105 days**

---

### 10. Upgrade Time Distribution (30-Day Buckets)

```sql
WITH trial AS (
  SELECT customer_id, start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
),
annual AS (
  SELECT customer_id, start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
),
buckets AS (
  SELECT
    WIDTH_BUCKET(annual_date - trial_date, 0, 365, 12) AS bucket
  FROM trial
  JOIN annual USING (customer_id)
)
SELECT
  ((bucket - 1) * 30 || ' - ' || bucket * 30 || ' days') AS range,
  COUNT(*) AS customers
FROM buckets
GROUP BY bucket
ORDER BY bucket;
```

---

### 11. Downgrade Analysis (Pro → Basic)

```sql
WITH transitions AS (
  SELECT
    customer_id,
    plan_id,
    LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_plan
  FROM foodie_fi.subscriptions
  WHERE DATE_PART('year', start_date) = 2020
)
SELECT COUNT(*) AS downgrade_count
FROM transitions
WHERE plan_id = 2 AND next_plan = 1;
```

📌 Insight:

* No customers downgraded from Pro to Basic in 2020

---

## 📌 Key Takeaways

* Strong conversion from trial to paid plans
* High churn rate (~30%) → needs attention
* Most users prefer monthly subscriptions
* Annual plan adoption is relatively low → opportunity for optimization
* Customer upgrade journey averages ~3.5 months

---


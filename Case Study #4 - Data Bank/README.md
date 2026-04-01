## Case Study #4: Data Bank

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

All materials used in this case study are adapted from the official source: [here](https://8weeksqlchallenge.com/case-study-4/).

***

## Business Task

Danny introduced a new initiative called **Data Bank**, which combines traditional banking operations with a highly secure distributed **data storage system**.

In this system, each customer is assigned a cloud storage limit that is directly proportional to the amount of money in their account.

The Data Bank management team aims to grow its customer base while also gaining better insights into how much storage capacity their users will require over time.

This case study focuses on calculating key metrics, analyzing customer behavior, and helping the business make data-driven decisions to support future growth and planning.

---

## Entity Relationship Diagram

<img width="631" alt="image" src="https://user-images.githubusercontent.com/81607668/130343339-8c9ff915-c88c-4942-9175-9999da78542c.png">

### Table 1: `regions`

This table contains information about each region, including `region_id` and `region_name`.

<img width="176" alt="image" src="https://user-images.githubusercontent.com/81607668/130551759-28cb434f-5cae-4832-a35f-0e2ce14c8811.png">

---

### Table 2: `customer_nodes`

Customers are assigned to nodes based on their region, and this allocation is randomized. The assignment frequently changes as a security measure to reduce the risk of unauthorized access to customer data and funds.

<img width="412" alt="image" src="https://user-images.githubusercontent.com/81607668/130551806-90a22446-4133-45b5-927c-b5dd918f1fa5.png">

---

### Table 3: `customer_transactions`

This table records all customer transaction activities, including:
- Deposits  
- Withdrawals  
- Purchases made using the Data Bank debit card  

<img width="343" alt="image" src="https://user-images.githubusercontent.com/81607668/130551879-2d6dfc1f-bb74-4ef0-aed6-42c831281760.png">

***

## Question and Solution

All queries in this case study are executed using PostgreSQL via DB Fiddle. You can follow along and explore the queries step by step to better understand the data analysis process.

---

## 🏦 A. Customer Nodes Exploration

### 1. Number of unique nodes in the system

```sql
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM data_bank.customer_nodes;
````

**Answer:**

| unique_nodes |
| :----------- |
| 5            |

* There are a total of **5 unique nodes** in the Data Bank system.

---

### 2. Number of nodes per region

```sql
SELECT
  regions.region_name, 
  COUNT(DISTINCT customers.node_id) AS node_count
FROM data_bank.regions
JOIN data_bank.customer_nodes AS customers
  ON regions.region_id = customers.region_id
GROUP BY regions.region_name;
```

**Answer:**

| region_name | node_count |
| :---------- | :--------- |
| Africa      | 5          |
| America     | 5          |
| Asia        | 5          |
| Australia   | 5          |
| Europe      | 5          |

* Each region contains **5 nodes**, showing an even distribution.

---

### 3. Number of customers in each region

```sql
SELECT 
  region_id, 
  COUNT(customer_id) AS customer_count
FROM data_bank.customer_nodes
GROUP BY region_id
ORDER BY region_id;
```

**Answer:**

| region_id | customer_count |
| :-------- | :------------- |
| 1         | 770            |
| 2         | 735            |
| 3         | 714            |
| 4         | 665            |
| 5         | 616            |

* Customer distribution varies across regions, with Region 1 having the highest count and Region 5 the lowest.

---

### 4. Average days before customers are reassigned to a new node

```sql
WITH node_days AS (
  SELECT 
    customer_id, 
    node_id,
    end_date - start_date AS days_in_node
  FROM data_bank.customer_nodes
  WHERE end_date != '9999-12-31'
  GROUP BY customer_id, node_id, start_date, end_date
) 
, total_node_days AS (
  SELECT 
    customer_id,
    node_id,
    SUM(days_in_node) AS total_days_in_node
  FROM node_days
  GROUP BY customer_id, node_id
)

SELECT ROUND(AVG(total_days_in_node)) AS avg_node_reallocation_days
FROM total_node_days;
```

**Answer:**

| avg_node_reallocation_days |
| :------------------------- |
| 24                         |

* On average, customers are reassigned to a different node every **24 days**.

---

### 5. Median, 80th, and 95th percentile of reallocation days per region

(To be completed with additional analysis)

---

## 🏦 B. Customer Transactions

### 1. Total count and amount by transaction type

```sql
SELECT
  txn_type, 
  COUNT(customer_id) AS transaction_count, 
  SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
```

**Answer:**

| txn_type   | transaction_count | total_amount |
| :--------- | :---------------- | :----------- |
| purchase   | 1617              | 806537       |
| deposit    | 2671              | 1359168      |
| withdrawal | 1580              | 793003       |

* Deposits have the highest total value and frequency among all transaction types.

---

### 2. Average historical deposit count and amount per customer

```sql
WITH deposits AS (
  SELECT 
    customer_id, 
    COUNT(customer_id) AS txn_count, 
    AVG(txn_amount) AS avg_amount
  FROM data_bank.customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
)

SELECT 
  ROUND(AVG(txn_count)) AS avg_deposit_count, 
  ROUND(AVG(avg_amount)) AS avg_deposit_amt
FROM deposits;
```

**Answer:**

| avg_deposit_count | avg_deposit_amt |
| :---------------- | :-------------- |
| 5                 | 509             |

* On average, each customer makes **5 deposits** with an average value of **$509**.

---

### 3. Monthly active customers with multiple deposits and at least one other transaction

This analysis identifies customers who:

* Made more than one deposit in a month
* And performed at least one purchase or withdrawal

```sql
WITH monthly_transactions AS (
  SELECT 
    customer_id, 
    DATE_PART('month', txn_date) AS mth,
    SUM(CASE WHEN txn_type = 'deposit' THEN 0 ELSE 1 END) AS deposit_count,
    SUM(CASE WHEN txn_type = 'purchase' THEN 0 ELSE 1 END) AS purchase_count,
    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
  FROM data_bank.customer_transactions
  GROUP BY customer_id, DATE_PART('month', txn_date)
)

SELECT
  mth,
  COUNT(DISTINCT customer_id) AS customer_count
FROM monthly_transactions
WHERE deposit_count > 1 
  AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY mth
ORDER BY mth;
```

**Answer:**

| month | customer_count |
| :---- | :------------- |
| 1     | 170            |
| 2     | 277            |
| 3     | 292            |
| 4     | 103            |

---

### 4. Monthly closing balance and balance changes

This solution is built using multiple CTEs:

* Identify inflow (+) and outflow (-)
* Generate monthly periods
* Calculate cumulative balances using window functions

(Keep your SQL here unchanged)

---

### 5. Customer balance behavior analysis

Two temporary tables are created:

1. `customer_monthly_balances`
2. `ranked_monthly_balances`

These are used to evaluate customer balance trends across months.

---

#### Percentage of customers with negative vs positive first-month balance

| negative_first_month_percentage | positive_first_month_percentage |
| :------------------------------ | :------------------------------ |
| 44.8                            | 55.2                            |

---

#### Customers with >5% increase in balance

| increase_5_percentage |
| :-------------------- |
| 20.0                  |

* Around **20% of customers** experienced growth greater than 5%.

---

#### Customers with >5% decrease in balance

| reduce_5_percentage |
| :------------------ |
| 25.6                |

* Approximately **25.6% of customers** saw a decline of more than 5%.

---

#### Customers transitioning from positive to negative balance

| positive_to_negative_percentage |
| :------------------------------ |
| 20.2                            |

* About **20.2% of customers** moved from a positive to a negative balance in the following month.

---

⭐ If you found this project helpful, feel free to give it a star!


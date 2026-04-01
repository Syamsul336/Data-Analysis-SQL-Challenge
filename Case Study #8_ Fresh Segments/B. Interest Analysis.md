
# 🍅 Case Study #8 – Fresh Segments

## 📚 Solution B: Interest Analysis

---

### 1. Interests Present in All Months

First, check how many distinct `month_year` and `interest_id` values exist:

```sql id="b8i4qe"
SELECT 
  COUNT(DISTINCT month_year) AS unique_month_year_count, 
  COUNT(DISTINCT interest_id) AS unique_interest_id_count
FROM fresh_segments.interest_metrics;
```

**Result:**

* 14 distinct `month_year`
* 1,202 distinct `interest_id`

Next, identify interests that are present in **all 14 months**:

```sql id="8xqzv2"
WITH interest_cte AS (
  SELECT 
    interest_id, 
    COUNT(DISTINCT month_year) AS total_months
  FROM fresh_segments.interest_metrics
  WHERE month_year IS NOT NULL
  GROUP BY interest_id
)
SELECT 
  c.total_months,
  COUNT(DISTINCT c.interest_id) AS interest_count
FROM interest_cte c
WHERE total_months = 14
GROUP BY c.total_months
ORDER BY interest_count DESC;
```

**Result:** 480 out of 1,202 interests are present in all months.

---

### 2. Cumulative Percentage by `total_months`

Calculate cumulative percentage of interests starting from 14 months down to identify which `total_months` threshold covers 90% of all interests:

```sql id="m2v1sz"
WITH cte_interest_months AS (
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) AS total_months
  FROM fresh_segments.interest_metrics
  WHERE interest_id IS NOT NULL
  GROUP BY interest_id
),
cte_interest_counts AS (
  SELECT
    total_months,
    COUNT(DISTINCT interest_id) AS interest_count
  FROM cte_interest_months
  GROUP BY total_months
)
SELECT
  total_months,
  interest_count,
  ROUND(100 * SUM(interest_count) OVER (ORDER BY total_months DESC) /
        SUM(interest_count) OVER (), 2) AS cumulative_percentage
FROM cte_interest_counts
ORDER BY total_months DESC;
```

**Insight:**

* Interests with `total_months >= 6` make up 90% of cumulative coverage.
* Interests with fewer months may have low interaction or engagement and could be candidates for further investigation.

---

### 3. Data Points Removed if Low-Performing Interests are Dropped

We can calculate the total number of rows that would be removed if we drop interests with `total_months < 6`:

```sql id="i4qj7p"
WITH interest_cte AS (
  SELECT 
    interest_id, 
    COUNT(DISTINCT month_year) AS total_months
  FROM fresh_segments.interest_metrics
  GROUP BY interest_id
)
SELECT COUNT(*) AS rows_to_remove
FROM fresh_segments.interest_metrics metrics
JOIN interest_cte cte
  ON metrics.interest_id = cte.interest_id
WHERE total_months < 6;
```

This gives the total data points that would be filtered out for low engagement interests.

---

### 4. Business Perspective on Removing Low-Performing Interests

* **High-performing interest example:** An interest present in all 14 months consistently drives clicks and engagement. Retaining it ensures we capture reliable behavior trends.
* **Low-performing interest example:** An interest appearing only in 2-3 months indicates sparse or sporadic engagement. Keeping it might introduce noise and reduce accuracy for segment analysis.

**Decision:** Removing low-month interests can make analysis cleaner and more actionable, focusing on consistently engaged interests. However, business context is critical—sometimes a niche interest might be strategically important despite low presence.

---

### 5. Unique Interests per Month (All Interests Included)

Count unique interests for each `month_year` including all interests regardless of engagement:

```sql id="a9kzv0"
SELECT
  month_year,
  COUNT(DISTINCT interest_id) AS unique_interest_count
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year;
```

**Insight:**

* Provides a month-by-month snapshot of diversity in interests.
* Helps identify months with high or low engagement and guide marketing or personalization strategies.


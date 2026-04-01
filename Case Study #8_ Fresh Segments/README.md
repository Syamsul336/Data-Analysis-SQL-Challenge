# Case Study #8: Fresh Segments

## 📚 Table of Contents

* [Business Task](#business-task)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Questions and Solutions](#questions-and-solutions)

All information in this case study has been sourced from [8 Week SQL Challenge – Case Study 8](https://8weeksqlchallenge.com/case-study-8/).

---

## Business Task

Fresh Segments is a digital marketing agency that helps other businesses analyze trends in online ad click behavior for their customer base.

Clients provide their customer lists to Fresh Segments, who then aggregate interest metrics and generate a unified dataset for further analysis.

For each client, the dataset contains the composition and rankings for various interests, showing the proportion of their customers who interacted with online content related to each interest, per month.

Danny has asked for assistance in analyzing the aggregated metrics for an example client and providing high-level insights about their customers and interests.

---

## Entity Relationship Diagram

**Table: `interest_map`**

| id | interest_name             | interest_summary                                                                   | created_at              | last_modified           |
| -- | ------------------------- | ---------------------------------------------------------------------------------- | ----------------------- | ----------------------- |
| 1  | Fitness Enthusiasts       | Consumers using fitness tracking apps and websites.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 2  | Gamers                    | Consumers researching game reviews and cheat codes.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 3  | Car Enthusiasts           | Readers of automotive news and car reviews.                                        | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 4  | Luxury Retail Researchers | Consumers researching luxury product reviews and gift ideas.                       | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 5  | Brides & Wedding Planners | People researching wedding ideas and vendors.                                      | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 6  | Vacation Planners         | Consumers reading reviews of vacation destinations and accommodations.             | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 7  | Motorcycle Enthusiasts    | Readers of motorcycle news and reviews.                                            | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 8  | Business News Readers     | Readers of online business news content.                                           | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 12 | Thrift Store Shoppers     | Consumers shopping online for clothing at thrift stores and researching locations. | 2016-05-26T14:57:59.000 | 2018-03-16T13:14:00.000 |
| 13 | Advertising Professionals | People who read advertising industry news.                                         | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |

**Table: `interest_metrics`**

| month | year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
| ----- | ---- | ---------- | ----------- | ----------- | ----------- | ------- | ------------------ |
| 7     | 2018 | Jul-18     | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7     | 2018 | Jul-18     | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| 7     | 2018 | Jul-18     | 18923       | 10.85       | 5.29        | 3       | 99.59              |
| 7     | 2018 | Jul-18     | 6344        | 10.32       | 5.10        | 4       | 99.45              |
| 7     | 2018 | Jul-18     | 100         | 10.77       | 5.04        | 5       | 99.31              |
| 7     | 2018 | Jul-18     | 69          | 10.82       | 5.03        | 6       | 99.18              |
| 7     | 2018 | Jul-18     | 79          | 11.21       | 4.97        | 7       | 99.04              |
| 7     | 2018 | Jul-18     | 6111        | 10.71       | 4.83        | 8       | 98.90              |
| 7     | 2018 | Jul-18     | 6214        | 9.71        | 4.83        | 8       | 98.90              |
| 7     | 2018 | Jul-18     | 19422       | 10.11       | 4.81        | 10      | 98.63              |

---

## Questions and Solutions

You can execute these queries using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17).

---

## 🧼 A. Data Exploration and Cleaning

**1. Convert the `month_year` column in `fresh_segments.interest_metrics` to a `date` type (start of month).**

```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER month_year TYPE DATE USING month_year::DATE;
```

---

**2. Count records in `interest_metrics` per `month_year` sorted chronologically, with `NULL` first.**

```sql
SELECT 
  month_year, COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```

---

**3. Handling `NULL` values**

`NULL`s appear in `_month`, `_year`, `month_year`, and `interest_id`. Other columns like `composition` and `ranking` are meaningless without them.

Check the percentage of `NULL`s:

```sql
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 / COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics;
```

* Result: 8.36% `NULL`s → less than 10%, safe to remove:

```sql
DELETE FROM fresh_segments.interest_metrics
WHERE interest_id IS NULL;
```

Confirm removal:

```sql
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 / COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics;
```

---

**4. Count `interest_id`s missing in either table.**

```sql
SELECT 
  COUNT(DISTINCT map.id) AS map_id_count,
  COUNT(DISTINCT metrics.interest_id) AS metrics_id_count,
  SUM(CASE WHEN map.id IS NULL THEN 1 END) AS not_in_metric,
  SUM(CASE WHEN metrics.interest_id IS NULL THEN 1 END) AS not_in_map
FROM fresh_segments.interest_map map
FULL OUTER JOIN fresh_segments.interest_metrics metrics
  ON metrics.interest_id = map.id;
```

* `interest_metrics` all exist in `interest_map`.
* 7 `interest_map` IDs do not exist in `interest_metrics`.

---

**5. Summarize IDs by total record count in `interest_metrics`.**

```sql
SELECT 
  id, 
  interest_name, 
  COUNT(*)
FROM fresh_segments.interest_map map
JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
GROUP BY id, interest_name
ORDER BY count DESC, id;
```

---

**6. Join type for analysis**

We should use `INNER JOIN`:

```sql
SELECT *
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.interest_id = 21246   
  AND metrics._month IS NOT NULL;
```

---

**7. Check if any `month_year` is before `created_at`.**

```sql
SELECT COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < map.created_at::DATE;
```

* 188 records → same month, considered valid.

```sql
SELECT COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < DATE_TRUNC('mon', map.created_at::DATE);
```

---

## 📚 B. Interest Analysis

**1. Interests present in all `month_year`s:**

```sql
SELECT COUNT(DISTINCT month_year) AS unique_month_year_count, 
       COUNT(DISTINCT interest_id) AS unique_interest_id_count
FROM fresh_segments.interest_metrics;
```

* 14 distinct months, 1202 unique interests.

```sql
WITH interest_cte AS (
  SELECT interest_id, COUNT(DISTINCT month_year) AS total_months
  FROM fresh_segments.interest_metrics
  WHERE month_year IS NOT NULL
  GROUP BY interest_id
)
SELECT COUNT(DISTINCT interest_id)
FROM interest_cte
WHERE total_months = 14;
```

* 480 interests appear in all 14 months.

---

**2. Cumulative percentage of interests by `total_months`.**

```sql
WITH cte_interest_months AS (
  SELECT interest_id, COUNT(DISTINCT month_year) AS total_months
  FROM fresh_segments.interest_metrics
  WHERE interest_id IS NOT NULL
  GROUP BY interest_id
),
cte_interest_counts AS (
  SELECT total_months, COUNT(DISTINCT interest_id) AS interest_count
  FROM cte_interest_months
  GROUP BY total_months
)
SELECT total_months,
       interest_count,
       ROUND(100 * SUM(interest_count) OVER (ORDER BY total_months DESC) / SUM(interest_count) OVER (), 2) AS cumulative_percentage
FROM cte_interest_counts;
```

* Interests with ≥6 months cover 90% of the dataset → interests below 6 months should be reviewed.

---


## 📊 C. Segment Analysis

Fresh Segments wants to classify interests into **Top, Middle, and Bottom segments** per month based on their **ranking percentile**.

**1. Classify each interest per month into Top, Middle, Bottom segments:**

```sql
SELECT
    month_year,
    interest_id,
    ranking,
    percentile_ranking,
    CASE 
        WHEN percentile_ranking >= 90 THEN 'Top'
        WHEN percentile_ranking >= 50 THEN 'Middle'
        ELSE 'Bottom'
    END AS segment
FROM fresh_segments.interest_metrics
ORDER BY month_year, percentile_ranking DESC;
```

* **Top segment:** percentile ≥ 90
* **Middle segment:** 50 ≤ percentile < 90
* **Bottom segment:** percentile < 50

This allows the business to identify the most influential interests per month.

---

**2. Count interests per segment per month:**

```sql
SELECT
    month_year,
    segment,
    COUNT(*) AS interest_count
FROM (
    SELECT
        month_year,
        interest_id,
        CASE 
            WHEN percentile_ranking >= 90 THEN 'Top'
            WHEN percentile_ranking >= 50 THEN 'Middle'
            ELSE 'Bottom'
        END AS segment
    FROM fresh_segments.interest_metrics
) AS sub
GROUP BY month_year, segment
ORDER BY month_year, segment DESC;
```

* This shows how the distribution of Top, Middle, Bottom interests changes over time.

---

**3. Trend visualization idea:**

* **Line chart:** Number of Top, Middle, Bottom interests vs month_year
* Helps see if more interests are gaining popularity or declining.

---

## 📈 D. Index Analysis

Index values represent **interest strength relative to the average**. Analyzing these can reveal **which interests are growing or shrinking** in influence.

---

**1. Average index per interest across all months:**

```sql
SELECT
    interest_id,
    AVG(index_value) AS avg_index
FROM fresh_segments.interest_metrics
GROUP BY interest_id
ORDER BY avg_index DESC;
```

* Top average index indicates interests consistently above the client’s baseline engagement.

---

**2. Identify rapidly growing interests (latest month vs previous month):**

```sql
WITH monthly_index AS (
    SELECT
        interest_id,
        month_year,
        AVG(index_value) AS avg_index
    FROM fresh_segments.interest_metrics
    GROUP BY interest_id, month_year
),
growth AS (
    SELECT
        curr.interest_id,
        curr.month_year AS curr_month,
        curr.avg_index AS curr_index,
        prev.avg_index AS prev_index,
        curr.avg_index - prev.avg_index AS growth
    FROM monthly_index curr
    JOIN monthly_index prev
      ON curr.interest_id = prev.interest_id
     AND curr.month_year = prev.month_year + INTERVAL '1 month'
)
SELECT *
FROM growth
ORDER BY growth DESC
LIMIT 10;
```

* This identifies **top 10 interests with the fastest growth** in index value month-to-month.
* Useful for recommending content targeting or ads.

---

**3. Visual idea:**

* **Bar chart**: Top 10 growing interests and their growth values
* **Line chart**: Index over time for selected interests to see trends.

---

## 💡 Summary Insights

1. **Data Cleaning:**

   * 8% NULLs in `interest_metrics` → safely removed.
   * All `interest_metrics` IDs exist in `interest_map`.

2. **Interest Coverage:**

   * 480 interests appear in all 14 months → core stable interests.
   * Interests appearing less than 6 months cover only 10% of dataset → can be deprioritized.

3. **Segment Analysis:**

   * Interests in Top segment (percentile ≥90) are highly influential.
   * Distribution of segments over months shows trends in audience engagement.

4. **Index Analysis:**

   * Average index identifies consistently strong interests.
   * Growth analysis highlights emerging interests for marketing focus.

5. **Actionable Recommendations:**

   * Focus marketing on Top segment interests.
   * Monitor Middle segment to identify potential Top interests.
   * Consider Bottom segment as niche but valuable for exploratory campaigns.
   * Track index growth for timely campaign adjustments.

---
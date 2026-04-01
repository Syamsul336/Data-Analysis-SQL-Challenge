---

# 🍅 Case Study #8 – Fresh Segments

## 🧼 Solution A: Data Exploration and Cleansing

### 1. Convert `month_year` to Date Type

Update the `fresh_segments.interest_metrics` table to set the `month_year` column as a `DATE`, representing the first day of the month:

```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER month_year TYPE DATE USING month_year::DATE;
```

---

### 2. Count Records per `month_year`

Get the number of records for each `month_year`, sorted chronologically with `NULL` values first:

```sql
SELECT 
  month_year, COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```

---

### 3. Handling `NULL` Values

`NULL` values appear in `_month`, `_year`, `month_year`, and `interest_id`. These rows lack meaningful metrics in `composition`, `index_value`, `ranking`, and `percentile_ranking`.

Check the percentage of `NULL`s:

```sql
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 / COUNT(*)), 2) AS null_perc
FROM fresh_segments.interest_metrics;
```

**Result:** 8.36% of rows contain `NULL` values. Since this is less than 10%, it is reasonable to drop them:

```sql
DELETE FROM fresh_segments.interest_metrics
WHERE interest_id IS NULL;

-- Verify deletion
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 / COUNT(*)), 2) AS null_perc
FROM fresh_segments.interest_metrics;
```

All `NULL` values have been removed.

---

### 4. Compare `interest_id` Between Tables

Find `interest_id`s in `interest_metrics` but not in `interest_map` and vice versa:

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

**Summary:**

* `interest_map`: 1,209 unique IDs
* `interest_metrics`: 1,202 unique IDs
* All `interest_metrics` IDs exist in `interest_map`
* 7 IDs in `interest_map` are missing from `interest_metrics`

---

### 5. Summarize `interest_map` IDs by Record Count

Original solution just counted total rows:

```sql
SELECT COUNT(*) FROM fresh_segments.interest_map;
```

A more meaningful summary joins metrics:

```sql
SELECT 
  id, 
  interest_name, 
  COUNT(*) AS record_count
FROM fresh_segments.interest_map map
JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
GROUP BY id, interest_name
ORDER BY record_count DESC, id;
```

---

### 6. Determine Join Type for Analysis

Use an `INNER JOIN` to combine `interest_map` and `interest_metrics`:

```sql
SELECT *
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.interest_id = 21246
  AND metrics._month IS NOT NULL;
```

**Result:** 10 rows for `interest_id = 21246`. This ensures we only analyze rows with complete date information.

---

### 7. Validate `month_year` vs `created_at`

Check for records where `month_year` is before the `created_at` date:

```sql
SELECT COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < map.created_at::DATE;
```

**Result:** 188 rows.

Check if they are in the same month:

```sql
SELECT COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < DATE_TRUNC('mon', map.created_at::DATE);
```

All records are in the same month, so we consider them valid.

---

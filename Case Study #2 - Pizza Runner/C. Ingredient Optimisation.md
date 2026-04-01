# 🍕 Case Study #2: Pizza Runner (SQL Analysis)

## 📚 Section C: Ingredient Optimisation

In this section, the analysis focuses on pizza ingredients, including standard recipes, customer modifications (extras & exclusions), and overall ingredient usage.

---

### 1. Standard Ingredients for Each Pizza

Each pizza has a predefined list of ingredients stored in the `pizza_recipes` table.  
These ingredients are represented as comma-separated `topping_id` values, which can be mapped to their corresponding names using the `pizza_toppings` table.

➡️ This structure allows flexibility in customizing pizzas while maintaining a standard base recipe.

---

### 2. Most Commonly Added Extra

```sql
WITH toppings_cte AS (
  SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.pizza_recipes
)

SELECT 
  t.topping_id,
  pt.topping_name, 
  COUNT(t.topping_id) AS topping_count
FROM toppings_cte t
JOIN pizza_runner.pizza_toppings pt
  ON t.topping_id = pt.topping_id
GROUP BY t.topping_id, pt.topping_name
ORDER BY topping_count DESC;
````

**Insight:**

* The most frequently added extra topping is the one with the highest `topping_count`.

➡️ This highlights customer preferences for additional ingredients and can guide upselling strategies.

---

### 3. Most Commonly Excluded Ingredient

To identify the most commonly removed ingredient, we need to:

* Extract exclusion values from the `customer_orders` table
* Split multiple exclusions into individual topping IDs
* Count their frequency

➡️ This helps identify ingredients that customers tend to avoid.

---

### 4. Generate Order Item Descriptions

Each order can be transformed into a more readable format such as:

* `Meat Lovers`
* `Meat Lovers - Exclude Beef`
* `Meat Lovers - Extra Bacon`
* `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

➡️ This improves clarity for both customers and operational teams.

---

### 5–6. Ingredient List per Order (Formatted Output)

For each order, we can:

* Combine all ingredients into a single list
* Sort them alphabetically
* Add prefix `2x` for duplicated ingredients (extras)

Example output:

```
Meat Lovers: 2xBacon, Beef, Cheese, Chicken, Salami
```

➡️ This representation makes it easier to track ingredient usage and order customization.

---

### 7. Total Ingredient Usage Across Delivered Orders

To calculate total ingredient usage:

* Filter only successfully delivered orders
* Combine base ingredients with extras
* Remove excluded ingredients
* Aggregate total usage per ingredient

➡️ This provides valuable insights for:

* Inventory planning
* Supply chain optimization
* Cost control

---

## 🎯 Key Takeaways

* Ingredient customization is a key part of customer behavior
* Certain toppings are consistently preferred as extras
* Some ingredients are frequently excluded, indicating potential dislike
* Ingredient-level analysis supports better inventory and menu decisions

---

## 🚀 Skills Demonstrated

* String manipulation in SQL (REGEXP_SPLIT_TO_TABLE)
* Data transformation from denormalized formats
* JOIN operations across multiple tables
* Aggregation and frequency analysis
* Handling customer customization logic (extras & exclusions)

---

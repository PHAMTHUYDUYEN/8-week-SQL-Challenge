# 🥓 C. Ingredient Optimization (WIP)

## Case Study Questions

1. What are the standard ingredients for each pizza?
2. What was the most commonly added extra?
3. What was the most common exclusion?
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
  - Meat Lovers
  - Meat Lovers - Exclude Beef
  - Meat Lovers - Extra Bacon
  - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

***

### 1. What are the standard ingredients for each pizza?
```SQL
WITH splt_pz AS(
  SELECT
      pizza_id,
      UNNEST(string_to_array(toppings, ','))::INT AS topping_id
  FROM pizza_recipes)

SELECT
    n.pizza_name,
    STRING_AGG(t.topping_name,', ') AS standard_ingredients
FROM splt_pz AS p
    JOIN pizza_toppings AS t   USING(topping_id)
    JOIN pizza_names AS n   USING(pizza_id)
GROUP BY 1
```
| pizza_name | standard_ingredients                                           |
| ---------- | -------------------------------------------------------------- |
| Meatlovers | BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef |
| Vegetarian | Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes          |
---
### 2. What was the most commonly added extra?
```SQL
SELECT
    pizza_toppings.topping_name,
    COUNT(union_extra.pizza_id) AS total_add
FROM (
      SELECT pizza_id, extra1 AS extra FROM cln_co
      UNION ALL
      SELECT pizza_id, extra2 AS extra FROM cln_co) AS union_extra
  JOIN pizza_toppings ON union_extra.extra = pizza_toppings.topping_id
WHERE extra IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
```
| topping_name | total_add |
| ------------ | --------- |
| Bacon        | 4         |
| Chicken      | 1         |
| Cheese       | 1         |
---
### 3. What was the most common exclusion?
```SQL
SELECT
    pizza_toppings.topping_name,
    COUNT(union_exclusion.pizza_id) AS total_exclusion
FROM (
      SELECT pizza_id, exclusion1 AS exclusion FROM cln_co
      UNION ALL
      SELECT pizza_id, exclusion2 AS exclusion FROM cln_co) AS union_exclusion
  JOIN pizza_toppings ON union_exclusion.exclusion = pizza_toppings.topping_id
WHERE exclusion IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
```
| topping_name | total_exclusion |
| ------------ | --------------- |
| Cheese       | 4               |
| Mushrooms    | 1               |
| BBQ Sauce    | 1               |

---
### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```SQL
WITH t_details AS (
    SELECT 
        o.order_id,
        p.pizza_name,
        e1.topping_name AS exclusions,
        e2.topping_name AS exclusions2,
        ex1.topping_name AS extras,
        ex2.topping_name AS extras2
    FROM cln_co AS o
    LEFT JOIN pizza_names AS p ON o.pizza_id = p.pizza_id
    LEFT JOIN pizza_toppings AS e1 ON o.exclusion1 = e1.topping_id
    LEFT JOIN pizza_toppings e2 ON o.exclusion2 = e2.topping_id
    LEFT JOIN pizza_toppings ex1 ON o.extra1 = ex1.topping_id
    LEFT JOIN pizza_toppings ex2 ON o.extra2 = ex2.topping_id
    GROUP BY 1,2,3,4,5,6
)

SELECT 
    order_id,
    pizza_name ||
    CASE 
        WHEN exclusions IS NOT NULL OR exclusions2 IS NOT NULL 
        THEN ' - Exclude ' || COALESCE(exclusions, '') || 
            CASE 
                WHEN exclusions IS NOT NULL AND exclusions2 IS NOT NULL 
                THEN ', ' 
                ELSE '' 
            END || COALESCE(exclusions2, '') 
        ELSE ''
    END ||
    CASE 
        WHEN extras IS NOT NULL OR extras2 IS NOT NULL 
        THEN ' - Extra ' || COALESCE(extras, '') || 
            CASE 
                WHEN extras IS NOT NULL AND extras2 IS NOT NULL 
                THEN ', ' 
                ELSE '' 
            END || COALESCE(extras2, '') 
        ELSE ''
    END AS order_description
FROM t_details;
```
| order_id | order_description                                               |
| -------- | --------------------------------------------------------------- |
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

---
### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

_(WIP)_

---

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

_(WIP)_

---
## See other Solutions
🍕[A. Pizza Metrics](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/A.%20PIZZA%20METRICS.md)\
🏃[B. Runner and Customer Experience](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/B.%20RUNNER%20AND%20CUSTOMERS%20EXPERIENCE.md)\
💰[D. Pricing and Ratings](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/D.%20PRICING%20AND%20RATINGS.md)\
✨[E. Bonus Questions](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/E.%20BONUS%20QUESTIONS.md)

---
👉 Get back to [**8-Week-SQL-Challenge Repository**](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge) to see other Challenges!

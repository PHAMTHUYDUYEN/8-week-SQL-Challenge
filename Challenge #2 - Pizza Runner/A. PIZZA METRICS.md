# :pizza: A. Pizza Metrics

## Case Study Questions

1. How many pizzas were ordered?
2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?
---
###  1. How many pizzas were ordered?
```SQL
 SELECT COUNT(pizza_id) AS total_ordered_pizza
 FROM cln_co
```
| total_ordered_pizza |
| ------------------- |
| 14                  |
---
###  2. How many unique customer orders were made?
```SQL
SELECT 
    COUNT(DISTINCT order_id) AS unique_orders
FROM cln_co
```
| unique_orders |
| ------------- |
| 10            |

---
###  3. How many successful orders were delivered by each runner?
```SQL
SELECT 
	runner_id,
	COUNT(DISTINCT order_id) AS total_delivered_orders
FROM cln_ro
WHERE cancellation IS NULL
GROUP BY runner_id
```
| runner_id | total_delivered_orders |
| --------- | ---------------------- |
| 1         | 4                      |
| 2         | 3                      |
| 3         | 1                      |

---
###  4. How many of each type of pizza was delivered?
```SQL
SELECT 
	c.pizza_id,
	COUNT(order_id) AS total_delivered_pizzas
FROM cln_co AS c JOIN cln_ro AS r USING(order_id)
WHERE cancellation IS NULL
GROUP BY c.pizza_id
```
| pizza_name | total_delivered_pizzas |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |

---
###  5. How many Vegetarian and Meatlovers were ordered by each customer?
```SQL
SELECT 
    p.pizza_name,
    COUNT(c.order_id)
FROM cln_co AS c JOIN pizza_names AS p USING(pizza_id)
GROUP BY 1
```
| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 10    |
| Vegetarian | 4     |

---
###  6. What was the maximum number of pizzas delivered in a single order?
```SQL
SELECT 
    order_id,
    COUNT(c.pizza_id) AS total_delivered_pizzas
FROM cln_co AS c JOIN cln_ro AS r USING(order_id)
WHERE r.cancellation IS NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
```
| order_id | total_delivered_pizzas |
| -------- | ---------------------- |
| 4        | 3                      |

---
###  7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```SQL
SELECT 
    c.customer_id,
    COUNT(*) FILTER(WHERE c.exclusion1 IS NULL AND c.exclusion2 IS NULL AND c.extra1 IS NULL AND c.extra2 IS NULL) AS no_change_pizzas,
    COUNT(*) FILTER(WHERE c.exclusion1 IS NOT NULL OR c.exclusion2 IS NOT NULL OR c.extra1 IS NOT NULL OR c.extra2 IS NOT NULL) AS pizzas_with_changes
FROM cln_co AS c JOIN cln_ro AS r USING(order_id)
WHERE r.cancellation IS NULL
GROUP BY 1
```
| customer_id | no_change_pizzas | pizzas_with_changes |
| ----------- | ---------------- | ------------------- |
| 101         | 2                | 0                   |
| 102         | 3                | 0                   |
| 105         | 0                | 1                   |
| 104         | 1                | 2                   |
| 103         | 0                | 3                   |

---
###  8. How many pizzas were delivered that had both exclusions and extras?
```SQL
SELECT 
    COUNT(*) FILTER(WHERE (c.exclusion1 IS NOT NULL OR c.exclusion2 IS NOT NULL) AND (c.extra1 IS NOT NULL OR c.extra2 IS NOT NULL)) AS both_change_pizzas
FROM cln_co AS c JOIN cln_ro AS r USING(order_id)
WHERE r.cancellation IS NULL
```
| both_change_pizzas |
| ------------------ |
| 1                  |

---
###  9. What was the total volume of pizzas ordered for each hour of the day?
```SQL
SELECT 
    EXTRACT(hour FROM order_time) AS hour,
    COUNT(pizza_id) AS total_ordered_pizzas
FROM cln_co 
GROUP BY 1
```
| hour | total_ordered_pizzas |
| ---- | -------------------- |
| 18   | 3                    |
| 23   | 3                    |
| 21   | 3                    |
| 11   | 1                    |
| 19   | 1                    |
| 13   | 3                    |

---
###  10. What was the volume of orders for each day of the week?
```SQL
SELECT 
    TO_CHAR(order_time, 'YYYY-MM') AS month,
    EXTRACT(week FROM order_time) AS week,
    EXTRACT (day FROM order_time) AS date,
    TO_CHAR(order_time, 'Day') AS day_name,
    COUNT(pizza_id) AS total_ordered_pizzas
FROM cln_co
GROUP BY 1,2,3,4
ORDER BY 1,2,3,4
```
| month   | week | date | day_name  | total_ordered_pizzas |
| ------- | ---- | ---- | --------- | -------------------- |
| 2020-01 | 1    | 1    | Wednesday | 2                    |
| 2020-01 | 1    | 2    | Thursday  | 2                    |
| 2020-01 | 1    | 4    | Saturday  | 3                    |
| 2020-01 | 2    | 8    | Wednesday | 3                    |
| 2020-01 | 2    | 9    | Thursday  | 1                    |
| 2020-01 | 2    | 10   | Friday    | 1                    |
| 2020-01 | 2    | 11   | Saturday  | 2                    |

---
## See other Solutions
🏃[B. Runner and Customer Experience](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/B.%20RUNNER%20AND%20CUSTOMERS%20EXPERIENCE.md)\
🥓[C. Ingredient Optimization](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/C.%20INGREDIENT%20OPTIMIZATION.md)\
💰[D. Pricing and Ratings](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/D.%20PRICING%20AND%20RATINGS.md)\
✨[E. Bonus Questions](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/E.%20BONUS%20QUESTIONS.md)

---
👉 Get back to [**8-Week-SQL-Challenge Repository**](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge) to see other Challenges!

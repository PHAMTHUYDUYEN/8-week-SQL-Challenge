# 🏃 B. Runner & Customer Experience

## Case Study Questions

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each customer?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?

---
###  1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```SQL
SELECT
    TO_CHAR(registration_date, 'WW') AS week,
    COUNT(runner_id) AS total_registered_runners
FROM runners
GROUP BY 1
ORDER BY 1
```
| week | total_registered_runners |
| ---- | ------------------------ |
| 01   | 2                        |
| 02   | 1                        |
| 03   | 1                        |

---
###  2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```SQL
WITH unique_ord AS(
	SELECT
      order_id,
      customer_id,
      MAX(order_time) AS order_time
	FROM cln_co
	GROUP BY 1,2)

SELECT
    r.runner_id,
    AVG(DATE_PART('min', r.pickup_time - c.order_time)) AS avg_pickup_duration    
FROM cln_ro AS r 
	JOIN unique_ord AS c 
  USING(order_id)
WHERE r.cancellation IS NULL
GROUP BY 1
ORDER BY 1
```
| runner_id | avg_pickup_duration |
| --------- | ------------------- |
| 1         | 14                  |
| 2         | 19.666666666666668  |
| 3         | 10                  |

---
###  3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```SQL
 WITH raw_data AS(    
   SELECT
      order_id,
      COUNT(c.pizza_id) AS total_pizzas,
      AVG(EXTRACT(min FROM (r.pickup_time - c.order_time))) AS prep_time
   FROM cln_ro AS r 
      JOIN cln_co AS c USING(order_id)
   WHERE r.cancellation IS NULL
   GROUP BY 1
   ORDER BY 1)
 
SELECT 
    CORR(total_pizzas, prep_time) AS correlation
FROM raw_data
```
| correlation        |
| ------------------ |
| 0.8372508882083807 |

- The figure 0.837 indicates a strong correlation: the number of pizzas in an order significantly affects the preparation time.
- However, other factors may also impact preparation time, such as kitchen productivity or the type of pizza.
- If Danny's restaurant wants to reduce waiting time, it could consider optimizing the production process for orders with more pizzas (breaking down the process, assigning each person a specific task; forecasting peak orders, improving employee skills, and investing in technology and equipment).

---
###  4. What was the average distance travelled for each customer?
```SQL
 SELECT
    c.customer_id,
    ROUND(AVG(r.distance_km),2) AS avg_distance_km
 FROM cln_ro AS r 
    JOIN cln_co AS c USING(order_id)
 WHERE r.cancellation IS NULL
 GROUP BY 1
 ORDER BY 1
```
| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20.00           |
| 102         | 16.73           |
| 103         | 23.40           |
| 104         | 10.00           |
| 105         | 25.00           |
---
###  5. What was the difference between the longest and shortest delivery times for all orders?
```SQL
 SELECT
	MAX(duration_mins) - MIN(duration_mins) AS duration_diff
 FROM cln_ro 
 WHERE cancellation IS NULL
```
| duration_diff |
| ------------- |
| 30            |

---
###  6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```SQL
 SELECT
    order_id,
    runner_id,
    ROUND(distance_km/NULLIF((duration_mins/60),0),2) AS avg_speed
 FROM cln_ro 
 WHERE cancellation IS NULL
 ORDER BY 2,1 ASC
```
| order_id | runner_id | avg_speed |
| -------- | --------- | --------- |
| 1        | 1         | 37.50     |
| 2        | 1         | 44.44     |
| 3        | 1         | 40.20     |
| 10       | 1         | 60.00     |
| 4        | 2         | 35.10     |
| 7        | 2         | 60.00     |
| 8        | 2         | 93.60     |
| 5        | 3         | 40.00     |

- Runners are getting faster over time.
  - Runner 1: 37.50 → 44.44 → 40.20 → 60.00 (increasing trend)
  - Runner 2: 35.10 → 60.00 → 93.60 (significant increase)

- Runner 2's significantly higher maximum speed (93 km/h, while the other two runners have speeds below 60 km/h), this could indicate:
  - A data entry issue for order 8
  - A scenario where the runner used a car instead of a bicycle
  - A very favorable condition (long straight road, no stops, etc.)
    
---
###  7. What is the successful delivery percentage for each runner?
```SQL
 SELECT
    runner_id,
    ROUND(COUNT(duration_mins)*100 / COUNT(order_id),2) AS successful_delivery_percentage
 FROM cln_ro 
 GROUP BY 1
```
| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 3         | 50.00                          |
| 2         | 75.00                          |
| 1         | 100.00                         |

**Note**: The successful delivery percentage did not fully represent the runners' performance due to cancellation orders, which are made by either customers or restaurant.

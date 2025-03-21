# 💰 D. Pricing and Ratings

## Case Study Questions

1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

***

###  1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```SQL
SELECT
  SUM(CASE
    WHEN pizza_id = 1 THEN 12
    WHEN pizza_id = 2 THEN 10
    END) AS total_revenue
    
FROM cln_co
```
| total_revenue |
| --------- |
| 160       |
---
###  2.1. What if there was an additional $1 charge for any pizza extras? 
```SQL
SELECT
  SUM(CASE
    WHEN pizza_id = 1 THEN 12
    WHEN pizza_id = 2 THEN 10
    END
    + COALESCE((extra1 IS NOT NULL)::INT,0)
    + COALESCE((extra2 IS NOT NULL)::INT,0)
  ) AS total_revenue
    
FROM cln_co
```
| total_revenue |
| --------- |
| 166       |
--- 
### 2.2. What if there was an additional $1 charge for cheese extra?
```SQL
SELECT
  SUM(CASE
    WHEN pizza_id = 1 THEN 12
    WHEN pizza_id = 2 THEN 10
    END
    + COALESCE((extra1 = 4)::INT,0)
    + COALESCE((extra2 = 4)::INT,0)
  ) AS total_revenue
    
FROM cln_co
```
| total_revenue |
| --------- |
| 161       |

---
###  3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```SQL
ROP TABLE IF EXISTS runner_rating;

CREATE TABLE runner_rating ("order_id" INTEGER, "rating" INTEGER, "description" TEXT);

INSERT INTO runner_rating
	("order_id","rating","description")
VALUES (1, 1, 'Very Poor'),
       (2, 3, 'Poor'),
       (3, 4, 'Great'),
       (4, 3,'OK' ),
       (5, 2, 'Poor'),
       (7, 5, 'Excellent'),
       (8, 4, 'Great'),
       (10, 5, 'Excellent');

SELECT *
FROM runner_rating
```
| order_id | rating | description |
| -------- | ------ | ----------- |
| 1        | 1      | Very Poor   |
| 2        | 3      | Poor        |
| 3        | 4      | Great       |
| 4        | 3      | OK          |
| 5        | 2      | Poor        |
| 7        | 5      | Excellent   |
| 8        | 4      | Great       |
| 10       | 5      | Excellent   |
---
###  4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

```SQL
SELECT
    co.customer_id,
    order_id,
    ro.runner_id,
    rr.rating,
    co.order_time,
    ro.pickup_time,
    DATE_PART('min', ro.pickup_time - co.order_time) AS order_pickup_diff,
    ro.duration_mins,
    ROUND(ro.distance_km/NULLIF((ro.duration_mins/60),0),2) AS avg_speed,
    COUNT(pizza_id) AS total_pizzas
    
FROM runner_rating AS rr
	JOIN cln_co AS co USING(order_id)
    JOIN cln_ro AS ro USING(order_id)

GROUP BY 1,2,3,4,5,6,7,8,9
```
| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | order_pickup_diff | duration_mins | avg_speed | total_pizzas |
| ----------- | -------- | --------- | ------ | ------------------- | ------------------- | ----------------- | ------------- | --------- | ------------ |
| 101         | 1        | 1         | 1      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 10                | 32            | 37.50     | 1            |
| 101         | 2        | 1         | 3      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10                | 27            | 44.44     | 1            |
| 102         | 3        | 1         | 4      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21                | 20            | 40.20     | 2            |
| 102         | 8        | 2         | 4      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20                | 15            | 93.60     | 1            |
| 103         | 4        | 2         | 3      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29                | 40            | 35.10     | 3            |
| 104         | 5        | 3         | 2      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10                | 15            | 40.00     | 1            |
| 104         | 10       | 1         | 5      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 15                | 10            | 60.00     | 2            |
| 105         | 7        | 2         | 5      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10                | 25            | 60.00     | 1            |

---
###  5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```SQL
SELECT 
	SUM(revenue) AS total_revenue
FROM(
    SELECT
        order_id,
        ro.distance_km,
        SUM(CASE
                WHEN pizza_id = 1 THEN 12
                WHEN pizza_id = 2 THEN 10
            END) 
            - (ro.distance_km*0.3) AS revenue    
    FROM cln_co AS co JOIN cln_ro AS ro USING(order_id)
    WHERE ro.cancellation IS NULL
    GROUP BY 1,2
	) AS rev_by_ord
```
| total_revenue |
| ------------- |
| 94.44         |

---
👉 Get back to [**8-Week-SQL-Challenge Repository**](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge)

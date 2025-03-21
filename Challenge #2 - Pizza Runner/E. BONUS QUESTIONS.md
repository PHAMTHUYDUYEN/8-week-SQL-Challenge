# ✨ E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

---
### Insert new type of pizza into pizza_names
```SQL
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (3, 'Supreme');
  
SELECT * FROM pizza_names
```
| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |
| 3        | Supreme    |
---
### Insert Supreme's recipe into pizza_recipes
```SQL
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (3, (SELECT STRING_AGG(topping_id::TEXT, ', ') FROM pizza_toppings));
  
SELECT * FROM pizza_recipes
```
| pizza_id | toppings                              |
| -------- | ------------------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10               |
| 2        | 4, 6, 7, 9, 11, 12                    |
| 3        | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 |
---

### Our new menu!
```SQL
WITH splt_pz AS(
  SELECT
      pizza_id,
      UNNEST(string_to_array(toppings, ','))::INT AS topping_id
  FROM pizza_recipes)

SELECT
    n.pizza_name,
    STRING_AGG(t.topping_name,',') AS standard_ingredients
FROM splt_pz AS p
	JOIN pizza_toppings AS t  USING(topping_id)
  JOIN pizza_names AS n  USING(pizza_id)
GROUP BY 1
```
| pizza_name | standard_ingredients                                                                                           |
| ---------- | -------------------------------------------------------------------------------------------------------------- |
| Supreme    | Onions, Tomatoes, Beef, Pepperoni, Tomato Sauce, Peppers, BBQ Sauce, Salami, Chicken, Mushrooms, Cheese, Bacon |
| Meatlovers | Mushrooms, Chicken, Salami, BBQ Sauce, Beef, Pepperoni, Bacon, Cheese                                          |
| Vegetarian | Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes                                                     |

---
👉 Get back to [**8-Week-SQL-Challenge Repository**](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge)

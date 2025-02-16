# :ramen: Solution to Challenge #1: Danny's Diner 

###  1. What is the total amount each customer spent at the restaurant?
```SQL
SELECT
   s.customer_id,
   SUM(m.price) AS total_spend
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.menu AS m
   USING(product_id)
GROUP BY customer_id;
```
Result:

| customer_id | total_spend |
| ----------- | ----------- |
| B           | 74          |
| C           | 36          |
| A           | 76          |

- Customer A spent $76
- Customer B spent $74
- Customer C spent $36

---
###  2. How many days has each customer visited the restaurant?
```SQL
SELECT
   customer_id,
   COUNT(DISTINCT order_date) AS total_visit
FROM dannys_diner.sales
GROUP BY customer_id;
```
Result:

| customer_id | total_visit |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

- Customer A visited 4 days
- Customer B visited 6 days
- Customer C visited 2 days

---
###  3. What was the first item from the menu purchased by each customer?
```SQL
WITH first_date AS(
  SELECT
     customer_id,
     MIN(order_date) AS order_date
  FROM dannys_diner.sales
  GROUP BY 1
)

SELECT 
   s.customer_id,
   s.order_date,
   m.product_name
FROM dannys_diner.sales AS s
  LEFT JOIN dannys_diner.menu AS m
   USING(product_id)
  INNER JOIN first_date AS f
   ON s.customer_id = f.customer_id 
   AND s.order_date = f.order_date
ORDER BY 1;
```
Result:

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |
| C           | 2021-01-01 | ramen        |

- Customer A's first purchase were sushi and curry first simultaneously
- Customer B chose curry as his/her first dish
- Customer C seemed to be attracted by ramen

---
###  4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```SQL
SELECT
   product_id,
   m.product_name,
   COUNT(s.order_date) AS purchase_count
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.menu AS m
   USING(product_id)
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;
```
Result:

| product_id | product_name | purchase_count |
| ---------- | ------------ | -------------- |
| 3          | ramen        | 8              |

- Ramen is Danny restaurant's best-seller with 8 orders.

---
###  5. Which item was the most popular for each customer?
```SQL
WITH ranking AS (
    SELECT
        s.customer_id,
        m.product_name,
        COUNT(s.order_date) AS purchase_count,
        DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.order_date) DESC) AS rank
    FROM dannys_diner.sales AS s
    INNER JOIN dannys_diner.menu AS m USING(product_id)
    GROUP BY s.customer_id, m.product_name
)

SELECT 
    customer_id,
    product_name,
    purchase_count
FROM ranking
WHERE rank = 1;
```
Result:

| customer_id | product_name | purchase_count |
| ----------- | ------------ | -------------- |
| A           | ramen        | 3              |
| B           | curry        | 2              |
| B           | ramen        | 2              |
| B           | sushi        | 2              |
| C           | ramen        | 3              |

- Both Customer A and C prefer ramen while Customer B favors all menu!

---
###  6. Which item was purchased first by the customer after they became a member?
```SQL
WITH first_date AS(
SELECT 
   customer_id,
    MIN(order_date) AS first_date
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.members AS mem
    USING(customer_id)
WHERE s.order_date > mem.join_date
GROUP BY 1
)

SELECT 
   s.customer_id,
   s.order_date,
   m.product_name
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m
   USING(product_id)
INNER JOIN first_date AS f
   ON s.customer_id = f.customer_id 
   AND s.order_date = f.first_date
ORDER BY 1;
```
Result:

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-10 | ramen        |
| B           | 2021-01-11 | sushi        |

- Customer A bought ramen first, after joining loyalty program
- Customer B purchased sushi right after being a member

---
###  7. Which item was purchased just before the customer became a member?
```SQL
WITH last_list AS(
SELECT 
   customer_id,
    MAX(order_date) AS last_date
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.members AS mem
    USING(customer_id)
WHERE s.order_date < mem.join_date
GROUP BY 1
)

SELECT 
   s.customer_id,
   s.order_date,
   m.product_name
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m
   USING(product_id)
INNER JOIN last_list AS l
   ON s.customer_id = l.customer_id 
   AND s.order_date = l.last_date
ORDER BY 1;
```
Result:

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |

- Customer A bought sushi and curry just before joining loyalty program
- Customer B purchased sushi before being a member, seems like a big fan of sushi 🍣!

---
###  8. What is the total items and amount spent for each member before they became a member?
```SQL
SELECT 
   s.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.members AS mem
    USING(customer_id)
    LEFT JOIN dannys_diner.menu AS m
    ON s.product_id = m.product_id
WHERE s.order_date < mem.join_date
GROUP BY 1;
```
Result:

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| B           | 3           | 40          |
| A           | 2           | 25          |

- Before joining loyalty program, Customer A spent on 2 items with $25
- Customer B spent $40 on 3 items  

---
###  9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```SQL
SELECT
   s.customer_id,
    SUM(CASE 
         WHEN m.product_name = 'sushi' THEN m.price*20
         ELSE m.price*10
      END)
FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.menu AS m 
    USING(product_id)
GROUP BY 1
ORDER BY 1;
```
Result:

| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Customer A: 860 pts
- Customer B: 940 pts
- Customer C: 360 pts

---
###  10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```SQL
SELECT
   s.customer_id,
    SUM(CASE
         WHEN s.order_date >= mem.join_date AND s.order_date < mem.join_date + 7 THEN price*20
         ELSE price*0
      END) AS total_points
FROM
   dannys_diner.sales AS s
    LEFT JOIN dannys_diner.menu AS m USING(product_id)
    LEFT JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
WHERE s.order_date >= mem.join_date
   AND s.order_date BETWEEN '2021-01-01' AND '2021-01-31'
GROUP BY 1
ORDER BY 1;
```
Result:

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1020         |
| B           | 200          |

- Customer A: 1020 pts
- Customer B: 200 pts
- Customer C: not join yet

---
### Bonus Question 1 - Join All The Things
**Creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.**
```SQL
SELECT
   s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
      WHEN s.order_date >= mem.join_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM
   dannys_diner.sales AS s
    LEFT JOIN dannys_diner.menu AS m USING(product_id)
    LEFT JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
ORDER BY 1,2,3;
```
Result:

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

---
### Bonus Question 2 - Rank All The Things
**Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

```SQL
WITH all_data AS(
SELECT
      s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
         WHEN s.order_date >= mem.join_date THEN 'Y'
        ELSE 'N'
      END AS member     
FROM
    dannys_diner.sales AS s
    LEFT JOIN dannys_diner.menu AS m USING(product_id)
    LEFT JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
)

SELECT
   *,
    CASE
      WHEN member = 'Y' THEN DENSE_RANK() OVER(PARTITION BY member,customer_id ORDER BY order_date ASC)
        ELSE null
      END AS ranking
FROM all_data
ORDER BY 1,2,3;
```
Result:

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |

---
👉 Click [here](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge) to get back to the 8-Week-SQL-Challenge repository!


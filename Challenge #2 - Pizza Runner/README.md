# 🍕 Challenge #2: Pizza Runner
<p align="center">
<img src="https://8weeksqlchallenge.com/images/case-study-designs/2.png" alt="Image" width="450" height="450">

If you want to give **Challenge #1** a try, click [here](https://8weeksqlchallenge.com/case-study-2/)

## Introduction
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Datasets 

| No. | Table  | Description |
|----|---------|------------|
| 1  | customer_orders   | Information of orders |
| 2  | runner_orders    | Information of orders delivered by runners |
| 3  | runners | Information of runners |
| 4  | pizza_names | Name of pizzas served at Danny's restaurant |
| 5  | pizza_recipes | Recipe of pizzas Danny's restaurant offers |
| 3  | pizza_toppings | Information of pizza toppings |

## Data Schema
<p align="center">
<img src="https://github.com/user-attachments/assets/15577c4f-27e3-4ef5-854e-d329a25a3792" alt="Image" width="670" height="350">

## Data Cleaning
There are two tables that need cleaning before being used:\
**_customer_orders_**
- `exclusions` and `extras` fields have blank spaces, 'null' values (which are text values) and are VARCHAR type -> Delete 'null' (text value); Convert to INTERGER.

**_runner_orders_**
- `pickup_time` is VARCHAR type and has 'null' as text values -> Delete 'null' text values; Convert data type to TIMESTAMP.
- `duration` is VARCHAR type, and has 'null', 'mins', 'minutes',etc. as text values together with numbers -> Remove text values from numbers; Convert data type to NUMERIC.
- `distance` is VARCHAR type, and has 'null', 'km' as text values together with numbers -> Remove text values from numbers; Convert data type to NUMERIC.
- `cancellation` has 'null' as text values -> Delete 'null' text values 

👉 Click [here](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/0efcbfcb2d437556138567d9860a608ebaf2ffa9/Challenge%20%232%20-%20Pizza%20Runner/0.%20DATA%20CLEANING.md) to view **Data Cleaning Queries**.

## Case Study Solutions
🍕[A. Pizza Metrics](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/A.%20PIZZA%20METRICS.md)\
🏃[B. Runner and Customer Experience](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/B.%20RUNNER%20AND%20CUSTOMERS%20EXPERIENCE.md)\
🥓[C. Ingredient Optimization](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/C.%20INGREDIENT%20OPTIMIZATION.md)\
💰[D. Pricing and Ratings](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/D.%20PRICING%20AND%20RATINGS.md)\
✨[E. Bonus Questions](https://github.com/PHAMTHUYDUYEN/8-week-SQL-Challenge/blob/main/Challenge%20%232%20-%20Pizza%20Runner/E.%20BONUS%20QUESTIONS.md)

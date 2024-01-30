# Week 2 Challenge!

<img src='Pizza-Runner.png' alt="Pizza Runner Logo" width=auto height="700">
For more information about the <a href="https://8weeksqlchallenge.com/case-study-2/">week 2</a> challenge. 

### Introduction
Danny has created a Pizza restaurant that offers an "Uber" style delivery system called Pizza Runner. We have been given an ERD but the data requires a little bit of cleaning this time so we can better direct runners and optimize Pizza Runner's operations.

### Problem Statement
To help pay off ther credit card debt that Danny has racked up getting Pizza Runner up and sprinting forward, we were asked to help him understand sales on Pizzas. Because we are curious we will also take a look at the pizza metrics. 
### Data
Danny is proving me 6 datasets to use for the analysis
 - runnder_orders
 - runners
 - customer_orders
 - pizza_names
 - pizza_recipes
 - pizza_toppings

Along with an entity relationship diagram (ERD henceforth).

<img src='Week2-ERD.png' alt="Entity Relationship Diagram for Week 2" width=auto height="500">

### Data Prep
The data that we were given is not sparkly clean so we'll have to bust out the Mr. Clean Magic Erasers and clean it up some. When we have the data cleaned we'll build temp tables to use in the analysis.


Step 1: customer_orders
  - Convert the null and NaN values into empty strings in the exclusions and extras columns
    - This will indicate that the customer did not alter the pizza

Here is the temp table that we will use for the customer orders data after the steps outlined above.

```sql
DROP TABLE IF EXISTS customers;
CREATE TEMP TABLE customers AS

SELECT 
  order_id,
  customer_id,
  pizza_id,
  CASE WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
       WHEN exclusions IS NOT NULL THEN exclusions
       END AS exclusions,
  CASE WHEN extras IS NULL OR extras LIKE 'null' THEN ''
       WHEN extras IS NOT NULL THEN extras
       END AS extras,
  order_time::timestamp
FROM pizza_runner.customer_orders
ORDER BY order_time
```

Step 2: runner_orders
 - Convert the 'null' strings into null values for the pickup_time, distance and duration columns.
 - Extract the numeric values in the distance and duration columns
   - I'll use regular expressions
 - Convert the 'null' and NaN into null values for cancellations

Here is the temp table created after cleaning the data with above steps.

```sql
DROP TABLE IF EXISTS run_orders;
CREATE TEMP TABLE run_orders AS

SELECT 
  order_id,
  runner_id,
  CASE WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ''
       WHEN pickup_time IS NOT NULL THEN pickup_time
       END AS pickup_time,
  regexp_replace(distance, '[^0-9.]+', '', 'g') AS distance_km,
  regexp_replace(duration, '[^0-9]+', '', 'g') AS duration_min,
  CASE WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ''
       ELSE cancellation
       END AS cancellation
FROM pizza_runner.runner_orders
```

### Questions

1. How many pizzas were ordered?

```sql
SELECT 
  COUNT(order_id) AS total_pizzas_ordered
FROM customers
```
| total\_pizzas\_ordered |
| ---------------------- |
| 14                     |

2. How many unique customer orders were made?

```sql
SELECT 
  COUNT(DISTINCT order_id) AS unique_orders
FROM customers
```
| unique\_orders |
| -------------- |
| 10             |

3. How many successful orders were delivered by each runner?

```sql
SELECT 
  runner_id,
  COUNT(*) AS successful_orders
FROM run_orders
WHERE cancellation = ''
GROUP BY 1
```

| runner\_id | successful\_orders |
| ---------- | ------------------ |
| 1          | 4                  |
| 2          | 3                  |
| 3          | 1                  |

4. How many of each type of pizza was delivered?

```sql
SELECT
  customers.pizza_id AS pizza_type,
  COUNT(*) AS frequency
FROM customers
JOIN run_orders
ON customers.order_id = run_orders.order_id
```

| pizza\_type | frequency |
| ----------- | --------- |
| 1           | 9         |
| 2           | 3         |

5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT 
  customers.customer_id,
  COUNT(names.pizza_name) FILTER (WHERE names.pizza_name = 'Meatlovers') AS meatlovers,
  COUNT(names.pizza_name) FILTER (WHERE names.pizza_name = 'Vegetarian') AS vegetarian
FROM customers
JOIN pizza_runner.pizza_names AS names
ON customers.pizza_id = names.pizza_id
GROUP BY 1
ORDER BY 1
```

| customer\_id | meatlovers | vegetarian |
| ------------ | ---------- | ---------- |
| 101          | 2          | 1          |
| 102          | 2          | 1          |
| 103          | 3          | 1          |
| 104          | 3          | 0          |
| 105          | 0          | 1          |

6. What was the maximum number of pizzas delivered in a single order?

```sql
SELECT 
 customers.order_id,
 COUNT(*) AS number_pizzas
FROM customers
JOIN run_orders
ON customers.order_id = run_orders.order_id
WHERE cancellation = ''
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
```
| order\_id | number\_pizzas |
| --------- | -------------- |
| 4         | 3              |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
with changes_made AS(
SELECT 
  customers.customer_id,
  CASE WHEN customers.exclusions !='' OR customers.extras !='' THEN 1 END AS exclusions
FROM customers
JOIN run_orders
ON customers.order_id = run_orders.order_id
WHERE cancellation = ''
)

SELECT 
  customer_id,
  SUM(exclusions) AS exclusions,
  SUM(CASE WHEN exclusions IS NULL THEN 1 END) AS extras
FROM exclusions
GROUP BY 1
```
| customer\_id | exclusions | extras |
| ------------ | ---------- | ------ |
| 101          |            | 2      |
| 102          |            | 3      |
| 103          | 3          |        |
| 104          | 2          | 1      |
| 105          | 1          |        |

8. How many pizzas were delivered that had both exclusions and extras?

```sql
with changes_made AS(
SELECT 
  customers.customer_id,
  CASE WHEN customers.exclusions !='' OR customers.extras !='' THEN 1 END AS exclusions
FROM customers
JOIN run_orders
ON customers.order_id = run_orders.order_id
WHERE cancellation = ''
),

groupby_counts AS(
SELECT 
  customer_id,
  SUM(exclusions) AS exclusions,
  SUM(CASE WHEN exclusions IS NULL THEN 1 END) AS extras
FROM exclusions
GROUP BY 1
)
SELECT *
FROM groupby_counts
WHERE exclusions >= 1 AND extras >= 1
```
| customer\_id | exclusions | extras |
| ------------ | ---------- | ------ |
| 104          | 2          | 1      |


9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
  date_part('hour', order_time) AS hour,
  COUNT(*) AS volumn
FROM customers
orde
GROUP BY 1
ORDER BY 1 
```

| hour | volumn |
| ---- | ------ |
| 11   | 1      |
| 13   | 3      |
| 18   | 3      |
| 19   | 1      |
| 21   | 3      |
| 23   | 3      |


10. What was the volume of orders for each day of the week?
I am assuming that Jan 1 is a Sunday for this analysis.

```sql
SELECT 
  CASE WHEN date_part('day', order_time)::int % 7 = 0 THEN 'sunday'
       WHEN date_part('day', order_time)::int % 7 = 1 THEN 'monday'
       WHEN date_part('day', order_time)::int % 7 = 2 THEN 'tuesday'
       WHEN date_part('day', order_time)::int % 7 = 3 THEN 'wednesday'
       WHEN date_part('day', order_time)::int % 7 = 4 THEN 'thursday'
       WHEN date_part('day', order_time)::int % 7 = 5 THEN 'friday'
       WHEN date_part('day', order_time)::int % 7 = 6 THEN 'saturday'
       END AS weekday,
  COUNT(*)
FROM customers
GROUP BY 1
ORDER BY 2 DESC
```

| weekday   | volumn |
| --------- | ------ |
| thursday  | 5      |
| monday    | 5      |
| tuesday   | 3      |
| wednesday | 1      |


**Stay tuned until the end for any parting thoughts on the challenge and what we could do for further analysese at the end.**

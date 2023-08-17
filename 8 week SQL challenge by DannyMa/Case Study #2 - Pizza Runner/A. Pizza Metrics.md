### <p align="center" style="margin-top: 0px;">  Case Study #2 - Pizza Runner - Pizza Metrics Solution

#### Case Study Questions:

*1) How many pizzas were ordered?*
##### Solution.
```sql
SELECT
   COUNT(order_id) AS pizzas_ordered	
FROM pizza_runner.customer_orders;
```
##### Output.
pizzas_ordered |
-- |
14 | 

*2) How many unique customer orders were made?*
```sql
SELECT
   COUNT(DISTINCT order_id) AS unique_customers	
FROM pizza_runner.customer_orders;
```
##### Output.
unique_customers |
-- |
10 | 


*3) How many successful orders were delivered by each runner?*
```sql
SELECT
   	runner_id,
    COUNT(*) AS total_orders
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY total_orders DESC;
```
##### Output.
runner_id | total_orders
-- | -- |
1 | 4
2 | 3
3 | 1


*4) How many of each type of pizza was delivered?*
```sql
SELECT
   	p.pizza_name,
    COUNT(*) AS total_pizza
FROM pizza_runner.runner_orders AS r
INNER JOIN pizza_runner.customer_orders AS c
 ON r.order_id = c.order_id
INNER JOIN pizza_runner.pizza_names AS p
 ON c.pizza_id = p.pizza_id
WHERE r.cancellation IS NULL
GROUP BY p.pizza_name
ORDER BY total_pizza DESC;
```
##### Output.
pizza_name | total_pizza
-- | -- |
Meatlovers | 9
Vegetarian | 3


*5) How many Vegetarian and Meatlovers were ordered by each customer?*
```sql
SELECT
    c.customer_id,
    p.pizza_name,
    COUNT(*) AS total_pizza
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.pizza_names AS p
 ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```
##### Output.
| customer_id | pizza_name | total_pizza |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |


*6) What was the maximum number of pizzas delivered in a single order?*
```sql
SELECT
   	c.order_id,
    COUNT(*) AS max_total_pizzas_delivered
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.order_id
ORDER BY max_total_pizzas_delivered DESC
LIMIT 1;
```
##### Output.
| order_id | max_total_pizzas_delivered
| ----------- | ---
| 4         | 3



*7) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?*
```sql
```sql
WITH d AS(
  SELECT
    c.customer_id,
    c.exclusions,
    c.extras,
    CASE
    WHEN c.exclusions IS NOT NULL OR c.extras IS NOT NULL THEN 'At least one change'
    ELSE 'No change' END status
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.cancellation IS NULL)

SELECT
   customer_id,
   status,
   COUNT(*) AS count_status
FROM d
GROUP BY customer_id, status
ORDER BY customer_id;
```
##### Output.
| customer_id | status              | count_status |
| ----------- | ------------------- | ------------ |
| 101         | No change           | 2            |
| 102         | No change           | 3            |
| 103         | At least one change | 3            |
| 104         | At least one change | 2            |
| 104         | No change           | 1            |
| 105         | At least one change | 1            |


*8) How many pizzas were delivered that had both exclusions and extras?*
```sql
WITH p AS (
  SELECT
    c.order_id,
    c.exclusions,
    c.extras,
    CASE
    WHEN c.exclusions IS NOT NULL AND c.extras IS NOT NULL THEN 'Both'
    ELSE 'No both' END status
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.distance IS NULL)

SELECT
   COUNT(*) AS count_both
FROM p
WHERE status = 'Both';
```
##### Output.
| count_both | 
| ----------- |
| 1 |


*9) What was the total volume of pizzas ordered for each hour of the day?*
```sql
SELECT
    EXTRACT(hour FROM order_time) AS day_hour,
    COUNT(*) AS total_ordered
FROM pizza_runner.customer_orders
GROUP BY EXTRACT(hour FROM order_time)
ORDER BY EXTRACT(hour FROM order_time);
```
##### Output.
| day_hour | total_ordered |
| -------- | ------------- |
| 11       | 1             |
| 13       | 3             |
| 18       | 3             |
| 19       | 1             |
| 21       | 3             |
| 23       | 3             |


*10) What was the volume of orders for each day of the week?*
```sql
SELECT
    TO_CHAR(order_time, 'day') AS day_week,
    COUNT(*) AS total_ordered
FROM pizza_runner.customer_orders
GROUP BY TO_CHAR(order_time, 'day')
ORDER BY total_ordered DESC;
```
##### Output.
| day_week  | total_ordered |
| --------- | ------------- |
| wednesday | 5             |
| saturday  | 5             |
| thursday  | 3             |
| friday    | 1             |

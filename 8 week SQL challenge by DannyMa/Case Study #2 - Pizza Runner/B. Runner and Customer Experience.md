### <p align="center" style="margin-top: 0px;">  Case Study #2 - Pizza Runner - Runner and Customer Experience Solution

#### Case Study Questions:

*1) How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01).*
##### Solution.
```sql
SELECT
   TO_CHAR(registration_date, 'w') AS week_number,
   COUNT(*) AS total_runners
FROM pizza_runner.runners
GROUP BY TO_CHAR(registration_date, 'w')
ORDER BY TO_CHAR(registration_date, 'w');
```
##### Output.
| week_number | total_runners |
| ----------- | ------------- |
| 1           | 2             |
| 2           | 1             |
| 3           | 1             |


*2) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?*
```sql
SELECT 
   r.runner_id,
   AVG(EXTRACT(min FROM (r.pickup_time::timestamp - c.order_time)))::int AS avg_min 
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY r.runner_id
ORDER BY r.runner_id;
```
##### Output.
| runner_id | avg_min |
| --------- | ------- |
| 1         | 15      |
| 2         | 23      |
| 3         | 10      |


*3) Is there any relationship between the number of pizzas and how long the order takes to prepare?*
```sql
WITH s AS (
  SELECT 
   c.order_id,
   COUNT(c.order_id) AS num_pizzas,
   AVG(EXTRACT(min FROM (r.pickup_time::timestamp - c.order_time)))::int AS avg_min 
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY c.order_id
ORDER BY num_pizzas DESC)

SELECT 
   num_pizzas,
   AVG(avg_min)::int AS avg_min
FROM s
GROUP BY num_pizzas
ORDER BY num_pizzas;
```
##### Output.
| num_pizzas | avg_min |
| ---------- | ------- |
| 1          | 12      |
| 2          | 18      |
| 3          | 29      |


*4) What was the average distance travelled for each customer?*
```sql
SELECT 
   c.customer_id,
   AVG(r.distance) AS avg_distance 
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY avg_distance DESC;
```
##### Output.
| num_pizzas | avg_min |
| ---------- | ------- |
| 1          | 12      |
| 2          | 18      |
| 3          | 29      |

*5) What was the difference between the longest and shortest delivery times for all orders?*

*6) What was the average speed for each runner for each delivery and do you notice any trend for these values?*

*7) What is the successful delivery percentage for each runner?*

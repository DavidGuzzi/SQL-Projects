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
   ROUND(AVG(r.distance)) AS avg_distance 
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.runner_orders AS r
 ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
##### Output.
| customer_id | avg_distance |
| ---------- | ------- |
| 101          | 20      |
| 102          | 17      |
| 103          | 23      |
| 104          | 10      |
| 105          | 25      |

*5) What was the difference between the longest and shortest delivery times for all orders?*
```sql
WITH maxmin AS(
SELECT 
   MAX(duration) AS max_duration,
   MIN(duration) AS min_duration
FROM pizza_runner.runner_orders
WHERE duration IS NOT NULL)

SELECT
   max_duration,
   min_duration,
   (max_duration - min_duration) AS difference
FROM maxmin;
```
##### Output.
| max_duration | min_duration | difference |
| ---------- | ------- |------- |
| 40          | 10      | 30    |

*6) What was the average speed for each runner for each delivery and do you notice any trend for these values?*
```sql
WITH speed AS(
SELECT 
   order_id,
   runner_id,
   distance,
   (duration / 60) AS duration_hrs,
   (distance / duration * 60) AS speed
FROM pizza_runner.runner_orders
WHERE distance IS NOT NULL AND duration IS NOT NULL)

SELECT 
   runner_id,
   order_id,
   AVG(speed) AS avg_speed
FROM speed
GROUP BY runner_id, order_id
ORDER BY runner_id, order_id;
```
##### Output.
| runner_id | order_id | avg_speed          |
| --------- | -------- | ------------------ |
| 1         | 1        | 37.5               |
| 1         | 2        | 44.44444444444444  |
| 1         | 3        | 40.2               |
| 1         | 10       | 60                 |
| 2         | 4        | 35.099999999999994 |
| 2         | 7        | 60                 |
| 2         | 8        | 93.6               |
| 3         | 5        | 40                 |

*7) What is the successful delivery percentage for each runner?*


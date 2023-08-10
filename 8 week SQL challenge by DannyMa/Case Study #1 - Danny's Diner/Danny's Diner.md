### <p align="center" style="margin-top: 0px;">  Case Study #1 - Danny's Diner Solution

#### Case Study Questions:

*1) What is the total amount each customer spent at the restaurant?*
##### Solution.
```sql
SELECT
    s.customer_id,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY SUM(m.price) DESC;
```
##### Output.
customer_id | total_spent
--| --
A | 76 
B | 74
C | 36 

*2) How many days has each customer visited the restaurant?*
##### Solution.
```sql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY days_visited DESC;
```
##### Output.
customer_id | days_visited
--| --
B | 6 
A | 4
C | 2 

*3) What was the first item from the menu purchased by each customer?*
##### Solution.
```sql
WITH rank AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.product_name,
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id 
      ORDER BY s.order_date) AS rank
  FROM dannys_diner.sales AS s
  INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id)

SELECT 
    r.customer_id,
    r.product_name AS first_purchased
FROM rank AS r
WHERE r.rank = 1
GROUP BY r.customer_id, r.product_name;
```
##### Output.
customer_id | first_purchased
--| --
A | curry
A | sushi
B | curry
C | ramen

*4) What is the most purchased item on the menu and how many times was it purchased by all customers?*
##### Solution.
##### Most purchased item:
```sql
SELECT 
    m.product_name AS most_purchased, 
    COUNT(m.product_name) AS total_purchased
FROM dannys_diner.menu AS m
INNER JOIN dannys_diner.sales AS s ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY COUNT(m.product_name) DESC
LIMIT 1;
```
##### Output.
most_purchased | total_purchased
--| --
ramen | 8
##### Most purchased item for each customer:
```sql
SELECT 
    m.product_name AS most_purchased,
    s.customer_id,
    COUNT(m.product_name) AS number_purchased
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
WHERE s.product_id IN (
  SELECT product_id
  FROM dannys_diner.sales
  GROUP BY product_id
  ORDER BY COUNT(product_id) DESC
  LIMIT 1)
GROUP BY m.product_name, s.customer_id
ORDER BY number_purchased DESC;
```
##### Output.
most_purchased | customer_id | number_purchased
--| -- | --
ramen | C | 3 
ramen | A | 3
ramen | B | 2

*5) Which item was the most popular for each customer?*
##### Solution.
```sql
WITH rank AS (
  SELECT 
    s.customer_id, 
    m.product_name AS popular_item, 
    COUNT(m.product_name),
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id 
      ORDER BY COUNT(s.customer_id) DESC) AS rank
  FROM dannys_diner.menu AS m
  INNER JOIN dannys_diner.sales AS s ON m.product_id = s.product_id
  GROUP BY s.customer_id, m.product_name
)

SELECT 
  customer_id, 
  popular_item, 
  count
FROM rank 
WHERE rank = 1;
```
##### Output.
customer_id | popular_item | count
--| -- | --
A | ramen | 3
B | ramen | 2
B | curry | 2
B | sushy | 2
C | ramen | 3

*6) Which item was purchased first by the customer after they became a member?*
##### Solution.
```sql
WITH rank AS (
  SELECT
  	s.customer_id,
    s.order_date,
    m.join_date,
    n.product_name,
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.members AS m 
ON s.customer_id = m.customer_id AND s.order_date > m.join_date
INNER JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id)

SELECT 
   customer_id,
   product_name
FROM rank
WHERE rank = 1;
```
##### Output.
customer_id | product_name
--| -- 
A | ramen
B | sushi

*7) Which item was purchased just before the customer became a member?*
##### Solution.
```sql
WITH rank AS (
  SELECT
  	s.customer_id,
    s.order_date,
    m.join_date,
    n.product_name,
    DENSE_RANK() OVER(
      PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.members AS m 
ON s.customer_id = m.customer_id AND s.order_date < m.join_date
INNER JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id)

SELECT 
   customer_id,
   product_name
FROM rank
WHERE rank = 1;
```
##### Output.
customer_id | product_name
--| -- 
A | sushi
A | curry
B | sushi


*8) What is the total items and amount spent for each member before they became a member?*
```sql
SELECT
  	s.customer_id,
    COUNT(n.product_name) AS total_items,
    SUM(n.price) AS total_amount
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.members AS m 
ON s.customer_id = m.customer_id AND s.order_date < m.join_date
INNER JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id
GROUP BY s.customer_id;
```
##### Output.
customer_id | total_items | total_amount
--| -- | --
B | 3 | 40
A | 2 | 25

*9)  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?*
```sql
WITH points AS (
  SELECT
  	s.customer_id,
    n.product_name,
    CASE n.product_name 
    WHEN 'sushi' THEN n.price * 20
    ELSE n.price * 10 
    END points
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id)

SELECT 
   customer_id,
   SUM(points) AS total_points
FROM points
GROUP BY customer_id
ORDER BY total_points DESC;
```
##### Output.
customer_id | total_points
--| -- 
B | 940
A | 860
C | 360


*10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?*


### <p align="center" style="margin-top: 0px;">  Bonus questions - Recreate tables

#### Bonus 1.
#### Output.
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
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

#### Solution.
```sql
SELECT
  	s.customer_id,
    s.order_date,
    n.product_name,
    n.price,
    CASE WHEN s.order_date >= m.join_date THEN 'Y'
    WHEN s.order_date < m.join_date THEN 'N'
    WHEN m.join_date IS NULL THEN 'N'
    END member
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS m 
ON s.customer_id = m.customer_id
LEFT JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id
ORDER BY s.customer_id, s.order_date;
```

#### Bonus 2.
#### Output.
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | -----------| ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 2       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 4       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 4       |
| B           | 2021-01-01 | curry        | 15    | N      | null    |
| B           | 2021-01-02 | curry        | 15    | N      | null    |
| B           | 2021-01-04 | sushi        | 10    | N      | null    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 4       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 5       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 6       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    |

#### Solution.
```sql
WITH tabla AS (
  SELECT
  	s.customer_id,
    s.order_date,
    n.product_name,
    n.price,
    CASE WHEN s.order_date >= m.join_date THEN 'Y'
    WHEN s.order_date < m.join_date THEN 'N'
    WHEN m.join_date IS NULL THEN 'N'
    END member
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS m 
ON s.customer_id = m.customer_id
LEFT JOIN dannys_diner.menu AS n
ON s.product_id = n.product_id
ORDER BY s.customer_id, s.order_date)

SELECT
    *,
    CASE 
    WHEN member = 'Y' THEN (DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date))
    ELSE NULL END ranking
 FROM tabla;
```

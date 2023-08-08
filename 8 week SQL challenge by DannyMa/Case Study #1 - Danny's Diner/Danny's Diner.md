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

*7) Which item was purchased just before the customer became a member?*

*8) What is the total items and amount spent for each member before they became a member?*

*9)  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?*

*10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?*

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
SELECT 
      s.customer_id,
      m.product_name AS first_purchased
FROM dannys_diner.sales AS s  
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
WHERE s.order_date IN  (
  SELECT MIN(order_date)
  FROM dannys_diner.sales)
GROUP BY  s.customer_id, m.product_name
ORDER BY s.customer_id;
```
##### Output.
customer_id | first_purchased
--| --
A | curry
A | sushi
B | curry
C | ramen



*4) What is the most purchased item on the menu and how many times was it purchased by all customers?*

*5) Which item was the most popular for each customer?*

*6) Which item was purchased first by the customer after they became a member?*

*7) Which item was purchased just before the customer became a member?*

*8) What is the total items and amount spent for each member before they became a member?*

*9)  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?*

*10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?*

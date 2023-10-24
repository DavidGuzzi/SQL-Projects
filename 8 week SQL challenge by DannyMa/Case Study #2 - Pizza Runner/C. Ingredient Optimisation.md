### <p align="center" style="margin-top: 0px;">  Case Study #2 - Pizza Runner - Ingredient Optimisation

#### Case Study Questions:

*1) What are the standard ingredients for each pizza?*
##### Solution.
```sql
WITH data AS (
  SELECT pn.pizza_name, t.topping_name
  FROM pizza_runner.pizza_recipes AS pc
  INNER JOIN pizza_runner.pizza_names AS pn
    ON pc.pizza_id = pn.pizza_id
  INNER JOIN pizza_runner.pizza_toppings AS t
    ON t.topping_id = ANY(string_to_array(pc.toppings, ', ')::int[]))

SELECT pizza_name, STRING_AGG(topping_name, ', ') AS standard_ingredients
FROM data
GROUP BY pizza_name;
```
##### Output.
| pizza_name | standard_ingredients                                                  |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

*2) What was the most commonly added extra?*
##### Solution.
```sql
SELECT t.topping_id, t.topping_name, COUNT(*) AS extra_count
FROM pizza_runner.pizza_toppings AS t
INNER JOIN pizza_runner.customer_orders AS c
 ON t.topping_id = ANY(string_to_array(c.extras, ', ')::int[])
 WHERE c.extras IS NOT NULL
 GROUP BY t.topping_id, t.topping_name
 ORDER BY extra_count DESC, t.topping_id;
```
##### Output.
| topping_id | topping_name | extra_count |
| ---------- | ------------ | ----------- |
| 1          | Bacon        | 4           |
| 4          | Cheese       | 1           |
| 5          | Chicken      | 1           |

*3) What was the most common exclusion?*
##### Solution.
```sql
SELECT t.topping_id, t.topping_name, COUNT(*) AS exclusion_count
FROM pizza_runner.pizza_toppings AS t
INNER JOIN pizza_runner.customer_orders AS c
 ON t.topping_id = ANY(string_to_array(c.exclusions, ', ')::int[])
 WHERE c.exclusions IS NOT NULL
 GROUP BY t.topping_id, t.topping_name
 ORDER BY exclusion_count DESC, t.topping_id;
```
##### Output.
| topping_id | topping_name | exclusion_count |
| ---------- | ------------ | --------------- |
| 4          | Cheese       | 4               |
| 2          | BBQ Sauce    | 1               |
| 6          | Mushrooms    | 1               |

*4) *Generate an order item for each record in the customers_orders table in the format of one of the following:*
        
        - Meat Lovers
        
        - Meat Lovers - Exclude Beef
        
        - Meat Lovers - Extra Bacon
        
        - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

*5) Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients*
        
        - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

*6) What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?*
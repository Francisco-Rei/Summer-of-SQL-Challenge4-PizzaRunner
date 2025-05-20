# SQL_Pizza_Runner

This is a challenge from [Summer of SQL](https://github.com/wjsutton/the_summer_of_sql) to help gain skills and confidence in SQL.

Skills tested in this challenge:
- Searching Text
- Case Statements
- NULLs
- Datetime and Scalar functions
- Comma Separated Strings
- Aggregation, Unions
- CTEs
- Type Conversions

Danny founded Pizza Runner, a startup that combines a traditional pizza delivery service with the modern "Uberized" gig economy model. To scale the business, he innovated by building a platform that allows freelance delivery drivers—called “runners”—to pick up and deliver pizzas from a central location: Danny’s house. Leveraging his background as a data scientist, Danny ensured the business collected detailed data to help guide operational improvements and inform business decisions. [Information](https://8weeksqlchallenge.com/case-study-2/)

## Problem Statement
Danny has designed an entity relationship diagram (ERD) for the business database and now needs help cleaning the data and performing key calculations to optimize deliveries, runner efficiency, and overall business performance.
This case study has LOTS of questions - they are broken up by area of focus including:
- Pizza Metrics
- Runner and Customer Experience
- Ingredient Optimisation

## Case Study Questions
### A. Pizza Metrics
1. How many pizzas were ordered?
````sql
SELECT COUNT(order_id) AS total_pizzas_ordered
FROM customer_orders
````
| TOTAL_PIZZAS_ORDERED |
|----------------------|
| 14                   |

2. How many unique customer orders were made?
````sql
SELECT COUNT(DISTINCT customer_id) AS unique_customers
FROM customer_orders;
````
| UNIQUE_CUSTOMERS_ORDERS |
|--------------------------|
| 5                        |

3. How many successful orders were delivered by each runner?
````sql
SELECT runner_id, COUNT(DISTINCT order_id) AS delivered_orders 
FROM runner_orders 
WHERE pickup_time <> 'null' 
GROUP BY runner_id;
````
| RUNNER_ID | DELIVERED_ORDERS |
|-----------|------------------|
| 3         | 1                |
| 2         | 3                |
| 1         | 4                |

4. How many of each type of pizza was delivered?
````sql
SELECT pizza_name, COUNT(c.order_id) AS delivered_pizzas 
FROM customer_orders AS c 
INNER JOIN pizza_names AS p ON p.pizza_id = c.pizza_id
INNER JOIN runner_orders AS r ON r.order_id = c.order_id 
WHERE pickup_time<>'null'
GROUP BY pizza_name;
````
| PIZZA_NAME  | DELIVERED_PIZZAS |
|-------------|------------------|
| Meatlovers  | 9                |
| Vegetarian  | 3                |


5. How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT c.customer_id, pn.pizza_name, COUNT(pn.pizza_name) AS ordered_pizzas
FROM customer_orders AS c
INNER JOIN pizza_names AS pn ON pn.pizza_id = c.pizza_id
GROUP BY  c.customer_id, pn.pizza_name
````
| CUSTOMER_ID | PIZZA_NAME  | ORDERED_PIZZAS |
|-------------|-------------|----------------|
| 105         | Vegetarian  | 1              |
| 104         | Meatlovers  | 3              |
| 101         | Vegetarian  | 1              |
| 101         | Meatlovers  | 2              |
| 103         | Vegetarian  | 1              |
| 102         | Meatlovers  | 2              |
| 102         | Vegetarian  | 1              |
| 103         | Meatlovers  | 3              |


6. What was the maximum number of pizzas delivered in a single order?
````sql
SELECT ro.order_id, COUNT(co.order_id) AS delivered_pizzas 
FROM customer_orders as co 
INNER JOIN pizza_names as pn on pn.pizza_id = co.pizza_id 
INNER JOIN runner_orders as ro on ro.order_id = co.order_id 
WHERE pickup_time<>'null'
GROUP BY ro.order_id 
ORDER BY COUNT(co.order_id) DESC 
LIMIT 1;
````
| ORDER_ID | DELIVERED_PIZZAS |
|----------|------------------|
| 4        | 3                |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
SELECT customer_id, 
  SUM(CASE 
    WHEN(
        (exclusions IS NOT NULL AND exclusions<>'null' AND LENGTH(exclusions)>0) 
        OR (extras IS NOT NULL AND extras<>'null' AND LENGTH(extras)>0)
        )=TRUE
    THEN 1 
    ELSE 0
  END) as changes, 
  SUM(CASE 
    WHEN 
        (
          (exclusions IS NOT NULL AND exclusions<>'null' AND LENGTH(exclusions)>0) 
        OR (extras IS NOT NULL AND extras<>'null' AND LENGTH(extras)>0)
        )=TRUE
    THEN 0 
    ELSE 1
  END) as no_changes 
FROM customer_orders as co 
INNER JOIN runner_orders as ro on ro.order_id = co.order_id 
WHERE pickup_time<>'null'
GROUP BY customer_id;
````
| CUSTOMER_ID | CHANGES | NO_CHANGES |
|-------------|---------|-------------|
| 103         | 3       | 0           |
| 104         | 2       | 1           |
| 105         | 1       | 0           |
| 101         | 0       | 2           |
| 102         | 0       | 3           |

8. How many pizzas were delivered that had both exclusions and extras?
````sql
SELECT COUNT(c.order_id) AS pizzas_delivered_with_exclusionsandextras
FROM customer_orders AS c
INNER JOIN runner_orders AS ro ON ro.order_id = c.order_id
WHERE pickup_time <> 'null' AND (exclusions IS NOT NULL AND exclusions<>'null' AND LENGTH(exclusions)>0) AND (extras IS NOT NULL AND extras<>'null' AND LENGTH(extras)>0);
````
| PIZZAS_DELIVERED_WITH_EXCLUSIONSANDEXTRAS |
|-------------------------------------------|
| 1                                         |

9. What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT HOUR(order_time) AS hours, COUNT(order_id) AS pizzas_delivered
FROM customer_orders
GROUP BY HOUR(order_time)
ORDER BY HOUR(order_time) ASC;
````
| HOURS | PIZZAS_DELIVERED |
|-------|------------------|
| 11    | 1                |
| 13    | 3                |
| 18    | 3                |
| 19    | 1                |
| 21    | 3                |
| 23    | 3                |

10. What was the volume of orders for each day of the week?
````sql
SELECT DAYNAME(order_time) AS day_of_week, count(order_id) AS orders
FROM customer_orders
GROUP BY DAYNAME(order_time)
````
| DAY_OF_WEEK | ORDERS |
|-------------|--------|
| Thu         | 3      |
| Wed         | 5      |
| Fri         | 1      |
| Sat         | 5      |

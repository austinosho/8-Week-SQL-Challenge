# Case Study #2 Pizza Runner https://8weeksqlchallenge.com/case-study-2/
<img src = "https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/b833d7ae-5db9-4f06-b530-e85540f6b62f" alt="img" width="600" height="500">

## 📖Table of Contents
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Tools](#tools)
- [Case Study Questions and Solution](#case-study-questions-and-solution)

## Introduction
Danny, a pizza lover with a great desire for innovation, launched Pizza Runner - a blend of retro vibes and modern convenience. Seeing the massive daily pizza consumption worldwide, he sensed an opportunity to shake up the food delivery scene. But Danny didn't stop at just serving pizza; he envisioned a fleet of delivery runners and a sleek mobile app to make ordering swift.
Being no stranger to data, Danny set up a robust database to track Pizza Runner's performance. Now, he's eager to dive into the numbers, clean up the data, and uncover insights that will help him fine-tune operations and delight customers. Incorporating a touch of data magic, Danny's on a mission to make Pizza Runner the go-to choice for pizza lovers everywhere.

## Entity Relationship Diagram
![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/002f8981-8a0a-4de0-94a8-67a5eb9a1970)

## 🧑‍💻Tools 
PostgreSQL

## Data Cleaning and Transformation
Before starting to write the SQL queries, it was necessary to investigate the data, as some actions needed to be taken regarding null values and data types in the customer_orders and runner_orders tables.

**Table: customer_orders**
````sql
SELECT * FROM pizza_runner.customer_orders;
````

Viewing the `customer_orders` table below, we can see that there are null and blank ' ' values in both the `exclusions` and `extras` columns 

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/57a93b9e-13bd-40d4-bcbe-8e62b9e43493)

To clean the table:
  1. We will create a duplicate table with all the columns.
  2. Remove null values in `exlusions` and `extras` columns and replace with blank space ' '.

````sql
CREATE TABLE pizza_runner.customer_order_cleaned AS
	SELECT 
	    order_id,
	    customer_id,
	    pizza_id,
	    CASE 
	        WHEN exclusions = 'null' OR exclusions IS NULL THEN ' '
	        ELSE exclusions
	    END AS exclusions,
	    CASE 
	        WHEN extras = 'null' OR extras IS NULL THEN ' '
	        ELSE extras
	    END AS extras,
	    order_time
	FROM pizza_runner.customer_orders;
````

View the `customer_order_cleaned` table
````sql
SELECT * FROM pizza_runner.customer_order_cleaned;
````
This is how the cleaned data looks and this table will be used to run all our queries

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/ea3b22fb-09aa-4a8a-bf38-84ee36bd5c53)

**Table: runner_orders**
````sql
SELECT * FROM pizza_runner.runner_orders;
````

Viewing the `runner_orders` table below, we can see that there are null and blank ' ' values in the `pickup_time`, `distance`, `duration` and `cancellation` columns, also there are some unwanted values in the `distance` and `duration` columns that need to be handled

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/e4f73d71-dff9-47cd-9ebf-19fc500257ec)

To clean the table:
  1. We will create a duplicate table with all the columns.
  2. pickup_time column: Remove null values and set them to NULL.
  3. distance column: Remove "km" and set null values to NULL.
  4. duration column: Remove "minutes", "minute", and set null values to NULL.
  5. cancellation column: Remove NULL and null values and replace them with blank space ' '.

````sql
DROP TABLE IF EXISTS pizza_runner.runner_orders_cleaned;
CREATE TABLE pizza_runner.runner_orders_cleaned AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN NULL 
-- leaving time as null to avoid errors when altering data type
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN NULL
-- leaving distance as null to avoid errors when altering data type
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN NULL
-- leaving duration as null to avoid errors when altering data type
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ' '
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
````
Next, we alter the `pickup_time`, `distance` and `duration` columns to the correct data type.

````sql
ALTER TABLE pizza_runner.runner_orders_cleaned
ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::timestamp without time zone,
ALTER COLUMN distance TYPE FLOAT USING distance::double precision,
ALTER COLUMN duration TYPE INT USING duration::integer;
````

View the runner_orders_cleaned table
````sql
SELECT * FROM pizza_runner.runner_orders_cleaned;
````

This is how the cleaned data looks and this table will be used to run all our queries
![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/c5a5b152-33bd-4bae-a090-e16dd58455de)

****
## Case Study Questions and Solution

## A. Pizza Metrics

**1. How many pizzas were ordered?**
````sql
SELECT COUNT(pizza_id)
FROM pizza_runner.customer_order_cleaned
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/22776865-fe6c-4492-aacd-37782cbcddc5)

- 14 pizzas were ordered in total.

****

**2. How many unique customer orders were made?**
````sql
SELECT COUNT(DISTINCT order_id)
FROM pizza_runner.customer_order_cleaned
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/89ef6b54-1fa1-4a95-a682-d98969c1ab76)

- 10 unique customer orders were made.

****

**3. How many successful orders were delivered by each runner?**
````sql
SELECT 
	runner_id,
	COUNT(duration) as successful_orders
FROM pizza_runner.runner_orders_cleaned
WHERE duraTion IS NOT NULL
AND distance!= 0
GROUP BY  runner_id
ORDER BY runner_id
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/79dc9fa2-bf2f-4f77-b503-b58c796742fa)

- Runner 1 delivered 4 successful orders.
- Runner 2 delivered 3 successful orders.
- Runner 3 delivered 1 successful orders.

****

**4. How many of each type of pizza was delivered?**
````sql
SELECT 
	c.pizza_name,
	COUNT(a.pizza_id) as "number delivered"
FROM pizza_runner.customer_order_cleaned as a
	JOIN pizza_runner.runner_orders_cleaned as b
		ON a.order_id = b.order_id
	LEFT JOIN pizza_runner.pizza_names as c
		ON a.pizza_id = c.pizza_id
	WHERE b.duration is NOT NULL
	AND b.distance!= 0
GROUP BY  c.pizza_name
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/ad470b8d-7d85-4936-8417-c4959994012d)

- 9 Meatlovers pizzas were delivered in total
- 3 Vegetarian pizzas were delivered in total

****

**5. How many Vegetarian and Meatlovers were ordered by each customer?**
````sql
SELECT 
	a.customer_id,
	b.pizza_name,
	COUNT(a.pizza_id) as "number of pizzas ordered"
FROM pizza_runner.customer_order_cleaned as a
	JOIN pizza_runner.pizza_names as b
		ON a.pizza_id = b.pizza_id
GROUP BY a.customer_id, b.pizza_name
ORDER BY a.customer_id
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/fc3f3f80-31d1-490b-94ca-0979334723ae)

- Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 102 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 3 Meatlovers pizzaS.
- Customer 105 ordered 1 Vegetarian pizza.

****

**6. What was the maximum number of pizzas delivered in a single order?**
````sql
SELECT 
	a.order_id,
	COUNT(a.pizza_id) as pizza_ordered
FROM pizza_runner.customer_order_cleaned as a
	JOIN pizza_runner.runner_orders_cleaned as b
	ON a.order_id = b.order_id
	WHERE duration IS NOT NULL
	AND distance!= 0
GROUP BY a.order_id
ORDER BY pizza_ordered desc
LIMIT 1;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/3ac4d2e0-f447-4795-be28-1be530ec77f4)

- The Maximum number of pizzas delivered in a single order was 3 pizzas.

**Another way to get the same result is using CTE;**
````sql
WITH SINGLE_ORDER_COUNT AS(
	SELECT 
		a.order_id,
		COUNT(a.pizza_id) as pizza_ordered
	FROM pizza_runner.customer_order_cleaned as a
		JOIN pizza_runner.runner_orders_cleaned as b
		ON a.order_id = b.order_id
		WHERE duration IS NOT NULL
		AND distance!= 0
	GROUP BY a.order_id
)
SELECT 
	MAX(pizza_ordered) as "Max single order delivery"
FROM SINGLE_ORDER_COUNT;
````

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/8d6109cf-79b1-4c68-b9f0-308d01920fcb)

****

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
````sql
SELECT 
    a.customer_id,
	SUM(CASE 
            WHEN a.exclusions!= '' OR a.extras!= '' THEN 1 
            ELSE 0 
        END) AS at_least_one_change,
    SUM(CASE 
            WHEN a.exclusions = '' AND a.extras = '' THEN 1 
            ELSE 0 
        END) AS no_change
FROM pizza_runner.customer_order_cleaned AS a
JOIN pizza_runner.runner_orders_cleaned AS b
    ON a.order_id = b.order_id
WHERE b.duration IS NOT NULL
  AND b.distance!= '0'  
GROUP BY a.customer_id
ORDER BY a.customer_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/4e944816-92b8-4608-a741-b3e5ca08ab71)

- Customers 101 and 102 love their pizzas just the way they come, without any changes.
- But Customers 103, 104, and 105 prefer to customize their pizzas. They each asked for at least one change, like adding extra toppings or leaving some out.

****

**8. How many pizzas were delivered that had both exclusions and extras?**
````sql
SELECT 
	SUM(CASE 
            WHEN a.exclusions!= '' AND a.extras!= '' THEN 1 
            ELSE 0 
        END) AS pizza_delivered
FROM pizza_runner.customer_order_cleaned AS a
JOIN pizza_runner.runner_orders_cleaned AS b
    ON a.order_id = b.order_id
WHERE b.duration IS NOT NULL
  AND b.distance!= '0';
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/091a79a9-5096-4add-9a8a-376bded5dfd8)

- One pizza had both extra and exclusion toppings from the delivered pizzas

****

**9. What was the total volume of pizzas ordered for each hour of the day?**
````sql
SELECT 
	EXTRACT(HOUR FROM order_time) AS hourly_order,
	COUNT(pizza_id) AS total_pizza_ordered
	FROM pizza_runner.customer_order_cleaned
GROUP BY hourly_order
ORDER BY hourly_order;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/bc997145-5993-48b8-9b74-f5f292446d94)

- Highest volume of pizza ordered is at 13 (1:00 pm), 18 (6:00 pm), 21 (9:00 pm) and 23 (11:00 pm).
- Lowest volume of pizza ordered is at 11 (11:00 am), 19 (7:00 pm).

****

**10. What was the volume of orders for each day of the week?**
````sql
SELECT 
	TO_CHAR(order_time + INTERVAL '2 day', 'Day') AS day_of_the_week, -- add 2 to set 1st day of the week as Monday
	COUNT(pizza_id) AS total_pizza_ordered
	FROM pizza_runner.customer_order_cleaned
GROUP BY day_of_the_week;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/20b7033f-83ad-49ee-9731-ddebe3b48f5c)

- There are 5 pizzas ordered on Monday.
- There are 5 pizzas ordered on Friday.
- There are 3 pizzas ordered on Saturday.
- There is 1 pizza ordered on Sunday.

****




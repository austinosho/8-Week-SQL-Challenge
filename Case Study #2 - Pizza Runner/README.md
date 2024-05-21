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
  



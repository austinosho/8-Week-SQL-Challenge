# Case Study #2 Pizza Runner https://8weeksqlchallenge.com/case-study-2/
<img src = "https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/b833d7ae-5db9-4f06-b530-e85540f6b62f" alt="img" width="600" height="500">

## 📖Table of Contents
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Tools](#tools)
- [Data Cleaning and Transformation](#data-cleaning-and-transformation)
- [Case Study Questions and Solution](#case-study-questions-and-solution)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)

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

## B. Runner and Customer Experience

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
````sql
SELECT 
    FLOOR((registration_date::date - DATE '2021-01-01') / 7) + 1 AS week_number,
    COUNT(runner_id) AS signups
FROM 
    pizza_runner.runners
WHERE 
    registration_date >= '2021-01-01'
GROUP BY 
    week_number
ORDER BY 
    week_number;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/e235a1ec-70e5-4b98-946b-33ceb35189e0)

- 2 new runners signed up in Week 1 of January 2021.
- 1 new runner signed up in Week 2 of January 2021.
- 1 new runner signed up in Week 3 of January 2021.

****

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
````sql
WITH time_taken AS (
  SELECT 
	r.runner_id,
    c.order_id, 
    c.order_time, 
    r.pickup_time, 
    EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)) / 60 AS pickup_minutes
  FROM pizza_runner.customer_order_cleaned AS c
  JOIN pizza_runner.runner_orders_cleaned AS r
    ON c.order_id = r.order_id
  GROUP BY r.runner_id, c.order_id, c.order_time, r.pickup_time
)
SELECT 
  runner_id,
  ROUND(AVG(pickup_minutes), 2) AS avg_pickup_minutes
FROM time_taken
GROUP BY runner_id
ORDER BY runner_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/2877deba-7bd4-42ea-9772-19d03d22861e)

- The average time in minutes for runner 1 to arrive at the Pizza Runner HQ for pickup was 14 minutes.
- The average time in minutes for runner 2 to arrive at the Pizza Runner HQ for pickup was 20 minutes.
- The average time in minutes for runner 3 to arrive at the Pizza Runner HQ for pickup was 10 minutes.

****

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
````SQL
WITH prep_cte AS (
    SELECT
        c.order_id,
        c.order_time,
        r.pickup_time,
        COUNT(c.pizza_id) AS no_of_pizzas,
		EXTRACT(MINUTE FROM (r.pickup_time - c.order_time)) AS prep_time_minutes
    FROM pizza_runner.customer_order_cleaned AS c
    JOIN pizza_runner.runner_orders_cleaned AS r
        ON c.order_id = r.order_id
    WHERE r.duration IS NOT NULL
        AND r.distance != 0
    GROUP BY c.order_id, c.order_time, r.pickup_time
)
SELECT 
    no_of_pizzas,
    ROUND(AVG(prep_time_minutes)) AS avg_prep_time_minutes
FROM prep_cte
GROUP BY no_of_pizzas
ORDER BY no_of_pizzas;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/2840ed43-e315-4dd6-9faf-92caccccca5e)

- It takes an average of 12 minutes to prepare an order for a single pizza.
- An order with 3 pizzas takes 29 minutes, averaging approximately 9.67 minutes per pizza.
- It takes 18 minutes to prepare an order with 2 pizzas, averaging 9 minutes per pizza, indicating the highest efficiency rate.

****

**4. What was the average distance travelled for each customer?**
````sql
SELECT 
  c.customer_id, 
  ROUND(AVG(r.distance)::NUMERIC, 2) AS avg_distance
FROM pizza_runner.customer_order_cleaned AS c
JOIN pizza_runner.runner_orders_cleaned AS r
  ON c.order_id = r.order_id
WHERE r.duration != 0
GROUP BY c.customer_id
ORDER BY c.customer_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/4015fe4d-16b9-451c-96f2-ff7b20b28d43)

- Customer 104 stays the nearest to Pizza Runner HQ at an average distance of 10 km.
- Customer 105 stays the furthest at 25 km.

****

**5. What was the difference between the longest and shortest delivery times for all orders?**
````sql
SELECT 
	(MAX(duration) - MIN(duration)) AS delivery_time_diff
FROM pizza_runner.runner_orders_cleaned
WHERE duration IS NOT NULL;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/c469f0dc-e640-4c69-91a6-b984d1a8e2d7)

- The difference between the longest and shortest delivery times for all orders is 30 minutes.

****

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
````sql
SELECT 
	r.runner_id,
	c.customer_id,
	c.order_id,
	COUNT(pizza_id) AS no_of_pizza,
	r.distance,
	(r.duration / 60) AS time_hr,
	ROUND((r.distance / r.duration * 60)::NUMERIC, 2) AS avg_speed	
FROM pizza_runner.runner_orders_cleaned AS r
JOIN pizza_runner.customer_order_cleaned AS c
	ON r.order_id = c.order_id
WHERE r.duration IS NOT NULL
	AND r.distance IS NOT NULL
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY c.order_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/ccf77ba9-37e7-49de-a3cb-79bc27f826c2)

- Average speed (avg_speed) varies significantly, ranging from 35.10 to 93.60 km per hour.
- Runner 2 recorded the highest average speed (93.60 km/hour) for order 8, which may indicate either exceptional efficiency.
- Runner 1 handled the most deliveries (orders 1, 2, 3, and 10) with consistent average speeds between 37.50 and 60 km/hour.
- Runner 3 had an average speed of 40km/h.
- Orders 4 and 8 both had a distance of 23.4 km, but differing average speeds and times suggest varying conditions or efficiencies.

****

**7. What is the successful delivery percentage for each runner?**
````SQL
SELECT 
    runner_id,
    CONCAT(ROUND((SUM(CASE WHEN distance != 0 THEN 1 ELSE 0 END) * 100.0) / NULLIF(COUNT(*), 0)), '%') AS successful_delivery_percentage
FROM pizza_runner.runner_orders_cleaned 
GROUP BY runner_id
ORDER BY runner_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/3a849be0-6852-42a9-a91a-a8cf67267385)

- Runner 1 had a successful delivery percentage of 100%.
- Runner 2 had a successful delivery percentage of 75%.
- Runner 3 had a successful delivery percentage of 50%.

****

## C. Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**

````sql
SELECT
    t.topping_name AS standard_ingredient
FROM
    pizza_runner.pizza_recipes AS p
JOIN
    pizza_runner.pizza_toppings AS t ON t.topping_id = ANY (string_to_array(p.toppings, ',')::int[])
JOIN
    pizza_runner.pizza_recipes AS pr ON pr.pizza_id <> p.pizza_id
JOIN
    pizza_runner.pizza_toppings AS pt ON pt.topping_id = ANY (string_to_array(pr.toppings, ',')::int[])
WHERE
    p.pizza_id = 1 
AND pr.pizza_id = 2 
AND t.topping_id = pt.topping_id
GROUP BY
    t.topping_id, t.topping_name;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/311bf7ca-ca4a-44f3-8261-fa8b9f336c29)

- Ingredients for Meatlovers pizza are: "Bacon , BBQ Sauce , Beef , Cheese , Chicken , Mushrooms , Pepperoni , Salami".
- Ingredients for Vegeterian pizza are: "Cheese , Mushrooms , Onions , Peppers , Tomatoes , Tomato Sauce".

****

**2. What was the most commonly added extra?**

````sql
WITH extra AS (
  SELECT 
    UNNEST(STRING_TO_ARRAY(extras, ','))::INTEGER AS extra_id
  FROM 
    pizza_runner.customer_order_cleaned
  WHERE
    extras IS NOT NULL AND extras <> ''
)

SELECT 
  e.extra_id,
  pt.topping_name AS extra_name,
  COUNT(e.extra_id) AS times_added
FROM 
  extra e
JOIN 
  pizza_runner.pizza_toppings pt
ON 
  e.extra_id = pt.topping_id
GROUP BY 
  e.extra_id, pt.topping_name
ORDER BY 
  times_added DESC
LIMIT 1;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/8b8b469f-950f-475c-93f9-8d6e01964c6b)

- The most commonly added extra was Bacon.

****

**3. What was the most common exclusion?**

````sql
WITH most_exclusion AS (
  SELECT 
    UNNEST(STRING_TO_ARRAY(exclusions, ','))::INTEGER AS exclusion_id
  FROM 
    pizza_runner.customer_order_cleaned
  WHERE
    exclusions IS NOT NULL AND exclusions <> ''
)

SELECT 
  e.exclusion_id,
  pt.topping_name AS exclusion_name,
  COUNT(e.exclusion_id) AS times_added
FROM 
  most_exclusion e
JOIN 
  pizza_runner.pizza_toppings pt
ON 
  e.exclusion_id = pt.topping_id
GROUP BY 
  e.exclusion_id, pt.topping_name
ORDER BY 
  times_added DESC
LIMIT 1;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/b211da0d-086a-4504-a35d-6bae2102ccaa)

- The most common exclusion was Cheese.

****

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:
	Meat Lovers
	Meat Lovers - Exclude Beef
	Meat Lovers - Extra Bacon
	Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**

 ````sql
WITH exclusions AS (
    SELECT 
        co.order_id,
        co.pizza_id,
        CAST(SPLIT_PART(co.exclusions, ',', i) AS INTEGER) as topping_id,
        pt.topping_name
    FROM 
        pizza_runner.customer_order_cleaned AS co
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(co.exclusions, ','))) AS i
    JOIN 
        pizza_runner.pizza_toppings AS pt ON pt.topping_id = CAST(SPLIT_PART(co.exclusions, ',', i) AS INTEGER)
    WHERE 
        LENGTH(co.exclusions) > 0 AND co.exclusions <> 'null'
),
extras AS (
    SELECT 
        co.order_id,
        co.pizza_id,
        CAST(SPLIT_PART(co.extras, ',', i) AS INTEGER) as topping_id,
        pt.topping_name
    FROM 
        pizza_runner.customer_order_cleaned AS co
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(co.extras, ','))) AS i
    JOIN 
        pizza_runner.pizza_toppings AS pt ON pt.topping_id = CAST(SPLIT_PART(co.extras, ',', i) AS INTEGER)
    WHERE 
        LENGTH(co.extras) > 0 AND co.extras <> 'null'
),
orders AS (
    SELECT DISTINCT
        co.order_id,
        co.pizza_id,
        unnest(STRING_TO_ARRAY(pr.toppings, ', '))::INTEGER as topping_id
    FROM 
        pizza_runner.customer_order_cleaned AS co
    INNER JOIN 
        pizza_runner.pizza_recipes AS pr ON co.pizza_id = pr.pizza_id
)
, orders_with_extras_and_exclusions AS (
    SELECT
        o.order_id,
        o.pizza_id,
        CASE 
            WHEN o.pizza_id = 1 THEN 'Meat Lovers'
            WHEN o.pizza_id = 2 THEN pn.pizza_name
        END AS pizza, 
        STRING_AGG(DISTINCT ext.topping_name, ', ') AS extras,
        STRING_AGG(DISTINCT exc.topping_name, ', ') AS exclusions
    FROM 
        orders AS o
    LEFT JOIN 
        extras AS ext ON ext.order_id = o.order_id AND ext.pizza_id = o.pizza_id AND ext.topping_id = o.topping_id
    LEFT JOIN 
        exclusions AS exc ON exc.order_id = o.order_id AND exc.pizza_id = o.pizza_id AND exc.topping_id = o.topping_id
    INNER JOIN 
        pizza_runner.pizza_names AS pn ON o.pizza_id = pn.pizza_id
    GROUP BY 
        o.order_id,
        o.pizza_id,
        CASE 
            WHEN o.pizza_id = 1 THEN 'Meat Lovers'
            WHEN o.pizza_id = 2 THEN pn.pizza_name
        END
)

SELECT 
    order_id,
    pizza_id,
    CONCAT(pizza, 
        CASE WHEN exclusions = '' THEN '' ELSE ' - Exclude ' || exclusions END,
        CASE WHEN extras = '' THEN '' ELSE ' - Extra ' || extras END) AS order_item
FROM 
    orders_with_extras_and_exclusions
ORDER BY 
    order_id; 
````


**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/b9d29824-d9d8-48f2-9d37-9ae637fa57b7)

- This has been the most complicated so far but i have learnt a lot of new things solving this.

****

**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**

````sql
WITH EXCLUSIONS AS (
    SELECT 
        order_id,
        pizza_id,
        SPLIT_PART(exclusions, ',', i) as topping_id
    FROM 
        pizza_runner.customer_order_cleaned
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(exclusions, ','))) AS i
    WHERE 
        LENGTH(SPLIT_PART(exclusions, ',', i)) > 0 AND SPLIT_PART(exclusions, ',', i) <> 'null'
),
EXTRAS AS (
    SELECT 
        order_id,
        pizza_id,
        SPLIT_PART(extras, ',', i) as topping_id,
        pt.topping_name
    FROM 
        pizza_runner.customer_order_cleaned
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(extras, ','))) AS i
    JOIN 
        pizza_runner.pizza_toppings AS pt ON pt.topping_id = SPLIT_PART(extras, ',', i)::INTEGER
    WHERE 
        LENGTH(SPLIT_PART(extras, ',', i)) > 0 AND SPLIT_PART(extras, ',', i) <> 'null'
),
ORDERS AS (
    SELECT DISTINCT
        co.order_id,
        co.pizza_id,
        t.topping_id::INTEGER as topping_id,
        pt.topping_name
    FROM 
        pizza_runner.customer_order_cleaned AS co
    INNER JOIN 
        pizza_runner.pizza_recipes AS pr ON co.pizza_id = pr.pizza_id
    CROSS JOIN 
        UNNEST(STRING_TO_ARRAY(pr.toppings, ', ')) AS t(topping_id)
    JOIN 
        pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.topping_id::INTEGER
),
ORDERS_WITH_EXTRAS_AND_EXCLUSIONS AS (
    SELECT 
        O.order_id,
        O.pizza_id,
        O.topping_id::INTEGER as topping_id,
        O.topping_name
    FROM 
        ORDERS AS O
    LEFT JOIN 
        EXCLUSIONS AS EXC ON EXC.order_id = O.order_id AND EXC.pizza_id = O.pizza_id AND EXC.topping_id::INTEGER = O.topping_id 
    WHERE 
        EXC.topping_id IS NULL

    UNION ALL 

    SELECT 
        order_id,
        pizza_id,
        topping_id::INTEGER as topping_id,
        topping_name
    FROM 
        EXTRAS
    WHERE 
        topping_id <> ''
),
TOPPING_COUNT AS (
    SELECT 
        O.order_id,
        O.pizza_id,
        O.topping_name,
        COUNT(*) as n
    FROM 
        ORDERS_WITH_EXTRAS_AND_EXCLUSIONS as O
    GROUP BY 
        O.order_id,
        O.pizza_id,
        O.topping_name
)
SELECT 
    order_id,
    pizza_id,
    STRING_AGG(
        CASE
            WHEN n > 1 THEN CONCAT(n, 'x', topping_name)
            ELSE topping_name
        END, ', ' ORDER BY topping_name) as ingredient
FROM 
    TOPPING_COUNT
GROUP BY 
    order_id,
    pizza_id;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/0c96add5-e7ac-47d7-9bc2-524dab381069)

****

**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

````sql
WITH exclusions AS (
    SELECT 
        order_id,
        pizza_id,
        SPLIT_PART(exclusions, ',', gs)::INTEGER AS topping_id
    FROM 
        pizza_runner.customer_order_cleaned
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(exclusions, ','))) AS gs
    WHERE 
        LENGTH(SPLIT_PART(exclusions, ',', gs)) > 0 
        AND SPLIT_PART(exclusions, ',', gs) <> 'null'
),
extras AS (
    SELECT 
        co.order_id,
        co.pizza_id,
        SPLIT_PART(co.extras, ',', gs)::INTEGER AS topping_id,
        pt.topping_name
    FROM 
        pizza_runner.customer_order_cleaned AS co
    CROSS JOIN 
        generate_series(1, CARDINALITY(STRING_TO_ARRAY(co.extras, ','))) AS gs
    JOIN 
        pizza_runner.pizza_toppings AS pt 
        ON pt.topping_id = SPLIT_PART(co.extras, ',', gs)::INTEGER
    WHERE 
        LENGTH(SPLIT_PART(co.extras, ',', gs)) > 0 
        AND SPLIT_PART(co.extras, ',', gs) <> 'null'
),
orders AS (
    SELECT DISTINCT
        co.order_id,
        co.pizza_id,
        unnest(STRING_TO_ARRAY(pr.toppings, ', '))::INTEGER AS topping_id
    FROM 
        pizza_runner.customer_order_cleaned AS co
    INNER JOIN 
        pizza_runner.pizza_recipes AS pr 
        ON co.pizza_id = pr.pizza_id
),
orders_with_extras_and_exclusions AS (
    SELECT 
        o.order_id,
        o.pizza_id,
        o.topping_id::INTEGER AS topping_id,
        pt.topping_name
    FROM 
        orders AS o
    JOIN 
        pizza_runner.pizza_toppings AS pt 
        ON pt.topping_id = o.topping_id
    LEFT JOIN 
        exclusions AS exc 
        ON exc.order_id = o.order_id 
        AND exc.pizza_id = o.pizza_id 
        AND exc.topping_id = o.topping_id
    WHERE 
        exc.topping_id IS NULL

    UNION ALL 

    SELECT 
        e.order_id,
        e.pizza_id,
        e.topping_id::INTEGER AS topping_id,
        e.topping_name
    FROM 
        extras AS e
    WHERE 
        e.topping_id IS NOT NULL
),
topping_count AS (
    SELECT 
        owe.topping_name,
        COUNT(*) AS quantity
    FROM 
        orders_with_extras_and_exclusions AS owe
    INNER JOIN 
        pizza_runner.runner_orders AS ro 
        ON owe.order_id = ro.order_id
    WHERE 
        ro.pickup_time IS NOT NULL
    GROUP BY 
        owe.topping_name
)
SELECT 
    topping_name,
    SUM(quantity) AS total_quantity
FROM 
    topping_count
GROUP BY 
    topping_name
ORDER BY 
    total_quantity DESC;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/d3e79277-a9f0-42a2-9bf2-3775f5212ce6)

****


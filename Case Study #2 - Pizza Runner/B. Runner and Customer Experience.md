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




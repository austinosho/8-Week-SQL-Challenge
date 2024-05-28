## D. Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

````sql
WITH base_prices AS (
    SELECT 
        c.order_id,
        c.pizza_id,
        CASE 
            WHEN c.pizza_id = 1 THEN 12 
            WHEN c.pizza_id = 2 THEN 10 
        END AS base_price
    FROM 
        pizza_runner.customer_order_cleaned AS c
    JOIN 
        pizza_runner.runner_orders_cleaned AS r ON c.order_id = r.order_id
    WHERE 
        r.distance!= 0 
)
SELECT 
    CONCAT('$', SUM(base_price)) AS total_revenue
FROM 
    base_prices;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/46b880b6-c9bf-409f-9d4f-66194674d207)

- Pizza Runner has made $138 so far.

****

**2. What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra**

````sql
SELECT 
    CONCAT('$', pizza_revenue + topping_revenue) AS total_revenue
FROM (
    SELECT 
        SUM(CASE 
                WHEN pizza_id = 1 THEN 12 
                ELSE 10 
            END) AS pizza_revenue,
        SUM(COALESCE(topping_count, 0)) AS topping_revenue
    FROM (
        SELECT 
            c.order_id,
            c.pizza_id,
            LENGTH(COALESCE(c.extras, '')) - LENGTH(REPLACE(COALESCE(c.extras, ''), ',', '')) + 1 AS topping_count
        FROM 
            pizza_runner.customer_order_cleaned AS c
        INNER JOIN 
            pizza_runner.runner_orders_cleaned AS r ON c.order_id = r.order_id
        WHERE 
            r.distance != 0
    ) AS extras_count
) AS revenue_calculation;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/a0f5029b-4f41-40ab-a00a-1a72d09eb63b)

- With the extra charge, Pizza runner has made $151 so far.

****

**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

````sql
-- Creating the ratings table
CREATE TABLE pizza_runner.runner_ratings (
    order_id INT,
    runner_id INT,
    rating INT CHECK (rating BETWEEN 1 AND 5), -- The rating given by the customer (between 1 and 5)
    comments TEXT, -- Comments provided by the customer (nullable)
    PRIMARY KEY (order_id)
);
-- Inserting sample data
INSERT INTO pizza_runner.runner_ratings (order_id, runner_id, rating, comments) VALUES
(1, 1, 5, 'Excellent service!'),
(2, 1, 4, 'Delivered right on time.'),
(3, 1, 3, NULL), -- No comments provided
(4, 2, 5, 'Very polite and professional.'),
(5, 3, 4, 'Pizza was warm and tasty.'),
(7, 2, 1, 'Arrived late with a cold pizza, horrible experience.'),
(8, 2, 5, 'Impressed with the quick delivery!'),
(10, 1, 4, 'Pizza arrived fresh and delicious.');

SELECT * FROM pizza_runner.runner_ratings;
````

**Output:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/0744cf0e-d453-4506-8ace-d91cbc3e2288)

****

**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas**

````sql
SELECT 
    c.customer_id,
    c.order_id,
    r.runner_id,
    rt.rating,
    c.order_time,
    r.pickup_time,
    ROUND((EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)) / 60)::NUMERIC, 2) AS pick_up_time, 
    r.duration AS delivery_duration,
    ROUND((r.distance * 60 / r.duration)::NUMERIC, 2) AS average_speed, 
    COUNT(c.pizza_id) AS total_pizza_count
FROM 
    pizza_runner.customer_order_cleaned AS c
INNER JOIN 
    pizza_runner.runner_orders_cleaned AS r ON c.order_id = r.order_id
INNER JOIN 
    pizza_runner.runner_ratings AS rt ON c.order_id = rt.order_id
GROUP BY 
    c.customer_id,
    c.order_id,
    r.runner_id,
    rt.rating,
    c.order_time,
    r.pickup_time,
    r.duration,
    r.distance
ORDER BY
    c.order_id;
````

**Output:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/9a9391a5-107f-4af8-917b-4de35a14739f)

****

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

````sql
SELECT CONCAT('$', ROUND(SUM(pizza_cost - delivery_cost), 2)) AS pizza_runner_revenue
FROM (
    SELECT 
        order_id,
        distance,
        SUM(pizza_cost) AS pizza_cost,
        ROUND((0.30 * distance)::numeric, 2) AS delivery_cost
    FROM (
        SELECT 
            c.order_id,
            r.distance,
            (CASE
                WHEN c.pizza_id = 1 THEN 12
                ELSE 10
            END) AS pizza_cost
        FROM 
            pizza_runner.customer_order_cleaned AS c
        INNER JOIN 
            pizza_runner.pizza_names AS pn ON c.pizza_id = pn.pizza_id
        INNER JOIN 
            pizza_runner.runner_orders_cleaned AS r ON c.order_id = r.order_id
        WHERE 
            r.distance != 0
        ORDER BY 
            c.order_id
    ) AS t1
    GROUP BY 
        order_id,
        distance
    ORDER BY 
        order_id
) AS t2;
````

**Answer:**

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/c1a72781-e882-4674-b4b6-f2620616bef4)

- Pizza Runner has $94.44 left.

****

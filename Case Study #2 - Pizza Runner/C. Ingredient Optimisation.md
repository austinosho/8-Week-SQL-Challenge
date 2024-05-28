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


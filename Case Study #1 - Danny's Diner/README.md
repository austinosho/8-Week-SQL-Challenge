# 👨‍🍳Case Study #1 - Danny's Diner https://8weeksqlchallenge.com/case-study-1/
<img src ="https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/c6c9a110-a235-4206-ba49-9e666e4afa40" alt="img" width="600" height="500">

## 📖Table of Contents
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Tools](#tools)
- [Case Study Questions and Solution](#case-study-questions-and-solution)

## Introduction
Danny seriously loves Japanese food so in 2021, Danny opened Danny's Diner that sells his 3 favourite foods: sushi, curry and ramen. Now, he seeks to leverage customer data to understand visiting patterns, spending habits, and menu preferences. This will aid in enhancing the dining experience, fostering loyalty, and potentially expanding the loyalty program. With provided customer data, the task is to extract insights via SQL queries and create basic datasets for easy analysis. The aim is to drive informed decisions and personalized experiences for Danny's Diner patrons.

## Entity Relationship Diagram
![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/a30b0420-4f1c-456b-952e-315bce987b1e)

## 🧑‍💻Tools 
PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

## Case Study Questions and Solution
**1. What is the total amount each customer spent at the restaurant?**
````sql
SELECT sales.customer_id,  
    CONCAT('$', SUM(menu.price)) AS total_amount_spent
FROM dannys_diner.sales 
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A           | $76                |
| B           | $74                |
| C           | $36                |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

#### Query Breakdown:
- The SELECT statement retrieves the customer_id from the sales table and calculates the total amount spent by each customer by summing the prices of items purchased from the menu table.
- The CONCAT('$', ...) function is used to add a dollar sign to the total amount spent, formatting it as a currency value.
- The LEFT JOIN clause is used to combine records from the sales and menu tables based on the product_id column.
- The GROUP BY clause groups the results by customer_id, ensuring that each customer's total spending is aggregated.
- The ORDER BY clause sorts the results by customer_id in ascending order.

***
  
**2. How many days has each customer visited the restaurant?**
````sql
SELECT customer_id, 
	CONCAT(COUNT(DISTINCT order_date), ' days') AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
````
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4 days       |
| B           | 6 days       |
| C           | 2 days       |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

#### Query Breakdown:
- The SQL query selects the customer_id and concatenates the count of distinct order dates with the string 'days' to represent the number of days each customer visited the restaurant.
- The COUNT(DISTINCT order_date) function calculates the number of unique order dates for each customer, indicating the days they visited the restaurant.
- The CONCAT function is used to combine the count of distinct order dates with the string 'days' to form the days_visited column.
- The GROUP BY clause groups the results by customer_id to aggregate the count of distinct order dates for each customer.
- The ORDER BY clause sorts the results by customer_id in ascending order.

***

**3. What was the first item from the menu purchased by each customer?**
````sql
WITH First_Item_Purchased(customer_id, product_order, product_name) AS(
  SELECT sales.customer_id,
  		DENSE_RANK() OVER(PARTITION BY sales.customer_id 
        ORDER BY sales.order_date),
  		menu.product_name
  FROM dannys_diner.sales
  JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id)
  SELECT customer_id,
         product_name
  FROM First_Item_Purchased       
  WHERE product_order = 1
  GROUP BY customer_id,
         product_name
 ;
````
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

- Customer A purchased both curry and sushi as his first order.
- Customer B initial purchase was curry.
- Customer C initial purchase was ramen.
  
#### Query Breakdown:
- The CTE(Common Table Expression) named First_Item_Purchased calculates the rank of each product purchased by each customer based on the order date.
- The use of the DENSE_RANK() window function within the Common Table Expression (CTE) allows for the ranking of products ordered by each customer based on the order date. 
- The SELECT statement outside the CTE retrieves the customer_id and product_name where the product_order is equal to 1, indicating the first item purchased.
- The GROUP BY clause ensures that each customer's first purchased item is distinct.

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
SELECT menu.product_name,
 	CONCAT(COUNT(sales.product_id), ' times') AS no_of_times_purchased
FROM dannys_diner.menu
JOIN dannys_diner.sales
ON menu.product_id = sales.product_id
GROUP BY menu.product_name
ORDER BY no_of_times_purchased desc
LIMIT 1;
````
| product_name | no_of_times_purchased |
| ------------ | --------------------- |
| ramen        | 8 times               |

- ramen is the most purchased product and was purchased 8 times.
  
#### Query Breakdown:
- The SQL query selects the product_name from the menu table and concatenates the count of sales.product_id occurrences with the string 'times' to represent the number of times each item was purchased.
- The JOIN clause combines data from the menu and sales tables based on the matching product_id.
- The COUNT(sales.product_id) function calculates the number of times each product was purchased by counting occurrences of the product_id in the sales table.
- The CONCAT function combines the count of product_id occurrences with the string 'times' to form the no_of_times_purchased column.
- The GROUP BY clause groups the results by product_name to aggregate the count of product_id occurrences for each menu item.
- The ORDER BY clause sorts the results by the count of product_id occurrences in descending order, ensuring the most purchased item appears first.
- The LIMIT 1 clause restricts the output to only the top row, showing the most purchased item.

***

**5. Which item was the most popular for each customer?**
````sql
WITH Item_popularity AS(
SELECT 
    sales.customer_id, 
    menu.product_name,
    COUNT(sales.product_id) AS no_of_times_purchased,
    RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.product_id) DESC) AS ranking
FROM 
    dannys_diner.sales
JOIN 
    dannys_diner.menu ON menu.product_id = sales.product_id
GROUP BY 
    sales.customer_id, menu.product_name)
SELECT customer_id,
       product_name,
       no_of_times_purchased
FROM Item_popularity
WHERE ranking = 1;
````
| customer_id | product_name | no_of_times_purchased |
| ----------- | ------------ | --------------------- |
| A           | ramen        | 3                     |
| B           | ramen        | 2                     |
| B           | curry        | 2                     |
| B           | sushi        | 2                     |
| C           | ramen        | 3                     |

- Customer A can't get enough of ramen!.
- Customer B is the ultimate foodie! Customer B enjoys a bit of everything - ramen, curry, and sushi are all on the menu.
- Customer C is a ramen lover! Customer C's go-to choice at Danny's Diner is always a comforting bowl of ramen.

#### Query Breakdown:
- The CTE (Common Table Expression) named Item_popularity calculates the count of sales.product_id occurrences for each customer and product combination, while also assigning a rank to each product within each customer group based on the count of occurrences using the RANK() window function.
- The SELECT statement outside the CTE retrieves the customer_id, product_name, and no_of_times_purchased columns from the Item_popularity CTE.
- The WHERE clause filters the results to only include rows where the ranking is equal to 1, indicating the most popular item(s) for each customer.

***

**6. Which item was purchased first by the customer after they became a member?**
````sql
WITH item_purchased_ranked AS (
    SELECT 
        members.customer_id,
        menu.product_name,
        sales.order_date,
        members.join_date,
        ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) AS purchase_rank
    FROM 
        dannys_diner.sales
    RIGHT JOIN 
        dannys_diner.members ON sales.customer_id = members.customer_id
    LEFT JOIN 
        dannys_diner.menu ON sales.product_id = menu.product_id
    WHERE 
        sales.order_date > members.join_date)
SELECT 
    customer_id,
    product_name
FROM 
    item_purchased_ranked
WHERE 
    purchase_rank = 1;
````
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

- Customer A first order after becoming a member was ramen
- Customer B first order after becoming a member was sushi
- Customer C is not yet a member

#### Query Breakdown:
- The query utilizes a Common Table Expression (CTE) named item_purchased_ranked to calculate the rank of each purchase made by customers after they became members.
- The CTE joins the sales, members, and menu tables to retrieve customer_id, product_name, order_date, and join_date.
- The ROW_NUMBER() window function is applied within the CTE to assign a unique rank to each purchase within each customer group, based on the order_date.
- The PARTITION BY clause ensures that the ranking is done separately for each customer.
- The WHERE clause filters the results to only include purchases made after the customer became a member, ensuring that only relevant purchases are considered for ranking.
- The main query selects the customer_id and product_name from the item_purchased_ranked CTE where the purchase_rank is equal to 1, indicating the first purchase made by each customer after becoming a member.

***

**7. Which item was purchased just before the customer became a member?**
````sql
WITH item_purchased_ranked AS (
    SELECT 
        members.customer_id,
        menu.product_name,
        sales.order_date,
        members.join_date,
        ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC) AS purchase_rank
    FROM 
        dannys_diner.sales
    RIGHT JOIN 
        dannys_diner.members ON sales.customer_id = members.customer_id
    LEFT JOIN 
        dannys_diner.menu ON sales.product_id = menu.product_id
    WHERE 
        sales.order_date < members.join_date
)
SELECT 
    customer_id,
    product_name
FROM 
    item_purchased_ranked
WHERE 
    purchase_rank = 1;
````
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

- Both Customer A and B have Sushi as their last order before becoming members

#### Query Breakdown:
- The CTE named item_purchased_before_join retrieves the purchases made by customers just before they became members.
- The ROW_NUMBER() window function is used to assign a rank to each purchase within each customer group, ordered by the order_date in descending order (from the latest to the earliest).
- The WHERE clause filters the results to include only purchases made before the join_date for each customer.
- The main query selects the customer_id and product_name for the purchase with the highest rank (i.e., the purchase made just before becoming a member).

****

**8. What is the total items and amount spent for each member before they became a member?**
````sql
SELECT
    sales.customer_id,
    COUNT(sales.product_id) AS total_items,
    CONCAT('$', SUM(MENU.PRICE)) as total_amount
FROM dannys_diner.sales
RIGHT JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
AND sales.order_date < members.join_date
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| A           | 2           | $25          |
| B           | 3           | $40          |

- Customer A spent $25 on two items before becoming a member
- Customer B spent $40 on three items before becoming a member

#### Query Breakdown:
- The SQL query retrieves the total number of items and the total amount spent by each member before they became a member.
- RIGHT JOIN is used to combine data from the sales and members tables based on the customer_id, ensuring that all members are included in the result set.
- The JOIN condition also includes a filter to only consider purchases made before the member's join_date.
- Another LEFT JOIN is used to bring in information about the menu items purchased, linking the product_id between the sales and menu tables.
- The COUNT(sales.product_id) function calculates the total number of items purchased by each member before they became a member.
- The SUM(menu.price) function calculates the total amount spent by each member before they became a member.
- The CONCAT('$', ...) function formats the total amount spent as a currency, adding a dollar sign ('$') to the result.
- The GROUP BY clause groups the results by customer_id to aggregate the total number of items and total amount spent for each member.
- Finally, the results are ordered by customer_id in ascending order to present the information clearly.

****

**9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
````sql
SELECT 
      sales.customer_id,
      SUM(CASE 
             WHEN menu.product_name != 'sushi' THEN menu.price * 10
             WHEN menu.product_name = 'sushi' THEN menu.price * 10 * 2
          END) AS points_earned
FROM dannys_diner.sales
JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | points_earned |
| ----------- | ------------- |
| A           | 860           |
| B           | 940           |
| C           | 360           |

- Customer A earned 860 points
- Customer B earned 940 points
- Customer C earned 360 points

#### Query Breakdown:
- The CASE statement calculates the points earned for each product. For sushi, it multiplies the price by 10 and then by 2 (to apply the 2x points multiplier). For other products, it simply multiplies the price by 10.
- The SUM function then calculates the total points earned for each customer by summing up the points earned for each product.
- The GROUP BY clause groups the results by customer_id to calculate the total points earned for each customer.

****

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
````sql
SELECT 
    sales.customer_id, 
    SUM(
        CASE
            WHEN menu.product_name = 'sushi' THEN menu.price * 10 * 2
            WHEN sales.order_date <= join_date + INTERVAL '6 days' THEN menu.price * 10 * 2
            ELSE menu.price * 10
        END) AS points
FROM 
    dannys_diner.sales
JOIN 
    dannys_diner.members ON sales.customer_id = members.customer_id
JOIN 
    dannys_diner.menu ON sales.product_id = menu.product_id
WHERE 
    sales.order_date <= DATE_TRUNC('month', '2021-01-31'::DATE) + INTERVAL '1   month' - INTERVAL '1 day' AND
    sales.order_date >= members.join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | points |
| ----------- | ------ |
| A           | 1020   |
| B           | 320    |

- For both Customer A and B, from Join date (Day 1) to 6 days after (Day 7), their purchases earned 20 points per $1 spent, including sushi. After Day 7 until the end of January, their purchases continued to earn 10 points per $1 spent, with sushi still earning double points at 20 points per $1 spent.

#### Query Breakdown:
- The query calculates points earned by each customer based on their purchases within the specified timeframes.
- The nested CASE statements handles different earning rates for different time periods and types of items purchased (sushi vs. other items).
- The sales.order_date is compared against the join date (members.join_date) and the end of the first week (members.join_date + INTERVAL '7 days') to determine which earning rate to apply.
- The SUM function aggregates the points earned for each customer.
- Finally, the results are grouped by customer_id to get the total points for each customer.

****

## Bonus Questions

**Join All The Things**

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data
````sql
SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,	
    menu.price,
    CASE 
    	WHEN sales.order_date >= members.join_date THEN 'Y'
      	ELSE 'N' 
    END AS member
FROM 
    dannys_diner.sales
LEFT JOIN 
    dannys_diner.members ON sales.customer_id = members.customer_id
JOIN 
    dannys_diner.menu ON sales.product_id = menu.product_id
 ORDER BY sales.customer_id,
          sales.order_date,
          menu.price desc;
````
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

****

**Rank All The Things**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

````sql
WITH customer_product_ranking AS(
   SELECT
        sales.customer_id,
        sales.order_date,
        menu.product_name,	
        menu.price,
        CASE 
            WHEN sales.order_date >= members.join_date THEN 'Y'
            ELSE 'N' 
        END AS member
    FROM 
        dannys_diner.sales
    LEFT JOIN 
        dannys_diner.members ON sales.customer_id = members.customer_id
    JOIN 
        dannys_diner.menu ON sales.product_id = menu.product_id
    ORDER BY sales.customer_id,
              sales.order_date,
              menu.price desc)
SELECT *,
	CASE
            WHEN member = 'N' THEN NULL
	    ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
        END AS ranking
FROM customer_product_ranking;
````

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | null    |
| B           | 2021-01-02 | curry        | 15    | N      | null    |
| B           | 2021-01-04 | sushi        | 10    | N      | null    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    | 

****

















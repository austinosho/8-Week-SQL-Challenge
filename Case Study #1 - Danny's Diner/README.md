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
#### Query Breakdown:
- The SQL query selects the product_name from the menu table and concatenates the count of sales.product_id occurrences with the string 'times' to represent the number of times each item was purchased.
- The JOIN clause combines data from the menu and sales tables based on the matching product_id.
- The COUNT(sales.product_id) function calculates the number of times each product was purchased by counting occurrences of the product_id in the sales table.
- The CONCAT function combines the count of product_id occurrences with the string 'times' to form the no_of_times_purchased column.
- The GROUP BY clause groups the results by product_name to aggregate the count of product_id occurrences for each menu item.
- The ORDER BY clause sorts the results by the count of product_id occurrences in descending order, ensuring the most purchased item appears first.
- The LIMIT 1 clause restricts the output to only the top row, showing the most purchased item.






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
SELECT * FROM pizza_runner.customer_orders;;
````

Viewing the `customer_orders` table below, we can see that there are null and blank ' ' values in both the `exclusions` and `extras` columns 

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/57a93b9e-13bd-40d4-bcbe-8e62b9e43493)

To clean the table:
  1. We will create a temporary table with all the columns.
  2. Remove null values in `exlusions` and `extras` columns and replace with blank space ' '.

````sql
CREATE TEMP TABLE customer_order_temp AS
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
FROM 
    pizza_runner.customer_orders;
````

View the `customer_order_temp` table
````sql
SELECT * FROM customer_order_temp
````
This is how the cleaned data looks and this table will be used to run all our queries

![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/ea3b22fb-09aa-4a8a-bf38-84ee36bd5c53)





## Case Study Questions and Solution


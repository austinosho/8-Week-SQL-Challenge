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
    SUM(menu.price) AS total_amount_spent
FROM dannys_diner.sales 
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A           | 76                 |
| B           | 74                 |
| C           | 36                 |





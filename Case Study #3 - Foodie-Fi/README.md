# Case Study #3 - Foodie-Fi - https://8weeksqlchallenge.com/case-study-3/
<img src = "https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/a6db00c8-85e9-4c15-9c2f-86d5377fea62" alt="img" width="600" height="500">

## 📖Table of Contents
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Tools](#tools)
- [Case Study Questions and Solution](#case-study-questions-and-solution)

## Introduction
Danny identified a market opportunity for a subscription-based service focused exclusively on food-related content. To address this gap, he launched Foodie-Fi in 2020 with a group of talented friends. Foodie-Fi offers monthly and annual subscriptions, providing customers with unlimited access to a vast collection of exclusive cooking shows and food videos from around the world.
Danny's goal for Foodie-Fi is to make all business decisions based on data. Therefore, he has designed the service with a strong data-driven approach to guide future investments and new features. This case study examines subscription data to answer key business questions and provides insights into the company's performance and customer behavior. The analysis is based on two main tables within the `foodie_fi` database schema, with an additional challenge to create a new table to support the Foodie-Fi team’s data needs.

## Entity Relationship Diagram
![image](https://github.com/austinosho/8-Week-SQL-Challenge/assets/166131518/a8075b9b-385d-48f6-835e-6f71240ce227)

## 🧑‍💻Tools 
PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)

## Case Study Questions and Solution

## A. Customer Journey

**Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!**
````sql
SELECT
  s.customer_id,
  p.plan_id, 
  p.plan_name,  
  s.start_date
FROM foodie_fi.plans p
JOIN foodie_fi.subscriptions s
  ON p.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id, s.start_date;
````

**Result:**
| customer_id | plan_id | plan_name     | start_date               |
| ----------- | ------- | ------------- | ------------------------ |
| 1           | 0       | trial         | 2020-08-01T00:00:00.000Z |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z |
| 2           | 0       | trial         | 2020-09-20T00:00:00.000Z |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z |
| 11          | 0       | trial         | 2020-11-19T00:00:00.000Z |
| 11          | 4       | churn         | 2020-11-26T00:00:00.000Z |
| 13          | 0       | trial         | 2020-12-15T00:00:00.000Z |
| 13          | 1       | basic monthly | 2020-12-22T00:00:00.000Z |
| 13          | 2       | pro monthly   | 2021-03-29T00:00:00.000Z |
| 15          | 0       | trial         | 2020-03-17T00:00:00.000Z |
| 15          | 2       | pro monthly   | 2020-03-24T00:00:00.000Z |
| 15          | 4       | churn         | 2020-04-29T00:00:00.000Z |
| 16          | 0       | trial         | 2020-05-31T00:00:00.000Z |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z |
| 16          | 3       | pro annual    | 2020-10-21T00:00:00.000Z |
| 18          | 0       | trial         | 2020-07-06T00:00:00.000Z |
| 18          | 2       | pro monthly   | 2020-07-13T00:00:00.000Z |
| 19          | 0       | trial         | 2020-06-22T00:00:00.000Z |
| 19          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z |
| 19          | 3       | pro annual    | 2020-08-29T00:00:00.000Z |

### Customer Onboarding Journeys

1. **Customer 1**
   - Started with a trial plan
   - Loved it and upgraded to the basic monthly plan just a week later.
2. **Customer 2**
   - Started with a trial plan
   - Went all-in by upgrading to the pro annual plan within a week.
3. **Customer 11**
   - Gave it a shot with a trial plan.
   - Decided it wasn’t their thing and churned on November 26, 2020, without committing to a paid plan.
4. **Customer 13**
   - Started sampling with a trial plan.
   - Found a favorite and upgraded to the basic monthly plan on December 22, 2020.
   - Craved more variety and moved up to the pro monthly plan on March 29, 2021.
5. **Customer 15**
   - Gave it a trial.
   - Quickly upgraded to the pro monthly plan on March 24, 2020.
   - Lost interest and churned on April 29, 2020.
6. **Customer 16**
   - Sampled with a trial plan.
   - Committed with the basic monthly plan on June 7, 2020.
   - Opted for the better experience with the pro annual plan on October 21, 2020.
7. **Customer 18**
   - Started with a trial plan.
   - Decided on the premium selection and upgraded to the pro monthly plan on July 13, 2020.
8. **Customer 19**
   - Began their journey with a trial.
   - Quickly chose premium monthly access on June 29, 2020.
     Settled for a long-term commitment with the pro annual plan on August 29, 2020.

****

## B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**
````sql
SELECT
  COUNT(DISTINCT(customer_id)) AS no_of_customers
FROM foodie_fi.subscription
````

**Answer:**
| no_of_customers |
| --------------- |
| 1000            |

- The total number of customers Foodie-Fi has had is 1,000.

****

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**
````sql
WITH trial_subscriptions AS (
  SELECT
    DATE_PART('month', start_date) AS month_date,
    customer_id
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
)
SELECT
  month_date,
  COUNT(customer_id) AS trial_plan_subscriptions
FROM trial_subscriptions
GROUP BY month_date
ORDER BY month_date;
````

**Answer:**
| month_date | trial_plan_subscriptions |
| ---------- | ------------------------ |
| 1          | 88                       |
| 2          | 68                       |
| 3          | 94                       |
| 4          | 81                       |
| 5          | 88                       |
| 6          | 79                       |
| 7          | 89                       |
| 8          | 88                       |
| 9          | 87                       |
| 10         | 79                       |
| 11         | 75                       |
| 12         | 84                       |

- March has the highest number of trial plans with 94 trial plans, while February has the lowest number of trial plans with 68 trial plans.


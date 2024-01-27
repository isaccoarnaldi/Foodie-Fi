# 🥑 Case Study #3: Foodie-Fi

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Dataset](#dataset)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-3/). 

***

## Business Task
Danny launched Foodie-Fi in 2020, a streaming service exclusively for cooking shows with monthly and annual subscriptions. The startup, founded with a data-driven approach, focuses on using subscription-style digital data for key business insights related to customer journeys, payments, and overall performance. 

This case study focuses on using subscription-style digital data to answer various business questions.

## Dataset


**Table 1: `plans`**  

<img width="207" alt="image" src="https://user-images.githubusercontent.com/81607668/135704535-a82fdd2f-036a-443b-b1da-984178166f95.png"> 		

*
**Table 2: `subscriptions`**

<img width="261" alt="Screenshot 2021-08-17 at 11 36 10 PM" src="https://user-images.githubusercontent.com/81607668/129756709-75919d79-e1cd-4187-a129-bdf90a65e196.png">

Foodie-Fi offers 5 customer plans:

- Trial: A 7-day free trial transitions to the pro monthly subscription plan unless canceled, downgraded to basic, or upgraded to an annual pro plan during the trial.
- Basic Plan: Monthly access at $9.90 with limited streaming capabilities.
- Pro Plan: Monthly subscription at $19.90 or $199 annually, offering unlimited streaming and offline video downloads.

Cancellation results in a Churn plan record with null pricing, active until the billing period ends. Subscription `start_date` indicates plan initiation. Downgrades or cancellations maintain the current plan until the period concludes. Upgrades take immediate effect. Churning allows access until the billing period ends, with the start date set on the cancellation day.

***

## Question and Solution



## 🎞️ A. Customer Journey

Based on the 8 sample customers provided in the `subscriptions` table, write a brief description of each customer’s onboarding journey.

**Answer:**

```sql
SELECT
      s.*,
      p.plan_name,
FROM foodie_fi.plans AS p
JOIN foodie_fi.subscriptions AS s
     ON p.plan_id = sub.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19);
```

| customer\_id | plan\_id | start\_date |  plan\_name   |
| ------------ | -------- | ----------- | ------------- |
| 1            | 0        | 2020-08-01  |     trial     |
| 1            | 1        | 2020-08-08  | basic monthly |
| 2            | 0        | 2020-09-20  |     trial     |
| 2            | 3        | 2020-09-27  |   pro annual  |
| 11           | 0        | 2020-11-19  |     trial     |
| 11           | 4        | 2020-11-26  |     churn     |
| 13           | 0        | 2020-12-15  |     trial     |
| 13           | 1        | 2020-12-22  | basic monthly |
| 13           | 2        | 2021-03-29  |   pro monthly |
| 15           | 0        | 2020-03-17  |     trial     |
| 15           | 2        | 2020-03-24  |   pro monthly |
| 15           | 4        | 2020-04-29  |     churn     |
| 16           | 0        | 2020-05-31  |     trial     |
| 16           | 1        | 2020-06-07  | basic monthly |
| 16           | 3        | 2020-10-21  |   pro annual  |
| 18           | 0        | 2020-07-06  |     trial     |
| 18           | 2        | 2020-07-13  |   pro monthly |
| 19           | 0        | 2020-06-22  |     trial     |
| 19           | 2        | 2020-06-29  |   pro monthly |
| 19           | 3        | 2020-08-29  |   pro annual  |

Based on the findings, three highlighted customers illustrate distinct onboarding journeys:

- Customer 1: Initiated a free trial on 1 Aug 2020, subscribing to the basic monthly plan on 8 Aug 2020 after the trial concluded.

- Customer 13: Started with a free trial on 15 Dec 2020, transitioning to the basic monthly plan on 22 Dec 2020. After three months, they upgraded to the pro monthly plan on 29 Mar 2021.

- Customer 15: Began with a free trial on 17 Mar 2020, upgraded to the pro monthly plan on 24 Mar 2020. However, on 29 Apr 2020, the customer chose to terminate the subscription, resulting in churn until the paid subscription concludes.


***

## B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

```sql
SELECT COUNT(DISTINCT s.customer_id) AS distinct_customers
FROM foodie_fi.subscriptions AS s;
```

**Answer:**
| distinct_customers|
| ----------------- | 
| 1000              | 

### 2. What is the monthly distribution of trial plan `start_date` values for our dataset - use the start of the month as the group by value

- Extracted the numerical value of the month from the `start_date` column using the `DATE_TRUNC()` function
- Filtered the results to include solely users with trial plan subscriptions (`plan_id = 0).

```sql
SELECT
  DATE_TRUNC('month', s.start_date) AS month,
  COUNT(DISTINCT s.customer_id) AS count
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id = 0 -- Trial plan ID is 0
GROUP BY 1
ORDER BY month;
```

**Answer:**
| month      | count |
| ---------- | ----- |
| 2020-01-01 | 88    |
| 2020-02-01 | 68    |
| 2020-03-01 | 94    |
| 2020-04-01 | 81    |
| 2020-05-01 | 88    |
| 2020-06-01 | 79    |
| 2020-07-01 | 89    |
| 2020-08-01 | 88    |
| 2020-09-01 | 87    |
| 2020-10-01 | 79    |
| 2020-11-01 | 75    |
| 2020-12-01 | 84    |

March has the highest number of trial plans, while February has the lowest number of trial plans.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

To put it simply, we have to determine the count of plans with start dates on or after 1 January 2021 grouped by plan names. 
1. Filter plans based on their start dates by including only the plans occurring on or after January 1, 2021.
2. Calculate the number of customers as the number of events. 
3. Group results based on the plan names. For better readability, order results in ascending order of the plan ID. 

````sql
SELECT 
  plans.plan_id,
  plans.plan_name,
  COUNT(sub.customer_id) AS num_of_events
FROM foodie_fi.subscriptions AS sub
JOIN foodie_fi.plans
  ON sub.plan_id = plans.plan_id
WHERE sub.start_date >= '2021-01-01'
GROUP BY plans.plan_id, plans.plan_name
ORDER BY plans.plan_id;
````

**Answer:**

| plan_id | plan_name     | num_of_events |
| ------- | ------------- | ------------- |
| 1       | basic monthly | 8             |
| 2       | pro monthly   | 60            |
| 3       | pro annual    | 63            |
| 4       | churn         | 71            |

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

Let's analyze the question:
- First, we need to determine
  - The number of customers who have churned, meaning those who have discontinued their subscription.
  - The total number of customers, including both active and churned ones.

- To calculate the churn rate, we divide the number of churned customers by the total number of customers. The result should be rounded to one decimal place.

```sql
SELECT
  COUNT(DISTINCT sub.customer_id) AS churned_customers,
  ROUND(100.0 * COUNT(sub.customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
    	FROM foodie_fi.subscriptions)
  ,1) AS churn_percentage
FROM foodie_fi.subscriptions AS sub
JOIN foodie_fi.plans
  ON sub.plan_id = plans.plan_id
WHERE plans.plan_id = 4; -- Filter results to customers with churn plan only
```

**Answer:**

<img width="368" alt="image" src="https://user-images.githubusercontent.com/81607668/129840630-adebba8c-9219-4816-bba6-ba8119f298d9.png">

- Out of the total customer base of Foodie-Fi, 307 customers have churned. This represents approximately 30.7% of the overall customer count.

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

Within a CTE called `ranked_cte`, determine which customers churned immediately after the trial plan by utilizing `ROW_NUMBER()` function to assign rankings to each customer's plans. 

In this scenario, if a customer churned right after the trial plan, the plan rankings would appear as follows:
- Trial Plan - Rank 1
- Churned - Rank 2

In the outer query:
- Apply 2 conditions in the WHERE clause:
  - Filter `plan_id = 4`. 
  - Filter for customers who have churned immediately after their trial with `row_num = 2`.
- Count the number of customers who have churned immediately after their trial period using a `CASE` statement by checking if the row number is 2 (`row_num = 2`) and the plan name is 'churn' (`plan_name = 'churn'`). 
- Calculate the churn percentage by dividing the `churned_customers` count by the total count of distinct customer IDs in the `subscriptions` table. Round percentage to a whole number.

```sql
WITH ranked_cte AS (
  SELECT 
    sub.customer_id, 
    plans.plan_id, 
	  ROW_NUMBER() OVER (
      PARTITION BY sub.customer_id 
      ORDER BY sub.start_date) AS row_num
  FROM foodie_fi.subscriptions AS sub
  JOIN foodie_fi.plans 
    ON sub.plan_id = plans.plan_id
)
  
SELECT 
	COUNT(CASE 
    WHEN row_num = 2 AND plan_name = 'churn' THEN 1 
    ELSE 0 END) AS churned_customers,
	ROUND(100.0 * COUNT(
    CASE 
      WHEN row_num = 2 AND plan_name = 'churn' THEN 1 
      ELSE 0 END) 
	  / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  ) AS churn_percentage
FROM ranked_cte
WHERE plan_id = 4 -- Filter to churn plan.
  AND row_num = 2; -- Customers who have churned immediately after trial have churn plan ranked as 2.
```

Here's another solution using the `LEAD()` window function:
```sql
WITH ranked_cte AS (
  SELECT 
    sub.customer_id,  
    plans.plan_name, 
	  LEAD(plans.plan_name) OVER ( 
      PARTITION BY sub.customer_id
      ORDER BY sub.start_date) AS next_plan
  FROM foodie_fi.subscriptions AS sub
  JOIN foodie_fi.plans 
    ON sub.plan_id = plans.plan_id
)
  
SELECT 
  COUNT(customer_id) AS churned_customers,
  ROUND(100.0 * 
    COUNT(customer_id) 
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  ) AS churn_percentage
FROM ranked_cte
WHERE plan_name = 'trial' 
  AND next_plan = 'churn;
```

**Answer:**

<img width="378" alt="image" src="https://user-images.githubusercontent.com/81607668/129834269-98ab360b-985a-4c25-9d42-c89b97ba6ba8.png">

- A total of 92 customers churned immediately after the initial free trial period, representing approximately 9% of the entire customer base.

### 6. What is the number and percentage of customer plans after their initial free trial?

```sql
WITH next_plans AS (
  SELECT 
    customer_id, 
    plan_id, 
    LEAD(plan_id) OVER(
      PARTITION BY customer_id 
      ORDER BY plan_id) as next_plan_id
  FROM foodie_fi.subscriptions
)

SELECT 
  next_plan_id AS plan_id, 
  COUNT(customer_id) AS converted_customers,
  ROUND(100 * 
    COUNT(customer_id)::NUMERIC 
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  ,1) AS conversion_percentage
FROM next_plans
WHERE next_plan_id IS NOT NULL 
  AND plan_id = 0
GROUP BY next_plan_id
ORDER BY next_plan_id;
```

**Answer:**

| plan_id | converted_customers | conversion_percentage |
| ------- | ------------------- | --------------------- |
| 1       | 546                 | 54.6                  |
| 2       | 325                 | 32.5                  |
| 3       | 37                  | 3.7                   |
| 4       | 92                  | 9.2                   |

- More than 80% of Foodie-Fi's customers are on paid plans with a majority opting for Plans 1 and 2. 
- There is potential for improvement in customer acquisition for Plan 3 as only a small percentage of customers are choosing this higher-priced plan.

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

In the cte called `next_dates`, we begin by filtering the results to include only the plans with start dates on or before '2020-12-31'. To identify the next start date for each plan, we utilize the `LEAD()` window function.

In the outer query,  we filter the results where the `next_date` is NULL. This step helps us identify the most recent plan that each customer subscribed to as of '2020-12-31'. 

Lastly, we perform calculations to determine the total count of customers and the percentage of customers associated with each trial plan. 

```sql
WITH next_dates AS (
  SELECT
    customer_id,
    plan_id,
  	start_date,
    LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_date
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
	plan_id, 
	COUNT(DISTINCT customer_id) AS customers,
  ROUND(100.0 * 
    COUNT(DISTINCT customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  ,1) AS percentage
FROM next_dates
WHERE next_date IS NULL
GROUP BY plan_id;
```

**Answer:**

<img width="448" alt="image" src="https://user-images.githubusercontent.com/81607668/130024738-f16ad7dc-5fed-469f-9c6d-0a24453e1dcd.png">

### 8. How many customers have upgraded to an annual plan in 2020?

```sql
SELECT COUNT(DISTINCT customer_id) AS num_of_customers
FROM foodie_fi.subscriptions
WHERE plan_id = 3
  AND start_date <= '2020-12-31';
```

**Answer:**

<img width="160" alt="image" src="https://user-images.githubusercontent.com/81607668/129848711-3b64442a-5724-4723-bea7-e4515a8687ec.png">

- 196 customers have upgraded to an annual plan in 2020.

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

This question is straightforward and the query provided is self-explanatory. 

````sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
), annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
)
-- Find the average of the differences between the start date of a trial plan and a pro annual plan.
SELECT 
  ROUND(
    AVG(
      annual.annual_date - trial.trial_date)
  ,0) AS avg_days_to_upgrade
FROM trial_plan AS trial
JOIN annual_plan AS annual
  ON trial.customer_id = annual.customer_id;
````

**Answer:**

<img width="182" alt="image" src="https://user-images.githubusercontent.com/81607668/129856015-4bafa22c-b732-4c71-93d6-c9417e8556b9.png">

- On average, customers take approximately 105 days from the day they join Foodie-Fi to upgrade to an annual plan.

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

To understand how the `WIDTH_BUCKET()` function works in creating buckets of 30-day periods, you can refer to this [StackOverflow](https://stackoverflow.com/questions/50518548/creating-a-bin-column-in-postgres-to-check-an-integer-and-return-a-string) answer.

```sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
), annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
), bins AS (
-- bins CTE: Put customers in 30-day buckets based on the average number of days taken to upgrade to a pro annual plan.
  SELECT 
    WIDTH_BUCKET(annual.annual_date - trial.trial_date, 0, 365, 12) AS avg_days_to_upgrade
  FROM trial_plan AS trial
  JOIN annual_plan AS annual
    ON trial.customer_id = annual.customer_id
)
  
SELECT 
  ((avg_days_to_upgrade - 1) * 30 || ' - ' || avg_days_to_upgrade * 30 || ' days') AS bucket, 
  COUNT(*) AS num_of_customers
FROM bins
GROUP BY avg_days_to_upgrade
ORDER BY avg_days_to_upgrade;
```

**Answer:**

| bucket         | num_of_customers |
| -------------- | ---------------- |
| 0 - 30 days    | 49               |
| 30 - 60 days   | 24               |
| 60 - 90 days   | 35               |
| 90 - 120 days  | 35               |
| 120 - 150 days | 43               |
| 150 - 180 days | 37               |
| 180 - 210 days | 24               |
| 210 - 240 days | 4                |
| 240 - 270 days | 4                |
| 270 - 300 days | 1                |
| 300 - 330 days | 1                |
| 330 - 360 days | 1                |

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
WITH ranked_cte AS (
  SELECT 
    sub.customer_id,  
  	plans.plan_id,
    plans.plan_name, 
	  LEAD(plans.plan_id) OVER ( 
      PARTITION BY sub.customer_id
      ORDER BY sub.start_date) AS next_plan_id
  FROM foodie_fi.subscriptions AS sub
  JOIN foodie_fi.plans 
    ON sub.plan_id = plans.plan_id
 WHERE DATE_PART('year', start_date) = 2020
)
  
SELECT 
  COUNT(customer_id) AS churned_customers
FROM ranked_cte
WHERE plan_id = 2
  AND next_plan_id = 1;
```

**Answer:**

In 2020, there were no instances where customers downgraded from a pro monthly plan to a basic monthly plan.

***

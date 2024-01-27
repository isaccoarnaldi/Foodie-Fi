# 🥑 Case Study #3: Foodie-Fi
<p align="center">
<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">
</p>
	
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

#
**Table 2: `subscriptions`**

<img width="261" alt="Screenshot 2021-08-17 at 11 36 10 PM" src="https://user-images.githubusercontent.com/81607668/129756709-75919d79-e1cd-4187-a129-bdf90a65e196.png">

Foodie-Fi offers 5 customer plans:

- Trial: A 7-day free trial transitions to the pro monthly subscription plan unless canceled, downgraded to basic, or upgraded to an annual pro plan during the trial.
- Basic Plan: Monthly access at $9.90 with limited streaming capabilities.
- Pro Plan: Monthly subscription at $19.90 or $199 annually, offering unlimited streaming and offline video downloads.

Cancellation results in a Churn plan record with null pricing, active until the billing period ends. Subscription `start_date` indicates plan initiation. Downgrades or cancellations maintain the current plan until the period concludes. Upgrades take immediate effect. Churning allows access until the billing period ends, with the start date set on the cancellation day.

***

## Question and Solution



# A. Customer Journey
Based on the 8 sample customers provided in the `subscriptions` table, write a brief description of each customer’s onboarding journey.

**Answer:**
```sql
SELECT s.customer_id,
       s.plan_id, 
       TO_CHAR(s.start_date, 'YYYY-MM-DD') AS start_date,
       p.plan_name
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE s.customer_id BETWEEN 1 AND 19
ORDER BY s.customer_id, p.plan_id;
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

- Extracted the numerical value of the month from the `start_date` column using the `DATE_TRUNC()` function and `TO_CHAR()` to remove timestamps
- Filtered the results to include solely users with trial plan subscriptions (`plan_id = 0`)

```sql
SELECT TO_CHAR(DATE_TRUNC('month', s.start_date), 'YYYY-MM-DD') AS month,
       COUNT(DISTINCT s.customer_id) AS count
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id = 0 -- Trial plan ID is 0
GROUP BY 1 -- 1 is first column, same as month
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


### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

- Counted the number of customers as the number of events.
- Filtered plans occurring after the year 2020.
- Grouped results based on non-aggregate elements. 

````sql
SELECT p.plan_id,
       p.plan_name,
       COUNT(s.customer_id) AS num_of_events
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
  ON s.plan_id = p.plan_id
WHERE DATE_PART('year', s.start_date) > 2020
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;
````

**Answer:**

| plan_id | plan_name     | num_of_events |
| ------- | ------------- | ------------- |
| 1       | basic monthly | 8             |
| 2       | pro monthly   | 60            |
| 3       | pro annual    | 63            |
| 4       | churn         | 71            |

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

- Identified the count of customers who have churned, indicating those who terminated their subscription.
- Identified the overall number of customers, encompassing both active and churned individuals.
- In computing the churn rate, the calculation involved dividing the number of churned customers by the total customer count. The outcome was rounded to one decimal place.

```sql
SELECT COUNT(DISTINCT s.customer_id) AS churn_count,
       ROUND(100.0 * COUNT(s.customer_id)
     / (SELECT COUNT(DISTINCT customer_id) 
    	FROM foodie_fi.subscriptions)
      , 1) AS churn_percentage
FROM foodie_fi.subscriptions AS s
WHERE s.plan_id = 4; -- Filter results to customers with churn plan only
```

**Answer:**
| churn\_count | churn\_percentage |
| -------------| ----------------- |
| 307          | 30.7              |


### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

- Utilized Common Table Expression (CTE) named ranked_cte.
	- Used `DENSE_RANK`() to assign rankings to customer plans: Trial Plan - Rank 1, Churned - Rank 2.
- WHERE clause conditions:
	- Filtered for `plan_id = 4`.
	- Identified churn after trial with `rank_num = 2`.
- Counted churned customers using `CASE` statement:
	- Checked `rank_num = 2` and `plan_name = 'churn'`.
- Calculated churn percentage:
	- Divided count of churned customers by total distinct customer IDs.
	- Rounded percentage to whole number.

```sql
WITH ranked_cte AS (
    SELECT s.customer_id, 
           p.plan_id,
           p.plan_name, 
	   DENSE_RANK() OVER (PARTITION BY s.customer_id 
           ORDER BY s.start_date) AS rank_num
    FROM foodie_fi.subscriptions AS s
    JOIN foodie_fi.plans AS p 
    ON s.plan_id = p.plan_id
)
  
SELECT 
    COUNT(CASE WHEN rank_num = 2 AND plan_name = 'churn' THEN 1 
    ELSE 0 END) AS churned_customers,

    ROUND(100.0 * COUNT(CASE WHEN rank_num = 2 AND plan_name = 'churn' THEN 1 
                       ELSE 0 END) 
          / (SELECT COUNT(DISTINCT customer_id) 
             FROM foodie_fi.subscriptions)
          , 0) AS percentage
FROM ranked_cte
WHERE plan_id = 4 -- Filter to churn plan.
AND rank_num = 2; -- Customers who have churned immediately after trial have churn plan ranked as 2.
```

**Answer:**
| churned_customers | percentage |
| ----------------- | ---------- |
| 92                | 9          |

### 6. What is the number and percentage of customer plans after their initial free trial?

```sql
WITH next_plans AS (
                   SELECT s.customer_id, 
                          s.plan_id, 
                          LEAD(s.plan_id) OVER(PARTITION BY s.customer_id 
                                        ORDER BY s.plan_id) AS next_plan_id
                   FROM foodie_fi.subscriptions s
                   )

SELECT next_plan_id AS plan_id, 
       COUNT(customer_id) AS converted_customers,
         ROUND(100 * COUNT(customer_id)::NUMERIC 
      / (SELECT COUNT(DISTINCT customer_id)
         FROM foodie_fi.subscriptions)
      , 1) AS conversion_percentage
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

```sql
WITH cte AS(
           SELECT s.*,
                  ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date DESC) AS rank
           FROM foodie_fi.subscriptions s
           WHERE s.start_date <= '2020-12-31'
)
SELECT plan_id,
       COUNT(DISTINCT customer_id) AS number_of_customer,
       ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER(), 1) AS percentage
FROM cte
WHERE rank = 1
GROUP BY plan_id
```

**Answer:**
| plan\_id | number\_of\_customer | percentage |
| -------- | -------------------- | ---------- |
| 0        | 19                   | 1.9        |
| 1        | 224                  | 22.4       |
| 2        | 326                  | 32.6       |
| 3        | 195                  | 19.5       |
| 4        | 236                  | 23.6       |


### 8. How many customers have upgraded to an annual plan in 2020?

```sql
SELECT COUNT(DISTINCT s.customer_id) AS num_of_customers
FROM foodie_fi.subscriptions s
WHERE plan_id = 3
  AND start_date <= '2020-12-31';
```

**Answer:**
| num_of_customers |  
|------------------|
| 195              |  

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

````sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
     SELECT s.customer_id, 
            s.start_date AS trial_date
     FROM foodie_fi.subscriptions s
     WHERE s.plan_id = 0
),
annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
     SELECT s.customer_id, 
            s.start_date AS annual_date
     FROM foodie_fi.subscriptions s
     WHERE s.plan_id = 3
)
-- Find the average of the differences between the start date of a trial plan and a pro annual plan.
SELECT ROUND(AVG(annual.annual_date - trial.trial_date), 0) AS avg_days_to_upgrade
FROM trial_plan AS trial
JOIN annual_plan AS annual
  ON trial.customer_id = annual.customer_id;
````

**Answer:**
| average_days_to_upgrade|  
|------------------------|
| 105                    |

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

To understand how the `WIDTH_BUCKET()` function works in creating buckets of 30-day periods, refer to this [StackOverflow](https://stackoverflow.com/questions/50518548/creating-a-bin-column-in-postgres-to-check-an-integer-and-return-a-string) answer.

```sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
     SELECT s.customer_id, 
            s.start_date AS trial_date
     FROM foodie_fi.subscriptions s
     WHERE s.plan_id = 0
),
annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
    SELECT s.customer_id, 
           s.start_date AS annual_date
    FROM foodie_fi.subscriptions s
    WHERE s.plan_id = 3
),
bins AS (
-- bins CTE: Put customers in 30-day buckets based on the average number of days taken to upgrade to a pro annual plan.
   SELECT WIDTH_BUCKET(annual.annual_date - trial.trial_date, 0, 365, 12) AS avg_days_to_upgrade
   FROM trial_plan AS trial
   JOIN annual_plan AS annual
    ON trial.customer_id = annual.customer_id
)
  
SELECT ((avg_days_to_upgrade - 1) * 30 || ' - ' || avg_days_to_upgrade * 30 || ' days') AS bucket, 
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
     SELECT s.customer_id,  
            p.plan_id,
            p.plan_name, 
	    LEAD(p.plan_id) OVER (PARTITION BY s.customer_id
                              ORDER BY s.start_date) AS next_plan_id
     FROM foodie_fi.subscriptions AS s
     JOIN foodie_fi.plans p
       ON s.plan_id = p.plan_id
     WHERE DATE_PART('year', start_date) = 2020
)
  
SELECT COUNT(customer_id) AS churned_customers
FROM ranked_cte
WHERE plan_id = 2
  AND next_plan_id = 1;
```

**Answer:**
|churned_customers |  
|------------------|
| 0                |

In 2020, there were no instances where customers downgraded from a pro monthly plan to a basic monthly plan.

***

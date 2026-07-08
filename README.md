# FOODIE-FI CASE STUDY
## Background
Recognizing the growing popularity of subscription-based businesses and a gap in the market for food-focused content, Danny launched Foodie-Fi in 2020 alongside a few close friends — a Netflix-style streaming platform dedicated exclusively to cooking shows, offering customers unlimited on-demand access to exclusive food videos from around the world through monthly and annual subscriptions. Built with a data-driven mindset from the start, Foodie-Fi relies on subscription data analysis to guide all future investment decisions and feature development, making this case study a exploration of how subscription-style digital data can be used to answer key business questions.

## Problem Statement
Danny launched Foodie-Fi in 2020 — a subscription-based streaming service focused exclusively on food-related content, offering customers unlimited on-demand access to exclusive cooking and food videos from around the world through monthly and annual subscription plans.

Since its inception, Danny has been committed to running Foodie-Fi with a data-driven mindset, aiming to base all future investment decisions and new feature development on solid data analysis rather than guesswork.

The challenge, therefore, is to analyze Foodie-Fi's subscription-style digital data in order to answer key business questions related to:

- Customer subscription behavior 
- Revenue and growth trends across monthly and annual plans
- Customer lifetime value and retention
- Plan performance — understanding which subscription tiers are most popular or profitable
- Providing actionable insights that can guide Foodie-Fi's future business strategy, marketing decisions, and product developmen

## SKILLS APPLIED
- Window Functions
- CTEs
- Aggregations
- JOINs
- Write scripts to generate basic reports that can be run every period
## Questions Explored
1. How many customers has Foodie-Fi ever had?
2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
6. What is the number and percentage of customer plans after their initial free trial?
7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
8. How many customers have upgraded to an annual plan in 2020?
9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

## Some interesting queries
### Q7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```
WITH cte1 AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS ranking
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
),
cte2 AS (
  SELECT p.plan_name, COUNT(cte1.plan_id) AS num_plans
  FROM cte1 
  INNER JOIN plans p ON cte1.plan_id = p.plan_id
  WHERE cte1.ranking = 1
  GROUP BY p.plan_name
)
SELECT plan_name, num_plans,
       ROUND((num_plans * 100.0) / SUM(num_plans) OVER (), 2) AS pct_plans;
```

### Q6. What is the number and percentage of customer plans after their initial free trial?
```
WITH customerFirstPaidPlan AS (
SELECT customer_id, MIN(plan_id) AS first_paid_plan_id
FROM subscriptions
WHERE plan_id !=0
GROUP BY customer_id
)
SELECT p.plan_name,
       COUNT(DISTINCT c.customer_id) AS customer_count,
       ROUND(
         COUNT(DISTINCT c.customer_id) * 100.0 
         / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 
         0
       ) AS percentage
FROM customerFirstPaidPlan c
JOIN plans p ON c.first_paid_plan_id = p.plan_id
GROUP BY p.plan_name;
```
### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place.

```
SELECT 
    plan_name,
    COUNT(DISTINCT sp.customer_id) AS count_of_churned,
    ROUND(COUNT(DISTINCT sp.customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100, 1) AS churn_percentage
FROM subscriptions AS sp
JOIN plans ON sp.plan_id = plans.plan_id
WHERE plan_name = 'churn'
GROUP BY plan_name;
```
## Insights
- 614 members churned straight after their initial free trial
- 1000 members is what Foodie-Fi has ever had
- 

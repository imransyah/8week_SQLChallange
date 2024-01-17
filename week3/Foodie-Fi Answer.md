Let's see with the dataset we will work on

```sql
SELECT *
FROM foodie_fi.plans
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/plans.PNG)

```sql
SELECT *
FROM foodie_fi.subscriptions
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/subscriptions.PNG)

Seems we got no problem with these dataset
Now we can work on the challange, Let's do this!


How many customers has Foodie-Fi ever had?

```sql
SELECT COUNT(DISTINCT customer_id) AS total_customer
FROM foodie_fi.subscriptions
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/1.PNG)

What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
WITH dist AS (
		SELECT EXTRACT(YEAR FROM start_date) AS YY,
			to_char(start_date, 'MM') AS mm,
			to_char(start_date, 'Month') AS mm_name,
			customer_id AS trial
		FROM 
			foodie_fi.subscriptions sub
		WHERE plan_id = 0
)
SELECT 
	YY,
	mm,
	mm_name,
	COUNT(trial)
FROM
	dist
GROUP BY
	YY,
	mm,
	mm_name
ORDER BY 
	YY ASC,
	mm ASC
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/2.PNG)

What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
WITH extraction AS (
		SELECT
				EXTRACT(YEAR FROM start_date) AS yy,
				sub.plan_id,
				pl.plan_name,
				customer_id AS total
		FROM foodie_fi.subscriptions sub
		JOIN foodie_fi.plans AS pl
		ON sub.plan_id = pl.plan_id
)
,
extraction_and_summary AS (
		SELECT 
				yy,
				plan_id,
				plan_name,
				COUNT(total) AS total_plan
		FROM
				extraction
		GROUP BY 
				1,
				2,
				3
)
,
extraction_and_summary_partitioned AS (
		SELECT 
				*, 
				SUM(total_plan)OVER(
						PARTITION BY yy
					) AS totalper_year
		FROM extraction_and_summary
)
SELECT *,
		round(total_plan::NUMERIC / totalper_year * 100, 2) AS percentage_differ
FROM
		extraction_and_summary_partitioned
WHERE 
		yy = '2021';
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/3.PNG)

What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
WITH total AS (
SELECT 
	(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions) AS total_customer,
	COUNT(DISTINCT customer_id) AS total_churned
FROM foodie_fi.subscriptions
WHERE plan_id = 4
)
SELECT 
total_customer,
total_churned,
round(total_churned*100)/total_customer AS percentage
FROM total
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/4.PNG)

How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH CTE AS ( 
SELECT
	customer_id,
	plan_name,
	ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS rn
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
)
,
CTE2 AS (
SELECT
	(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions) AS total_customer,
	COUNT(DISTINCT customer_id) AS churned_after_trial
FROM CTE
WHERE rn = 2 
AND plan_name = 'churn'
	)
SELECT 
	total_customer,
	churned_after_trial,
	ROUND((churned_after_trial*100)/total_customer) AS percentage
FROM CTE2
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/5.PNG)

What is the number and percentage of customer plans after their initial free trial?

```sql
WITH CTE AS ( 
SELECT
	customer_id,
	plan_name,
	ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS rn
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
),
CTE2 AS (
SELECT
	plan_name,
	(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions) AS total_customer,
	COUNT(customer_id) AS total_customer_after_trial
FROM CTE
WHERE rn = 2
GROUP BY plan_name
	)
SELECT
	total_customer_after_trial,
	ROUND((total_customer_after_trial*100.1)/total_customer) AS percentage
FROM CTE2
ORDER BY total_customer_after_trial
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/6.PNG)

What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
WITH CTE AS(
SELECT *,
	ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date DESC) as rn
FROM foodie_fi.subscriptions
WHERE start_date <= '2020-12-31'
),
CTE2 AS (
		SELECT 
		plan_name,
		(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions) AS total_customer,
		COUNT(customer_id) AS customer_count
		FROM CTE
	INNER JOIN foodie_fi.plans AS p ON CTE.plan_id = p.plan_id
	WHERE rn = 1
	GROUP BY plan_name 
)
SELECT 
	customer_count,
	ROUND((customer_count*100.1)/total_customer) AS percentage
FROM CTE2
ORDER BY customer_count
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/7.PNG)

How many customers have upgraded to an annual plan in 2020?

```sql
SELECT 
	COUNT(customer_id) AS annual_upgrade_count
FROM 
	foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
WHERE DATE_PART('year', start_date) = 2020
AND p.plan_name = 'pro annual' 
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/8.PNG)

How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

```sql
WITH trial AS (
SELECT 
	customer_id,
	start_date AS trial_start
FROM foodie_fi.subscriptions
WHERE plan_id = 0
),
annual AS (
SELECT 
	customer_id,
	start_date AS annual_start
FROM foodie_fi.subscriptions
WHERE plan_id = 3
)
SELECT 
	--t.customer_id,
	--trial_start,
	--annual_start,
	AVG(annual_start - trial_start) AS avg_days_from_trial_to_annual
FROM trial AS t
INNER JOIN annual AS a
on t.customer_id = a.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/9.PNG)

Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```sql
WITH start_date_trial AS (
SELECT 
	customer_id,
	start_date AS date_trial
FROM foodie_fi.subscriptions
WHERE plan_id = 0
),
annual_date AS (
SELECT 
	customer_id,
	start_date AS annual_date_plan
FROM foodie_fi.subscriptions 
WHERE plan_id = 3
),
day_diff_bucket AS (
SELECT 
	WIDTH_BUCKET(annual_date_plan - date_trial, 0, 300, 10) AS day_diff
FROM start_date_trial sd
JOIN annual_date ad
ON sd.customer_id = ad.customer_id
)
SELECT 
	CONCAT((day_diff -1) * 30, '-', (day_diff) * 30, ' days') AS time_breakdown,
	COUNT(*) AS customer_count
FROM
	day_diff_bucket
GROUP BY
	1,
	day_diff
ORDER BY 
	day_diff ASC
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/10.PNG)

How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
WITH pro_mon AS (
SELECT
	customer_id,
	start_date AS pro_month_start
FROM foodie_fi.subscriptions
WHERE plan_id = 2
),
basic_mon AS (
SELECT 
	customer_id,
	start_date AS basic_month_start
FROM foodie_fi.subscriptions
WHERE plan_id = 1
)
SELECT 
	COUNT(*) AS downgrade_account
FROM pro_mon AS P
INNER JOIN basic_mon AS b ON p.customer_id = b.customer_id
WHERE pro_month_start < basic_month_start
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week3/11.PNG)


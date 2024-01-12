Hi there, before we begin to complete this week 2 challange
It's always the best option for us to analyze our dataset first

```sql
SELECT *
FROM customer_orders AS co;
SELECT *
FROM runner_orders AS ro;
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/customer_orders.PNG)
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/runner_orders.PNG)

- See? There's a lot of things that can cause trouble in the process
- Let's clean the customer_orders table first
- clean null string and empty into 0 value in exclusions
- clean null string, empty string and also [NULL] into 0 value in extras
- wrap it inside a view

```sql
CREATE VIEW pizza_runner.customer_orders_cleaned AS
SELECT
	order_id,
	customer_id,
	pizza_id,
	CASE
		WHEN exclusions = 'null' THEN '0'
		WHEN exclusions IS NULL THEN '0'
		WHEN exclusions = '' THEN '0'
		ELSE exclusions
	END AS exclusions,
	CASE
		WHEN extras IS NULL THEN '0'
		WHEN extras = 'null' THEN '0'
		WHEN extras = '' THEN '0'
		ELSE extras
	END AS extras,
	order_time
FROM
	pizza_runner.customer_orders AS co
```
Looks better now

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/customer_orders_cleaned.PNG)


let us clean the runner_orders table
- transform null pickup time and change format of pickup time column to datetime
- clean up distance column, removing KM, clean null and change type into float
- clean up duration column, removing minutes,mins, null
- clean up cancellation column, replace null values to 'no cancellation'
- create a view for the runners_orders cleaned table

```sql
CREATE VIEW pizza_runner.runner_orders_cleaned AS
WITH cleaned_runner_order as(
SELECT
	order_id,
	runner_id,
	CASE
		WHEN pickup_time = 'null' THEN NULL
		ELSE pickup_time
	END AS pickup_time,
	CASE
		WHEN distance LIKE 'null' THEN '0'
		WHEN distance ILIKE '%km' THEN trim('km' FROM distance)
		ELSE distance
	END,
	CASE
		WHEN duration = 'null' THEN '0'
		WHEN duration LIKE '%mins' THEN trim('mins' FROM duration)
		WHEN duration LIKE '%minutes' THEN trim('minutes' FROM duration)
		WHEN duration LIKE '%minute' THEN trim('minute' FROM duration)
		ELSE duration
	END,
	CASE
		WHEN cancellation = '' THEN 'No Cancellation'
		WHEN cancellation IS NULL THEN 'No Cancellation'
		WHEN cancellation = 'null' THEN 'No Cancellation'
		ELSE cancellation
	END
FROM
	pizza_runner.runner_orders AS ro
)

CREATE VIEW pizza_runner.cleaned_data_type AS (
SELECT
        order_id,
        runner_id,
        pickup_time::timestamp,
        distance::float,
        duration::int,
        cancellation
FROM runner_pizza.runner_orders_cleaned
)

SELECT *
FROM runner_pizza.cleaned_data_type
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/runner_order_cleaned.PNG)

Now that we're good and settled!! let's beat the challange!!

A. Pizza metrics

How many pizzas were ordered?

```sql
SELECT 
	COUNT(order_id) AS Total_Ordered
FROM 
	pizza_runner.customer_orders_cleaned
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a1.PNG)

How many unique customer orders were made?

```sql
SELECT 
	COUNT(DISTINCT order_id) AS Total_Customer
FROM
	pizza_runner.customer_orders_cleaned
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a2.PNG)

How many successful orders were delivered by each runner?

```sql
SELECT runner_id, count(order_id) AS Total_Succesful_Order
FROM pizza_runner.runner_orders_cleaned
WHERE cancellation = 'No Cancellation'
GROUP BY runner_id
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a3.PNG)

How many of each type of pizza was delivered?

```sql
SELECT co.pizza_id, COUNT(DISTINCT ro.order_id) as Total_Delivered
FROM pizza_runner.customer_orders_cleaned AS co
JOIN pizza_runner.runner_orders_cleaned AS ro
ON co.order_id = ro.order_id
WHERE ro.cancellation = 'No Cancellation'
GROUP BY co.pizza_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a4.PNG)

How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT DISTINCT customer_id, COUNT(pizza_id) AS Total_Ordered,
	CASE
		WHEN pizza_id = 1 THEN 'Meat Lovers'
		ELSE 'Vegetarian'
		END AS pizza_kind
FROM pizza_runner.customer_orders_cleaned
GROUP BY customer_id, pizza_id
ORDER BY customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a5.PNG)

What was the maximum number of pizzas delivered in a single order?

```sql
WITH total_orders AS(
		SELECT order_time, COUNT(*) AS Maximum_Orders
		FROM pizza_runner.customer_orders_cleaned
		GROUP BY order_time
		HAVING COUNT(*) > 1
		ORDER BY Maximum_Orders
		)

SELECT MAX(Maximum_Orders) AS Max_Ordered
FROM total_orders
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a6.PNG)

For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 	co.customer_id,
		SUM(CASE WHEN exclsuions = '0' THEN 1
		ELSE 0
		END) AS no_change,
		SUM(CASE WHEN exclsuions != '0' THEN 1
		ELSE 0
		END) AS change_atleast1
FROM pizza_runner.customer_orders_cleaned co
JOIN pizza_runner.runner_orders_cleaned ro
ON co.order_id = ro.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY co.customer_id
ORDER BY co.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a7.PNG)

How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT co.customer_id, 
		SUM(CASE WHEN co.exclsuions != '0' AND co.extras != '0'
		THEN 1
		ELSE 0
		END) AS have_both
FROM pizza_runner.customer_orders_cleaned co
JOIN pizza_runner.runner_orders_cleaned ro
ON co.order_id = ro.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY co.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a8.PNG)


What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT EXTRACT(HOUR FROM co.order_time) AS HOUR, COUNT(*) AS volume
FROM pizza_runner.customer_orders_cleaned AS co
JOIN pizza_runner.runner_orders_cleaned  AS ro
ON co.order_id = ro.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY HOUR
ORDER BY HOUR
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a9.PNG)

What was the volume of orders for each day of the week?

```sql
SELECT to_char(co.order_time, 'Day') AS Weekday, COUNT(*) as Total
FROM pizza_runner.customer_orders_cleaned co
JOIN pizza_runner.runner_orders_cleaned ro
ON co.order_id = ro.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY Weekday
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/a10.PNG)



B. Runner and Customer Experience

How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT
	DATEPART(WEEK, registration_date) AS week_number,
	COUNT(*) AS total_regist
FROM dbo.runners
GROUP BY 
	DATEPART(WEEK, registration_date)
ORDER by week_number
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b1.PNG)

What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT	runner_id, 
		avg(duration) as pickup_avg
FROM dbo.runner_orders
WHERE duration IS NOT NULL
GROUP BY runner_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b2.PNG)

What was the average distance travelled for each customer?

```sql
SELECT	co.customer_id, 
		avg(ro.distance) AS [avg distance]
FROM customer_orders co
JOIN runner_orders ro
ON co.order_id = ro.order_id
WHERE ro.distance IS NOT NULL
GROUP BY customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b4.PNG)

What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT 
	MAX(duration) AS longest,
	MIN(duration) AS shortest,
	MAX(duration) - MIN(duration) AS duration_difference
FROM dbo.runner_orders
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b5.PNG)

What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT 
	runner_id, 
	AVG(distance) AS avgdistance, 
	AVG(duration) AS avgduration
FROM runner_orders
WHERE duration is not null
GROUP BY runner_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b6.PNG)

What is the successful delivery percentage for each runner?
```sql
SELECT 
		runner_id,
		COUNT(CASE WHEN cancellation = 'No Cancellation' THEN 1 END) AS Successful_deliveries,
		COUNT(CASE WHEN cancellation IS NULL THEN 1 END) AS Unsuccesful_deliveries,
		COUNT(*) AS Total,
		(COUNT(CASE WHEN cancellation = 'No Cancellation' THEN 1 END) * 100/ COUNT(*)) AS Percentage_total
FROM dbo.runner_orders
GROUP BY runner_id
```
![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week2/b7.PNG)

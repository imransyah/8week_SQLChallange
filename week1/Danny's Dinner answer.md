1. **What is the total amount each customer spent at the restaurant?**

```sql
SELECT
      sal.customer_id,
      sum(men.price) AS spending
FROM
      sales sal
JOIN
      menu men

ON    sal.product_id = men.product_id

GROUP BY
      sal.customer_id 
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/1.png)

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
      customer_id,
      count(distinct order_date) AS [Total Kunjungan]
FROM
      sales
GROUP BY
      customer_id
```

--hanya munculin 1 data per kolom nya (jadi klo sehari ada 2 orderdate yg ditampilkan hanya 1 karena pakai 'distinct')

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/2.png)


**3. What was the first item from the menu purchased by each customer?**

(SUB QUERY)

```sql
SELECT
      sal.customer_id,
      men.product_name AS pembelianpertama
FROM
      sales sal
JOIN
      menu men
ON
      sal.product_id = men.product_id
WHERE
      sal.order_date = (select min(order_date) from sales)

GROUP BY
      sal.customer_id, men.product_name
ORDER BY
      sal.customer\_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/3a.png)

(CTE)

```sql
WITH
      table_filter AS (SELECT sal.customer_id,
      men.product_name,
      sal.order_date, row_number() OVER (PARTITION BY customer_id ORDER BY order_date asc) AS pembelianpertama
FROM
      sales sal
JOIN
      menu men ON sal.product_id = men.product_id
GROUP BY
      sal.customer_id, men.product_name, sal.order_date)
SELECT * FROM
      table_filter
WHERE
      pembelianpertama = 1
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/3b.png)

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

#1 kalau mau lihat total sales semua produk tiap customer

```sql
SELECT
      customer_id,
      purchase_count,
      product_name
FROM
(SELECT
      customer_id,
      product_name,
      COUNT(*) AS purchase_count,
DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*)DESC)AS rnk
FROM
      sales
JOIN
      menu
ON
      sales.product_id = menu.product_id
GROUP BY
      customer_id, product_name) ranked
WHERE
      rnk = 1
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/4a.png)

#2 kalau mau cari produk dengan sales terbanyak, dan brp sales produk tsb dari tiap customer

```sql
SELECT
    sal.customer_id,
    men.product_name,
    count(sal.product_id) AS purchase_count
FROM
    sales sal
JOIN
    menu men
ON
    sal.product_id = men.product_id
WHERE
    sal.product_id = (
        SELECT
              MAX (product_id)
        FROM
              sales)
GROUP BY
    sal.customer_id,
    men.product_name

ORDER BY
    sal.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/4b.png)

**5. Which item was the most popular for each customer?**

```SQL
SELECT
    sal.customer_id,
    men.product_name
FROM
    sales sal
JOIN
    menu men
ON
    sal.product_id = men.product_id
WHERE
    sal.product_id = (SELECT MAX (product_id)
                      from sales)
GROUP BY
    sal.customer_id,
    men.product_name
ORDER BY
    sal.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/5.png)

**6. Which item was purchased first by the customer after they became a member?**

```sql
SELECT
    mem.customer_id,
    mem.join_date,
    men.product_id,
    men.product_name AS [pembelian pertama]
FROM
    members mem
JOIN
    sales sal
ON
    mem.customer_id = sal.customer_id
JOIN
    menu men
ON
    sal.product_id = men.product_id
WHERE
    sal.order_date = (SELECT MIN(join_date)
                      FROM members)
GROUP BY
    mem.customer_id,
    mem.join_date,
    men.product_id,
    men.product_name
ORDER BY
    mem.customer\_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/6a.png)

_**(ini untuk nampilin semua pembelian setelah jadi member)**_

```sql
SELECT
    mem.customer_id,
    mem.join_date,
    men.product_id AS [pembelian setelah jadi member],
    men.product_name AS [nama menu]
FROM
    members mem
JOIN
    sales sal
ON
    mem.customer_id = sal.customer_id
JOIN
    menu men
ON
    sal.product_id = men.product_id
WHERE
    sal.order_date >= mem.join_date
GROUP BY
    mem.customer_id,
    mem.join_date,
    men.product_id,
    men.product_name
ORDER BY
    mem.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/6b.png)

**7. Which item was purchased just before the customer became a member?**

```sql
SELECT
    mem.customer_id,
    sal.order_date,
    men.product_id as [pembelian sebelum jadi member],
    men.product_name as [nama menu]
FROM
    members mem
LEFT OUTER JOIN sales sal
ON
    mem.customer_id = sal.customer_id
JOIN
    menu men
ON
    sal.product_id = men.product_id
WHERE
    sal.order_date < mem.join_date
GROUP BY
    mem.customer_id,
    sal.order_date,
    men.product_id,
    men.product_name
ORDER BY
    mem.customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/7.png)

**8. What is the total items and amount spent for each member before they became a member?**

```sql
WITH
    table_filter as (
                      SELECT
                            mem.customer_id,
                            sal.order_date,
                            SUM
                              (men.price) AS harga
                      FROM
                            members mem
                      LEFT OUTER JOIN
                            sales sal
                      ON
                            mem.customer_id = sal.customer_id
                      JOIN
                            menu men
                      ON
                            sal.product_id = men.product_id
                      WHERE
                            sal.order_date < mem.join_date
                      GROUP BY
                            mem.customer_id,
                            sal.order_date,
                            men.price
                  )

SELECT
    customer_id,
    SUM(harga) AS [Total Spending]
FROM
    table_filter
GROUP BY
    customer_id
ORDER BY
    customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/8.png)

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
WITH 
    table_filter AS (
                      select 
                            sal.customer_id, 
                            men.product_name, 
                            men.price,
                            COUNT(sal.product_id) AS penjualan,
                              CASE WHEN 
                                      men.product_name = 'sushi' THEN (men.price * 20 * COUNT (sal.product_id))
                                      ELSE (men.price * 10 * COUNT (sal.product_id))
                                      END AS points
                            FROM 
                                sales sal
                            JOIN
                                menu men
                            ON 
                                sal.product_id = men.product_id
                            GROUP BY
                                sal.customer_id, 
                                men.price, 
                                men.product_name
                                                    )
SELECT 
      customer_id,
      sum(points) AS [Points Achieved]

FROM
      table_filter
GROUP BY 
      customer\_id
ORDER BY 
      customer\_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/9.png)

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi, how many points do customer A and B have at the end of January?**

```sql
WITH
    table_filter AS (
                      SELECT
                            mem.customer_id,
                            mem.join_date,
                            men.price,
                            COUNT(sal.product_id) AS penjualan,
                            sal.order_date,
                                CASE WHEN
                                    (sal.order_date >= mem.join_date)
                                THEN
                                    (men.price * 20 * COUNT(sal.product_id))
                                ELSE 0
                                END AS points
                            FROM
                                sales sal
                            JOIN
                                menu men
                            ON
                                sal.product_id = men.product_id
                            JOIN
                                members mem
                            ON
                                mem.customer_id = sal.customer_id
                            WHERE
                                sal.order_date >= mem.join_date
                            GROUP BY
                                mem.customer_id,
                                mem.join_date,
                                men.product_id,
                                men.price,
                                sal.order_date )
SELECT
      customer_id,
      SUM(points) AS [Points Achieved]
FROM
      table_filter
WHERE
      order_date < '2021-02-01'
GROUP BY
      customer_id
ORDER BY
      customer_id
```

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/10.png)

1. **What is the total amount each customer spent at the restaurant?**

select sal.customer\_id,sum(men.price)as spending

from sales sal

join menu men

on sal.product\_id = men.product\_id

groupby sal.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/1.png)

**2. How many days has each customer visited the restaurant?**

select customer\_id,count(distinct order\_date)as [Total Kunjungan]

--hanya munculin 1 data per kolom nya (jadi klo sehari ada 2 orderdate yg ditampilkan hanya 1 karena pakai 'distinct')

from sales

groupby customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/2.png)

**3. What was the first item from the menu purchased by each customer?**

(SUB QUERY)

select sal.customer\_id, men.product\_name as pembelianpertama

from sales sal

join menu men

on sal.product\_id = men.product\_id

where sal.order\_date =(selectmin(order\_date)

from sales)

groupby sal.customer\_id, men.product\_name

orderby sal.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/3a.png)

(CTE)

WITH table\_filter as

(select sal.customer\_id, men.product\_name, sal.order\_date,

row\_number()over (partitionby customer\_id orderby order\_date asc)as pembelianpertama

from sales sal

join menu men

on sal.product\_id = men.product\_id

groupby sal.customer\_id, men.product\_name, sal.order\_date)

SELECT\*

from table\_filter

where pembelianpertama = 1

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/3b.png)

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

#1 kalau mau lihat total sales semua produk tiap customer

SELECT customer\_id, purchase\_count, product\_name

FROM (

SELECT customer\_id, product\_name,COUNT(\*)AS purchase\_count,

DENSE\_RANK()OVER (PARTITIONBY customer\_id ORDERBYCOUNT(\*)DESC)AS rnk

FROM sales

join menu

on sales.product\_id = menu.product\_id

GROUPBY customer\_id, product\_name

) ranked

WHERE rnk = 1

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/4a.png)

#2 kalau mau cari produk dengan sales terbanyak, dan brp sales produk tsb dari tiap customer

select sal.customer\_id, men.product\_name,count(sal.product\_id)as purchase\_count

from sales sal

join menu men

on sal.product\_id = men.product\_id

where sal.product\_id =(

selectmax(product\_id)from sales)

groupby sal.customer\_id, men.product\_name

orderby sal.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/4b.png)

**5. Which item was the most popular for each customer?**

select sal.customer\_id, men.product\_name

from sales sal

join menu men

on sal.product\_id = men.product\_id

where sal.product\_id =(

selectmax(product\_id)from sales)

groupby sal.customer\_id, men.product\_name

orderby sal.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/5.png)

**6. Which item was purchased first by the customer after they became a member?**

select mem.customer\_id, mem.join\_date, men.product\_id, men.product\_name as [pembelian pertama]

from members mem

join sales sal

on mem.customer\_id = sal.customer\_id

join menu men

on sal.product\_id = men.product\_id

where sal.order\_date =(

selectmin(join\_date)

from members)

groupby mem.customer\_id, mem.join\_date, men.product\_id, men.product\_name

orderby mem.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/6a.png)

_**(ini untuk nampilin semua pembelian setelah jadi member)**_

select mem.customer\_id, mem.join\_date, men.product\_id as [pembelian setelah jadi member], men.product\_name as [nama menu]

from members mem

join sales sal

on mem.customer\_id = sal.customer\_id

join menu men

on sal.product\_id = men.product\_id

where sal.order\_date \>= mem.join\_date

groupby mem.customer\_id, mem.join\_date, men.product\_id, men.product\_name

orderby mem.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/6b.png)

**7. Which item was purchased just before the customer became a member?**

select mem.customer\_id, sal.order\_date, men.product\_id as [pembelian sebelum jadi member], men.product\_name as [nama menu]

from members mem

leftouterjoin sales sal

on mem.customer\_id = sal.customer\_id

join menu men

on sal.product\_id = men.product\_id

where sal.order\_date \< mem.join\_date

groupby mem.customer\_id, sal.order\_date, men.product\_id, men.product\_name

orderby mem.customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/7.png)

**8. What is the total items and amount spent for each member before they became a member?**

with table\_filter as (

select mem.customer\_id, sal.order\_date,sum(men.price)as harga

from members mem

leftouterjoin sales sal

on mem.customer\_id = sal.customer\_id

join menu men

on sal.product\_id = men.product\_id

where sal.order\_date \< mem.join\_date

groupby mem.customer\_id, sal.order\_date, men.price

)

select customer\_id,sum(harga)as [Total Spending]

from table\_filter

groupby customer\_id

orderby customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/8.png)

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

with table\_filter as (

select sal.customer\_id, men.product\_name, men.price,count(sal.product\_id)as penjualan,

case

when men.product\_name ='sushi'then (men.price \* 20 \*count(sal.product\_id))

Else (men.price \* 10 \*count(sal.product\_id))

ENDAS points

from sales sal

join menu men

on sal.product\_id = men.product\_id

groupby sal.customer\_id, men.price, men.product\_name

)

select customer\_id,sum(points)as [Points Achieved]

from table\_filter

groupby customer\_id

orderby customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/9.png)

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi, how many points do customer A and B have at the end of January?**

with table\_filter as (

select mem.customer\_id, mem.join\_date, men.price,count(sal.product\_id)as penjualan, sal.order\_date,

case

when (sal.order\_date \>= mem.join\_date)then (men.price \* 20 \*count(sal.product\_id))

Else 0 ENDAS points

from sales sal

join menu men

on sal.product\_id = men.product\_id

join members mem

on mem.customer\_id = sal.customer\_id

where sal.order\_date \>= mem.join\_date

groupby mem.customer\_id, mem.join\_date, men.product\_id, men.price, sal.order\_date

)

select customer\_id,sum(points)as [Points Achieved]

from table\_filter

where order\_date \<'2021-02-01'

groupby customer\_id

orderby customer\_id

![1](https://github.com/imransyah/8week_SQLChallange/blob/main/week1/10.png)
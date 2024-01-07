1.	What is the total amount each customer spent at the restaurant?


select sal.customer_id, sum(men.price) as spending
from sales sal
join menu men
	on sal.product_id = men.product_id
group by sal.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\1.png)


2. How many days has each customer visited the restaurant?


select customer_id, count(distinct order_date) as [Total Kunjungan] 
--hanya munculin 1 data per kolom nya (jadi klo sehari ada 2 orderdate yg ditampilkan hanya 1 karena pakai 'distinct')
from sales
group by customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\2.png)


3. What was the first item from the menu purchased by each customer?


(SUB QUERY)
select sal.customer_id, men.product_name as pembelianpertama
from sales sal
join menu men 
	on sal.product_id = men.product_id
where sal.order_date = (select min(order_date) 
from sales) 
group by sal.customer_id, men.product_name
order by sal.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\3a.png)


(CTE)
WITH table_filter as 
(select sal.customer_id, men.product_name, sal.order_date,
	row_number () over (partition by customer_id order by order_date asc) as pembelianpertama
from sales sal
join menu men 
	on sal.product_id = men.product_id
group by sal.customer_id, men.product_name, sal.order_date)
SELECT * 
from table_filter
where pembelianpertama = 1

![1](C:\Users\imran\Pictures\Answer1 pic\3b.png)


4. What is the most purchased item on the menu and how many times was it purchased by all customers?


#1 kalau mau lihat total sales semua produk tiap customer
SELECT customer_id, purchase_count, product_name
FROM (
    SELECT customer_id,  product_name, COUNT(*) AS purchase_count, 
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS rnk
    FROM sales
	join menu 
		on sales.product_id = menu.product_id
    GROUP BY customer_id, product_name
	) ranked
WHERE rnk = 1

![1](C:\Users\imran\Pictures\Answer1 pic\4a.png)
 

#2 kalau mau cari produk dengan sales terbanyak, dan brp sales produk tsb dari tiap customer


select sal.customer_id, men.product_name, count(sal.product_id) as purchase_count
from sales sal
join menu men
	on sal.product_id = men.product_id
where sal.product_id = (
	select max(product_id) from sales)
group by sal.customer_id, men.product_name
order by sal.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\4b.png)

5. Which item was the most popular for each customer?


select sal.customer_id, men.product_name
from sales sal
join menu men
	on sal.product_id = men.product_id
where sal.product_id = (
	select max(product_id) from sales)
group by sal.customer_id, men.product_name
order by sal.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\5.png)

6. Which item was purchased first by the customer after they became a member?


select mem.customer_id, mem.join_date, men.product_id, men.product_name as [pembelian pertama]
from members mem
join sales sal
	on mem.customer_id = sal.customer_id
join menu men
	on sal.product_id = men.product_id
where sal.order_date = ( 
	select min(join_date) 
	from members)
group by mem.customer_id, mem.join_date, men.product_id, men.product_name
order by mem.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\6a.png)

(ini untuk nampilin semua pembelian setelah jadi member)


select mem.customer_id, mem.join_date, men.product_id as [pembelian setelah jadi member], men.product_name as [nama menu]
from members mem
join sales sal
	on mem.customer_id = sal.customer_id
join menu men
	on sal.product_id = men.product_id 
where sal.order_date > = mem.join_date
group by mem.customer_id, mem.join_date, men.product_id, men.product_name
order by mem.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\6b.png)

7. Which item was purchased just before the customer became a member?


select mem.customer_id, sal.order_date, men.product_id as [pembelian sebelum jadi member], men product_name as [nama menu]
from members mem
left outer join sales sal
	on mem.customer_id = sal.customer_id
join menu men
	on sal.product_id = men.product_id 
where sal.order_date < mem.join_date
group by mem.customer_id, sal.order_date, men.product_id, men.product_name
order by mem.customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\7.png) 

8. What is the total items and amount spent for each member before they became a member?


with table_filter as (
select mem.customer_id, sal.order_date, sum(men.price) as harga
from members mem
left outer join sales sal
	on mem.customer_id = sal.customer_id
join menu men
	on sal.product_id = men.product_id 
where sal.order_date < mem.join_date
group by mem.customer_id, sal.order_date, men.price 
)
select customer_id, sum(harga) as [Total Spending]
from table_filter
group by customer_id
order by customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\8.png) 

9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


with table_filter as (
select sal.customer_id, men.product_name, men.price, count(sal.product_id) as penjualan,
case 
	when men.product_name = 'sushi' then (men.price * 20 * count(sal.product_id))
	Else (men.price * 10 * count(sal.product_id))
END AS points
from sales sal
join menu men 
	on sal.product_id = men.product_id
group by sal.customer_id, men.price, men.product_name
)
select customer_id, sum(points) as [Points Achieved] 
from table_filter
group by customer_id
order by customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\9.png) 

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi, how many points do customer A and B have at the end of January?


with table_filter as (
select mem.customer_id, mem.join_date, men.price, count(sal.product_id) as penjualan, sal.order_date,
case 
	when (sal.order_date > = mem.join_date) then (men.price * 20 * count(sal.product_id))
	Else 0 END AS points
from sales sal
join menu men 
	on sal.product_id = men.product_id
join members mem
	on mem.customer_id = sal.customer_id
where sal.order_date > = mem.join_date
group by mem.customer_id, mem.join_date, men.product_id, men.price, sal.order_date
)
select customer_id, sum(points) as [Points Achieved]
from table_filter
where order_date < '2021-02-01'
group by customer_id
order by customer_id

![1](C:\Users\imran\Pictures\Answer1 pic\10.png)
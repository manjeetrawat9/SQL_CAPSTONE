# 1st What is the total amount each customer spent at the restaurant?
```sql
select m.customer_id,m.Cust_name,sum(o.amount) as Tot_spent
FROM members as m join orders as o 
on m.customer_id=o.customer_id
group by 1
```

# 2nd How many days has each customer visited the restaurant?
```sql
select m.customer_id,m.cust_name,count(distinct o.date) as visit_count
from members as m join orders as o 
on m.customer_id=o.customer_id
group by 1
```

# 3rd What was the first item from the menu purchased by each customer?
```sql
select d.customer_id,d.order_id,d.food_name
from 
(select c.customer_id,c.order_id,c.food_name, dense_rank() 
over(partition by c.customer_id order by c.date)as rank_
from 
(select b.customer_id,b.restaurant_id,b.order_id,
b.date,b.food_id,f.food_name
from 
(select a.customer_id,a.restaurant_id,
a.order_id,a.date,od.food_id
from 
(select m.customer_id,o.restaurant_id,o.order_id,o.date
from members as m join orders as o 
on  m.customer_id=o.customer_id) as a join orders_details as od
on a.order_id=od.order_id) as b join food as f 
on f.food_id=b.food_id) as c)as d
where rank_=1
```

# 4th What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
select f.food_name,count(od.food_id)as most_purchased 
 from orders_details as od join food as f 
 on f.food_id=od.food_id
 group by 1 order by 2 desc limit 1
 ```
 
 # 5th Which item was the most popular one for each customer?
 
 ```sql
 with  cte as 
 (select b.customer_id,b.food_id,count(b.food_id)as purchased_count
from 
 (select a.customer_id,a.order_id,od.food_id
 from 
(select m.customer_id,o.order_id
from orders as o join members as m 
on m.customer_id=o.customer_id)as a join orders_details as od
on a.order_id=od.order_id) as b
 group by 1,2) 
 
 select d.customer_id,d.food_name,d.purchased_count
 from
 (select cte.customer_id,f.food_name,cte.purchased_count,dense_rank() 
 over(partition by cte.customer_id order by purchased_count desc) as rank_
from cte  join food as f 
on cte.food_id=f.food_id) as d 
where rank_=1
```

# 6th which item was purchased first by the customer after they became a member?
```sql
with a as
(select o.order_id,m.customer_id,m.cust_name,o.date,m.join_date
from orders as o join members as m 
on o.customer_id=m.customer_id
where o.date>m.join_date)

select d.customer_id,d.food_id,f.food_name
from 
(select a.customer_id,od.food_id,a.date,a.order_id,dense_rank() 
over(partition by a.customer_id order by date) as rank_ 
from a join orders_details as od 
on od.order_id=a.order_id) as d join food as f 
on f.food_id=d.food_id
where rank_=1
order by 1
```

#7th If each customers’ Rs 1 spent equates to 10 points and Kadai Paneer has a 2x points multiplier — how many points would each customer have?
```sql
with b as 
(with a as
(with cte as
(select o.order_id,o.customer_id,o.amount,od.food_id,o.restaurant_id
from orders as o join orders_details as od
on o.order_id=od.order_id)
select cte.*,f.food_name
from cte join food as f 
on cte.food_id=f.food_id)
select *, (case when a.food_name="Kadai paneer" 
then a.amount*2 else a.amount*1 end)as points
from a)
select customer_id,sum(points) as total_points from b
group by 1
order by 1 asc
```
# 8th find customer who have never order
```sql
### 1st methord

select customer_id,cust_name from members
where customer_id not in (select customer_id from orders)

### 2nd Methord

select m.customer_id,m.cust_name
from members as m left join orders as o
on m.customer_id=o.customer_id
where order_id is null
```














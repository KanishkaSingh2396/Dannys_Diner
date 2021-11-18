# Dannys_Diner

```sql


--All Tables
select* from dannys_diner.members;
select* from dannys_diner.menu;
select* from dannys_diner.sales;

 --1. What is the total amount each customer spent at the restaurant?
select 
  sales.customer_id,
  sum(menu.price)
from dannys_diner.sales 
  join dannys_diner.menu 
    on sales.product_id= menu.product_id
group by customer_id
order by customer_id;

--2. How many days has each customer visited the restaurant?
select
customer_id,
count(distinct order_date)
from dannys_diner.sales
group by customer_id;




--3. What was the first item from the menu purchased by each customer?

with prod_sales as 
  (
  select M.product_name, S.customer_id, S.order_date
  from dannys_diner.sales as S
    join dannys_diner.menu as M
      on M.product_id = S.product_id
  )
select product_name, customer_id 
from 
  (
  select product_name, customer_id, order_date, rank() over(partition by customer_id order by order_date) as ranking
  from prod_sales
  ) as sub_table
where ranking = 1;
  --------------------------------------

WITH ordered_sales AS (
  SELECT
    sales.customer_id,
    RANK() OVER (
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date
    ) AS order_rank,
    menu.product_name
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    on sales.product_id = menu.product_id
)

SELECT DISTINCT
  customer_id,
  product_name
FROM ordered_sales
-- what about this?
WHERE order_rank = 1;

----------------------------------------------

--4. What is the most purchased item on the menu and how many times was it purchased by all customers?

with purchase_count as (
  select M.product_name, S.order_date
  from dannys_diner.menu as M
  join dannys_diner.sales as S
    on S.product_id= M.product_id)

select product_name, Count(product_name) as prod_count
from purchase_count
group by product_name
order by Count(product_name) desc;

--5. Which item was the most popular for each customer?
select
 S.customer_id, M.product_name, count(M.product_name)
from dannys_diner.menu as M
  join dannys_diner.sales as S
    on S.product_id= M.product_id
group by M.product_name, S.customer_id;



--6. Which item was purchased first by the customer after they became a member?
with joined_table as 
(
  select sales.customer_id, menu.product_name, members.join_date, sales.order_date
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
)
select product_name, join_date, customer_id, order_date
from
  (select rank() over(partition by product_name order by order_date) as "ranking",
  product_name, join_date, customer_id, order_date
  from joined_table
  where order_date >=join_date) as ranking_table
where ranking = 1;

--7. Which item was purchased just before the customer became a member?
with joined_table as 
(
  select sales.customer_id, menu.product_name, members.join_date, sales.order_date
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
)
select product_name, join_date, customer_id, order_date, ranking
from
  (select rank() over(partition by product_name order by order_date) as "ranking",
  product_name, join_date, customer_id, order_date
  from joined_table
  where order_date <=join_date) as ranking_table
where ranking>1;



--8. What is the total items and amount spent for each member before they became a member?
with joined_tb_modify as 
  (SELECT 
    sales.customer_id, Menu.product_name, Menu.price, sales.order_date, members.join_date
  FROM dannys_diner.sales 
   JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
   JOIN dannys_diner.members 
    ON sales.customer_id = members.customer_id)
select 
  customer_id,
  sum(price) as total_spend,
  count(product_name) as total_items
from joined_tb_modify
where order_date < join_date
GROUP BY customer_id;

--9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
with joined_points as (
  SELECT 
    sales.customer_id, Menu.product_name, Menu.price
  FROM dannys_diner.sales 
   JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
)
select
  customer_id,
  sum(case
         when product_name = 'sushi' then 20 * price
         else 10 * price 
        end) as total_points
from joined_points
group by customer_id
order by total_points desc;

--10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

with joined_table as (
  SELECT 
    sales.customer_id, 
    Menu.product_name, 
    Menu.price, 
    sales.order_date, 
    members.join_date, 
    join_date +6 as one_week
  FROM dannys_diner.sales 
   JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
   JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
)
select 
  sum(case
    when order_date >= join_date and order_date <= one_week then price* 20
    when  product_name = 'sushi' then price*20
    else price* 10
  end) as total_counts,
  customer_id
from joined_table
where order_date <= '2021-01-31'
group by customer_id;
----------------------------------------
with my_table as (
select ME.customer_id as ci , M.product_id as pi, * from dannys_diner.menu as M 
              LEFT JOIN dannys_diner.sales as S 
              ON M.product_id = S.product_id 
              INNER JOIN dannys_diner.members as ME
              ON S.customer_id = ME.customer_id
              ),

my_table2 as (
select ci, price , product_name , pi , order_date, join_date, 
join_date + INTERVAL '6 day'  as one_week_addition
              from my_table
              ),

my_table3 as (
select ci, price, order_date, join_date, one_week_addition ,
case when order_date >= join_date AND order_date <= one_week_addition then price * 20
when product_name = 'sushi' then price*20
else price * 10
end as pts
from my_table2
where order_date < '2021-01-31'
          )
select ci, sum(pts) as pts from my_table3
        group by ci
        
```

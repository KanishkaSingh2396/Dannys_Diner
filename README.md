# Dannys_Diner

![image](https://user-images.githubusercontent.com/89623051/143564498-b99619f0-2e9d-4518-a3f0-e60c7cc2eed7.png)

### Introduction

This tutorial contains exactly the same content as the 8 Week SQL Challenge Danny’s Diner case study - however everything has been optimised for our SQLPad Docker instance and there are also solutions included below…but there is a twist!

### Context

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

### Example Datasets

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

sales
menu
members
You can inspect the entity relationship diagram and example data below.

### Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/89623051/143564804-13544558-aa3b-4a9e-89f0-8c47d8f039c6.png)

### TABLES

![image](https://user-images.githubusercontent.com/89623051/143564922-15ca27b7-55e0-40e2-a83f-76ef7f94e7eb.png)

![image](https://user-images.githubusercontent.com/89623051/143565031-48938f68-442c-489e-901e-51ef31d32a2b.png)

### Case Study Questions

Each of the following case study questions can be answered using a single SQL statement:

What is the total amount each customer spent at the restaurant?

How many days has each customer visited the restaurant?

What was the first item from the menu purchased by each customer?

What is the most purchased item on the menu and how many times was it purchased by all customers?

Which item was the most popular for each customer?

Which item was purchased first by the customer after they became a member?

Which item was purchased just before the customer became a member?

What is the total items and amount spent for each member before they became a member?

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### CASE STUDY SOLUTION

##### 1. What is the total amount each customer spent at the restaurant?

```sql
select 
  sales.customer_id,
  sum(menu.price)
from dannys_diner.sales 
  join dannys_diner.menu 
    on sales.product_id= menu.product_id
group by customer_id
order by customer_id;
```
![image](https://user-images.githubusercontent.com/89623051/143566320-ca50da0a-59d7-4b41-b436-90efb64c9f6d.png)

##### 2. How many days has each customer visited the restaurant?

```sql
select
customer_id,
count(distinct order_date)
from dannys_diner.sales
group by customer_id;
```
![image](https://user-images.githubusercontent.com/89623051/143566390-5b967dea-badc-4ded-aabe-f2e8f1cab8d7.png)

##### 3. What was the first item from the menu purchased by each customer?
```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143566475-ff6c887e-f306-4845-a1e8-6af977cd962b.png)

##### 2nd Approach

```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143566580-fbbaadf3-eb1b-4353-b056-aa903fd78448.png)

##### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
with purchase_count as (
  select M.product_name, S.order_date
  from dannys_diner.menu as M
  join dannys_diner.sales as S
    on S.product_id= M.product_id)

select product_name, Count(product_name) as prod_count
from purchase_count
group by product_name
order by Count(product_name) desc;
```
![image](https://user-images.githubusercontent.com/89623051/143566893-653d58b0-7680-4071-9161-45efd0039076.png)

##### 5. Which item was the most popular for each customer?
```sql
select
 S.customer_id, M.product_name, count(M.product_name)
from dannys_diner.menu as M
  join dannys_diner.sales as S
    on S.product_id= M.product_id
group by M.product_name, S.customer_id;
```
![image](https://user-images.githubusercontent.com/89623051/143566940-ac9f0979-7bd0-4af4-8748-b03211109520.png)


##### 6. Which item was purchased first by the customer after they became a member?
```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143567020-d054f6c0-79c5-48a7-b351-44bb346805b9.png)

##### 7. Which item was purchased just before the customer became a member?
```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143567093-9445205d-9c67-4d78-9be6-d4d9630b13ac.png)

##### 8. What is the total items and amount spent for each member before they became a member?
```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143567202-e982e333-0501-4761-99b5-f177a9d61315.png)

##### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143567266-fe98bc7a-1f0b-4631-8beb-be44dbb23a85.png)

##### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
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
```
![image](https://user-images.githubusercontent.com/89623051/143567396-0c4f436c-362f-4014-8bd4-7f22fbd9b452.png)

##### 2ND Approach

```sql
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
![image](https://user-images.githubusercontent.com/89623051/143567504-4e1c26d3-1bc0-455d-9973-dbcd419977eb.png)


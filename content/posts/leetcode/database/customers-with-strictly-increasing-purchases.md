---
title: '[leetcode][Database][Hard] 2474. Customers With Strictly Increasing Purchases'
date: '2022-12-18T21:30:30.719Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Orders`
```
+--------------+------+  
| Column Name  | Type |  
+--------------+------+  
| order_id     | int  |  
| customer_id  | int  |  
| order_date   | date |  
| price        | int  |  
+--------------+------+  

order_id is the primary key for this table.  
Each row contains the id of an order, the id of customer that ordered it, the date of the order, and its price.
```

Write an SQL query to report the IDs of the customers with the `total purchases` strictly increasing yearly.

*   The `total purchases` of a customer in one year is the sum of the prices of their orders in that year. If for some year the customer did not make any order, we consider the total purchases `0`.
*   The first year to consider for each customer is the year of their `first order`.
*   The last year to consider for each customer is the year of their `last order`.

Return the result table `in any order`.

SQL Schema
```sql
Create table If Not Exists Orders (order_id int, customer_id int, order_date date, price int)  
Truncate table Orders  
insert into Orders (order_id, customer_id, order_date, price) values ('1', '1', '2019-07-01', '1100')  
insert into Orders (order_id, customer_id, order_date, price) values ('2', '1', '2019-11-01', '1200')  
insert into Orders (order_id, customer_id, order_date, price) values ('3', '1', '2020-05-26', '3000')  
insert into Orders (order_id, customer_id, order_date, price) values ('4', '1', '2021-08-31', '3100')  
insert into Orders (order_id, customer_id, order_date, price) values ('5', '1', '2022-12-07', '4700')  
insert into Orders (order_id, customer_id, order_date, price) values ('6', '2', '2015-01-01', '700')  
insert into Orders (order_id, customer_id, order_date, price) values ('7', '2', '2017-11-07', '1000')  
insert into Orders (order_id, customer_id, order_date, price) values ('8', '3', '2017-01-01', '900')  
insert into Orders (order_id, customer_id, order_date, price) values ('9', '3', '2018-11-07', '900')
```
## Idea

The query result format is in the following example.
```
+-------------+  
| customer_id |  
+-------------+  
| 1           |  
+-------------+
```
Fulfill requirements :

I need to find the range between the `first order` year to `last order` year of each customer, and set price as `0` if the year have not order record(s) of customer.

Then, I can calculate strictly increasing purchases with each year of each customer via window function `lead(column, offset) over()`.

Marking the calculate result `1` if the total purchase price of next year larger than current year, otherwise, marking `0` . Name marking result as `mark_increase_purchases` .

Finally, to compare the counting result of records and total `mark_increase_purchases` of each customer, I can get the result for customers are strictly increasing purchases or not. Due to next order after last order of each customer is not exist, so the records counting need to minus `1` .

## Solution
```sql
with recursive  
cte_customer_order_year as (  
    select   
        customer_id,  
        year(min(order_date)) as first_order_year,   
        year(max(order_date)) as last_order_year  
    from Orders  
    group by customer_id  
),  
cte_customer_zero_order as (  
    select customer_id, first_order_year, last_order_year from cte_customer_order_year  
    union  
    select customer_id, first_order_year+1, last_order_year from cte_customer_zero_order where first_order_year < last_order_year  
),  
cte_orders as (  
    select distinct  
        customer_id, year(order_date) as order_year,  
        sum(price) over(partition by customer_id, year(order_date) order by year(order_date)) as price  
    from (  
        select customer_id, makedate(first_order_year, 1) as order_date, 0 as price from cte_customer_zero_order  
        union  
        select customer_id, order_date, price from Orders  
    ) union_orders  
      
)  
  
select  
    a.customer_id  
from (  
    select   
        customer_id, order_year,  
        if(  
            lead(price, 1) over(partition by customer_id order by order_year)-price > 0,  
            1,  
            0  
        ) as mark_increase_purchases  
    from cte_orders  
) a  
group by a.customer_id  
having sum(a.mark_increase_purchases) = (count(a.customer_id)-1)
```
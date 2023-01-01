---
title: '[leetcode][Database][Hard] 2362. Generate the Invoice'
date: '2022-12-18T21:10:22.259Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Products`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| product_id  | int  |  
| price       | int  |  
+-------------+------+  
product_id is the primary key for this table.  
Each row in this table shows the ID of a product and the price of one unit.
```

Table: `Purchases`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| invoice_id  | int  |  
| product_id  | int  |  
| quantity    | int  |  
+-------------+------+  
(invoice_id, product_id) is the primary key for this table.  
Each row in this table shows the quantity ordered from one product in an invoice.
```

Write an SQL query to show the details of the invoice with the highest price. If two or more invoices have the same price, return the details of the one with the smallest `invoice_id`.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Products (product_id int, price int)  
Create table If Not Exists Purchases (invoice_id int, product_id int, quantity int)  
Truncate table Products  
insert into Products (product_id, price) values ('1', '100')  
insert into Products (product_id, price) values ('2', '200')  
Truncate table Purchases  
insert into Purchases (invoice_id, product_id, quantity) values ('1', '1', '2')  
insert into Purchases (invoice_id, product_id, quantity) values ('3', '2', '1')  
insert into Purchases (invoice_id, product_id, quantity) values ('2', '2', '3')  
insert into Purchases (invoice_id, product_id, quantity) values ('2', '1', '4')  
insert into Purchases (invoice_id, product_id, quantity) values ('4', '1', '10')
```

## Idea

The query result format is shown in the following example.
```
+------------+----------+-------+  
| product_id | quantity | price |  
+------------+----------+-------+  
| 2          | 3        | 600   |  
| 1          | 4        | 400   |  
+------------+----------+-------+
```
Fulfill requirements :

I used the function `sum() over()` to calculate totally price of each invoice, and raking the above calaulate result by sorted with `invoide_id` ascending, and totally pricing descending, and name ranking result as `rn`.

Then, finding the record which `rn` equals to `1` for query result.

## Solution
```sql
with  
cte as (  
    select  
        invoice_id, product_id, quantity, price, s_price,  
        row_number() over(order by s_price desc, invoice_id) as rn  
    from (  
        select  
            a.invoice_id, a.product_id, a.quantity,  
            ifnull(a.quantity * b.price, 0) as price,  
            sum(ifnull(a.quantity * b.price, 0)) over(partition by a.invoice_id) as s_price  
        from Purchases a  
        left join Products b using(product_id)  
    ) detail  
)  
  
select  
    product_id, quantity, price  
from cte  
where invoice_id = (select invoice_id from cte where rn = 1)
```
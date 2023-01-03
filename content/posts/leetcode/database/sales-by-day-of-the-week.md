---
title: '[leetcode][Database][Hard] 1479. Sales by Day of the Week'
date: '2022-12-06T08:37:04.705Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Orders`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| order_id      | int     |  
| customer_id   | int     |  
| order_date    | date    |   
| item_id       | varchar |  
| quantity      | int     |  
+---------------+---------+  
(ordered_id, item_id) is the primary key for this table.  
This table contains information on the orders placed.  
order_date is the date item_id was ordered by the customer with id customer_id.
```

Table: `Items`
```
+---------------------+---------+  
| Column Name         | Type    |  
+---------------------+---------+  
| item_id             | varchar |  
| item_name           | varchar |  
| item_category       | varchar |  
+---------------------+---------+  
item_id is the primary key for this table.  
item_name is the name of the item.  
item_category is the category of the item.
```

You are the business owner and would like to obtain a sales report for category items and the day of the week.

Write an SQL query to report how many units in each category have been ordered on each `day of the week`.

Return the result table `ordered` by `category`.

SQL Schema
```sql
Create table If Not Exists Orders (order_id int, customer_id int, order_date date, item_id varchar(30), quantity int)  
Create table If Not Exists Items (item_id varchar(30), item_name varchar(30), item_category varchar(30))  
Truncate table Orders  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('1', '1', '2020-06-01', '1', '10')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('2', '1', '2020-06-08', '2', '10')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('3', '2', '2020-06-02', '1', '5')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('4', '3', '2020-06-03', '3', '5')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('5', '4', '2020-06-04', '4', '1')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('6', '4', '2020-06-05', '5', '5')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('7', '5', '2020-06-05', '1', '10')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('8', '5', '2020-06-14', '4', '5')  
insert into Orders (order_id, customer_id, order_date, item_id, quantity) values ('9', '5', '2020-06-21', '3', '5')  
Truncate table Items  
insert into Items (item_id, item_name, item_category) values ('1', 'LC Alg. Book', 'Book')  
insert into Items (item_id, item_name, item_category) values ('2', 'LC DB. Book', 'Book')  
insert into Items (item_id, item_name, item_category) values ('3', 'LC SmarthPhone', 'Phone')  
insert into Items (item_id, item_name, item_category) values ('4', 'LC Phone 2020', 'Phone')  
insert into Items (item_id, item_name, item_category) values ('5', 'LC SmartGlass', 'Glasses')  
insert into Items (item_id, item_name, item_category) values ('6', 'LC T-Shirt XL', 'T-shirt')
```

## 解題思考

*   樞紐表要求依據分類 `category`列出周一至周日的數量 `quantity`統計，因此可以使用 `left join` 逐一列出樞紐表周一至周日的欄位。
```
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+  
| Category   | Monday    | Tuesday   | Wednesday | Thursday  | Friday    | Saturday  | Sunday    |  
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+  
| Book       | 20        | 5         | 0         | 0         | 10        | 0         | 0         |  
| Glasses    | 0         | 0         | 0         | 0         | 5         | 0         | 0         |  
| Phone      | 0         | 0         | 5         | 1         | 0         | 0         | 10        |  
| T-Shirt    | 0         | 0         | 0         | 0         | 0         | 0         | 0         |  
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
```
*   利用 `dayOfWeek()` 將 `orders.order_date` 轉換成周一至周日 `day`，並透過 `with clause` 建立新表格 `orders_with_group`，擷取 `orders.order_id` 、`orders.quantity` `dayOfWeek()` 轉換後的 `day` 以及和 `items` 表格關聯後取得的 `items.item_category` 欄位
*   依據 `orders_with_group`加總每個 `day` 的 `quantity` ，並透過 `with clause` 建立新表 `sum_quantity_by_category` 。  
    最後，從 `items` 表格列出不重複的 `item_category` ，並利用 `left join` 分別關聯周一至周日 `day` 的 `quantity` 加總。

## 解決方案
```sql
with  
orders_with_group as (  
    select  
        a.order_id,   
        dayofweek(a.order_date) as day,   
        b.item_category, a.quantity  
    from orders a  
    join items b using(item_id)  
),  
sum_quantity_by_category as (  
    select  
        item_category,  
        day,  
        sum(quantity) as sum_quantity  
    from orders_with_group  
    group by item_category, day  
)  
  
select   
    distinct(a.item_category) as Category,  
    ifnull(Mon.sum_quantity, 0) as Monday,  
    ifnull(Tue.sum_quantity, 0) as Tuesday,  
    ifnull(Wen.sum_quantity, 0) as Wednesday,  
    ifnull(Thu.sum_quantity, 0) as Thursday,  
    ifnull(Fri.sum_quantity, 0) as Friday,  
    ifnull(Sat.sum_quantity, 0) as Saturday,  
    ifnull(Sun.sum_quantity, 0) as Sunday  
from items a  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=2) Mon using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=3) Tue using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=4) Wen using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=5) Thu using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=6) Fri using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=7) Sat using(item_category)  
left join ( select item_category, sum_quantity from sum_quantity_by_category where day=1) Sun using(item_category)  
order by a.item_category  
  
/*  
Map the return value of function dayOfWeek and text  
1 = Sunday  
2 = Monday  
3 = Tuesday  
4 = Wednesday  
5 = Thursday  
6 = Friday  
7 = Saturday  
*/
```
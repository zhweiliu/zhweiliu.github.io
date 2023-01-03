---
title: '[leetcode][Database][Hard] 1384. Total Sales Amount by Year'
date: '2022-12-06T09:05:48.196Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Product`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| product_id    | int     |  
| product_name  | varchar |  
+---------------+---------+  
product_id is the primary key for this table.  
product_name is the name of the product.
```

Table: `Sales`
```
+---------------------+---------+  
| Column Name         | Type    |  
+---------------------+---------+  
| product_id          | int     |  
| period_start        | date    |  
| period_end          | date    |  
| average_daily_sales | int     |  
+---------------------+---------+  
product_id is the primary key for this table.   
period_start and period_end indicate the start and end date for the sales period, and both dates are inclusive.  
The average_daily_sales column holds the average daily sales amount of the items for the period.  
The dates of the sales years are between 2018 to 2020.
```

Write an SQL query to report the total sales amount of each item for each year, with corresponding `product_name`, `product_id`, `product_name`, and `report_year`.

Return the result table `ordered` by `product_id` and `report_year`.

SQL Schema
```sql
Create table If Not Exists Product (product_id int, product_name varchar(30))  
Create table If Not Exists Sales (product_id int, period_start date, period_end date, average_daily_sales int)  
Truncate table Product  
insert into Product (product_id, product_name) values ('1', 'LC Phone ')  
insert into Product (product_id, product_name) values ('2', 'LC T-Shirt')  
insert into Product (product_id, product_name) values ('3', 'LC Keychain')  
Truncate table Sales  
insert into Sales (product_id, period_start, period_end, average_daily_sales) values ('1', '2019-01-25', '2019-02-28', '100')  
insert into Sales (product_id, period_start, period_end, average_daily_sales) values ('2', '2018-12-01', '2020-01-01', '10')  
insert into Sales (product_id, period_start, period_end, average_daily_sales) values ('3', '2019-12-01', '2020-01-31', '1')
```
## 解題思考

*   透過 `with clause` 分別建立 `report_year` 2018、2019 以及 2020 的產品銷售表格 `sell_2018`、 `sell_2019` 、 `sell_2020`  
    由於 `sales` 表格的 `period_start` 和 `period_end` 有跨越年度的可能性，而在其他的測試資料集中，也可能含有早於 2018 年的銷售資料，或者晚於 2020 年後的銷售資料，因此需要特別注意時間範圍的切割。
*   Union `sell_2018` 、 `sell_2019` 和 `sell_2020` ，並從 union 表格中取出每日平均銷售金額 `average_daily_sales` 乘以當年度整體銷售天數 `datediff(period_end, period_start)+1` ，便可得到當年度的平均銷售總額。

## 解決方案
```sql
with  
sell_2018 as (  
    select  
        product_id,  
        if( year(period_start) < '2018', '2018-01-01', period_start) as period_start,  
        if( year(period_end) > '2018', DATE_FORMAT(period_end,'2018-12-31'), period_end ) as period_end,  
        average_daily_sales  
    from sales  
    where year(period_start) <= '2018'   
),  
sell_2019 as (  
    select  
        product_id,  
        if( year(period_start) < '2019', '2019-01-01', period_start) as period_start,  
        if( year(period_end) > '2019', DATE_FORMAT(period_end,'2019-12-31'), period_end ) as period_end,  
        average_daily_sales  
    from sales  
    where year(period_start) <= '2019' and year(period_end) >= '2019'  
),  
sell_2020 as (  
    select  
        product_id,  
        if( year(period_start) < '2020', '2020-01-01', period_start) as period_start,  
        if( year(period_end) > '2020', DATE_FORMAT(period_end,'2020-12-31'), period_end ) as period_end,  
        average_daily_sales  
    from sales  
    where year(period_start) <= '2020' and year(period_end) >= '2020'  
),  
product_sell_info as (  
    select   
        a.product_id,  
        b.product_name,  
        date_format(a.period_start, "%Y") as report_year,  
        average_daily_sales * (datediff(a.period_end, a.period_start)+1) as total_amount  
    from (  
        select * from sell_2018  
        union  
        select * from sell_2019  
        union  
        select * from sell_2020  
    ) a  
    left join product b using(product_id)  
)  
  
select   
     product_id, product_name, report_year, total_amount  
from product_sell_info  
order by product_id, report_year
```
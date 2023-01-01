---
title: '[leetcode][Database][Hard] 579. Find Cumulative Salary of an Employee'
date: '2022-12-06T15:18:29.163Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Employee`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| id          | int  |  
| month       | int  |  
| salary      | int  |  
+-------------+------+  
(id, month) is the primary key for this table.  
Each row in the table indicates the salary of an employee in one month during the year 2020.
```
Write an SQL query to calculate the `cumulative salary summary` for every employee in a single unified table.

The `cumulative salary summary` for an employee can be calculated as follows:

*   For each month that the employee worked, `sum` up the salaries in `that month` and the `previous two months`. This is their `3-month sum` for that month. If an employee did not work for the company in previous months, their effective salary for those months is `0`.
*   Do `not` include the 3-month sum for the `most recent month` that the employee worked for in the summary.
*   Do `not` include the 3-month sum for any month the employee `did not work`.

Return the result table ordered by `id` in `ascending order`. In case of a tie, order it by `month` in `descending order`.

SQL Schema
```sql
Create table If Not Exists Employee (id int, month int, salary int)  
Truncate table Employee  
insert into Employee (id, month, salary) values ('1', '1', '20')  
insert into Employee (id, month, salary) values ('2', '1', '20')  
insert into Employee (id, month, salary) values ('1', '2', '30')  
insert into Employee (id, month, salary) values ('2', '2', '30')  
insert into Employee (id, month, salary) values ('3', '2', '40')  
insert into Employee (id, month, salary) values ('1', '3', '40')  
insert into Employee (id, month, salary) values ('3', '3', '60')  
insert into Employee (id, month, salary) values ('1', '4', '60')  
insert into Employee (id, month, salary) values ('3', '4', '70')  
insert into Employee (id, month, salary) values ('1', '7', '90')  
insert into Employee (id, month, salary) values ('1', '8', '90')
```
## 解題思考

*   題目要求輸出`每位員工`在`每個月分`的`累計薪資`，並且過濾每個員工最後一次薪資紀錄的月份。  
    定義 `累計薪資` → 該月份薪資與前兩個月的加總
```
+----+-------+--------+  
| id | month | Salary |  
+----+-------+--------+  
| 1  | 7     | 90     |  
| 1  | 4     | 130    |  
| 1  | 3     | 90     |  
| 1  | 2     | 50     |  
| 1  | 1     | 20     |  
| 2  | 1     | 20     |  
| 3  | 3     | 100    |  
| 3  | 2     | 40     |  
+----+-------+--------+
```
*   透過 `with clause` 建立 `month_dense_rank` 表格，以便找出每位員工需要過濾有薪資紀錄的最後一個月份。  
    利用 `dense_rank()` 函式，依據員工編號 `id` 與紀錄月份 `month` 降序，標註每位員工，每個月分薪資紀錄的排序結果。
*   透過 `with clause` 建立 `cumulate_month_salary` 表格，統計每位員工，每個月份的累積薪資。  
    利用子查詢關聯外部主表 `Employee`，並過濾出符合`子查詢 id=Employee.id` 以及`子查詢薪資紀錄月份 month` 落在主表 `Employee.month-2` 和 `Employee.month` 之間的 record set，依據該 record set 的員工編號 `id` 對該 record set的薪資 `salary` 進行加總。
*   利用子表查詢 `month_dense_rank` 過濾 `month_dense_rank.r_month > 1` 的 record set 作為主要表格，並關聯 `cumulate_month_salary` 表格，最後帶出符合題目要求的輸出結果 員工編號`id` 、 薪資月份`month` 、 累積薪資`salary` 。

## 解決方案
```sql
with   
month_dense_rnak as (  
    select   
        id,   
        month,   
        dense_rank() over(partition by id order by month desc) as r_month   
    from Employee  
),  
cumulate_month_salary as (  
    select  
        id,  
        month,  
        (  
            select   
                sum(salary)  
            from Employee   
            where id=e.id and month between e.month-2 and e.month  
            group by id  
        ) as salary  
    from Employee e  
)  
  
select  
    a.id, a.month, b.salary  
from (  
    select id, month from month_dense_rnak where r_month > 1  
) a  
join cumulate_month_salary b on b.id = a.id and b.month = a.month
```
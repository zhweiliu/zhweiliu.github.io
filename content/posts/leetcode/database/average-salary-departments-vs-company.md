---
title: '[leetcode][Database][Hard] 615. Average Salary: Departments VS Company'
date: '2022-12-06T14:33:17.216Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Salary`
```pre
+-------------+------+  
| Column Name   | Type |  
+-------------+------+  
| id            | int  |  
| employee_id  | int  |  
| amount        | int  |  
| pay_date     | date |  
+-------------+------+  

id is the primary key column for this table.  
Each row of this table indicates the salary of an employee in one month.  
employee_id is a foreign key from the Employee table.
```

Table: `Employee`
```pre
+---------------+------+  
| Column Name   | Type |  
+---------------+------+  
| employee_id   | int  |  
| department_id | int  |  
+---------------+------+  

employee_id is the primary key column for this table.  
Each row of this table indicates the department of an employee.
```

Write an SQL query to report the comparison result `(higher/lower/same)` of the average salary of employees in a department to the company’s average salary.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Salary (id int, employee_id int, amount int, pay_date date)  
Create table If Not Exists Employee (employee_id int, department_id int)  
Truncate table Salary  
insert into Salary (id, employee_id, amount, pay_date) values ('1', '1', '9000', '2017/03/31')  
insert into Salary (id, employee_id, amount, pay_date) values ('2', '2', '6000', '2017/03/31')  
insert into Salary (id, employee_id, amount, pay_date) values ('3', '3', '10000', '2017/03/31')  
insert into Salary (id, employee_id, amount, pay_date) values ('4', '1', '7000', '2017/02/28')  
insert into Salary (id, employee_id, amount, pay_date) values ('5', '2', '6000', '2017/02/28')  
insert into Salary (id, employee_id, amount, pay_date) values ('6', '3', '8000', '2017/02/28')  
Truncate table Employee  
insert into Employee (employee_id, department_id) values ('1', '1')  
insert into Employee (employee_id, department_id) values ('2', '2')  
insert into Employee (employee_id, department_id) values ('3', '2')
```

## 解題思考

*   題目要求列出每個月份，每個部門平均薪資，和整個公司平均薪資的比較結果，並顯示高 `higher`/ 相等 `same`/ 低 `lower`的結果。
```sql
+-----------+---------------+------------+  
| pay_month | department_id | comparison |  
+-----------+---------------+------------+  
| 2017-02   | 1             | same       |  
| 2017-03   | 1             | higher     |  
| 2017-02   | 2             | same       |  
| 2017-03   | 2             | lower      |  
+-----------+---------------+------------+
```
*   透過 `with clause` 建立 `company_salary` 表格，帶出每個月份公司的平均薪資。  
    選擇 `salary` 作為主要表格，因為 `company_salary` 表格不需要區分個別部門的員工薪資。
*   透過 `with clause` 建立 `department_salary` 表格，帶出每個月份，每個部門的平均薪資。  
    查詢 `employee` 作為主要表格並關聯 `salary` 表格，`employee.department_id` 可以作為區分員工部門的依據，便能計算出每個部門的平均薪資。
*   查詢 `company_salary` 作為主要表格並關聯 `department_salary` 表格，最後以 `pay_month` 升序、 `department_id` 升序輸出最後查詢結果。  
    利用 `if( condition, statement for condition true, statement for condition false )` 函示，並使用連續的 `if()` 判斷式來達成顯示高 `higher`/ 相等 `same`/ 低 `lower`的結果。

## 解決方案
```sql
with   
company_salary as (  
    select  
        date_format(pay_date, "%Y-%m") as pay_month,  
        avg(amount) as avg_salary  
    from salary  
    group by month(pay_date)  
),  
department_salary as (  
    select  
        a.department_id,   
        date_format(b.pay_date, "%Y-%m") as pay_month,  
        avg(b.amount) as avg_salary  
    from employee a  
    join salary b using (employee_id)  
    group by a.department_id, month(b.pay_date)  
)  
  
select  
    a.pay_month as pay_month,  
    b.department_id as department_id,  
    if(b.avg_salary>a.avg_salary, "higher", if(b.avg_salary < a.avg_salary, "lower", "same")) as comparison  
from company_salary a  
join department_salary b using(pay_month)  
order by a.pay_month, b.department_id
```
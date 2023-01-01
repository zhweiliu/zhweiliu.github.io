---
title: '[leetcode][Database][Hard] 185. Department Top Three Salaries'
date: '2022-12-06T17:21:44.783Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

題目

Table: `Employee`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| id           | int     |  
| name         | varchar |  
| salary       | int     |  
| departmentId | int     |  
+--------------+---------+  

id is the primary key column for this table.  
departmentId is a foreign key of the ID from the Department table.  
Each row of this table indicates the ID, name, and salary of an employee. It also contains the ID of their department.
```

Table: `Department`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| id          | int     |  
| name        | varchar |  
+-------------+---------+  
id is the primary key column for this table.  
Each row of this table indicates the ID of a department and its name.
```

A company’s executives are interested in seeing who earns the most money in each of the company’s departments. A `high earner` in a department is an employee who has a salary in the `top three unique` salaries for that department.

Write an SQL query to find the employees who are `high earners` in each of the departments.

Return the result table `in any order`.

SQL Schema
```sql
Create table If Not Exists Employee (id int, name varchar(255), salary int, departmentId int)  
Create table If Not Exists Department (id int, name varchar(255))  
Truncate table Employee  
insert into Employee (id, name, salary, departmentId) values ('1', 'Joe', '85000', '1')  
insert into Employee (id, name, salary, departmentId) values ('2', 'Henry', '80000', '2')  
insert into Employee (id, name, salary, departmentId) values ('3', 'Sam', '60000', '2')  
insert into Employee (id, name, salary, departmentId) values ('4', 'Max', '90000', '1')  
insert into Employee (id, name, salary, departmentId) values ('5', 'Janet', '69000', '1')  
insert into Employee (id, name, salary, departmentId) values ('6', 'Randy', '85000', '1')  
insert into Employee (id, name, salary, departmentId) values ('7', 'Will', '70000', '1')  
Truncate table Department  
insert into Department (id, name) values ('1', 'IT')  
insert into Department (id, name) values ('2', 'Sales')
```

解題思考

*   題目要求輸出每個部門中，收入排名前三高的員工。  
    若有收入相同且位於收入排名前三高的員工，也併入輸出結果。
```
+------------+----------+--------+  
| Department | Employee | Salary |  
+------------+----------+--------+  
| IT         | Max      | 90000  |  
| IT         | Joe      | 85000  |  
| IT         | Randy    | 85000  |  
| IT         | Will     | 70000  |  
| Sales      | Henry    | 80000  |  
| Sales      | Sam      | 60000  |  
+------------+----------+--------+
```
*   透過 `with clause` 建立 `employee_info` 表格，以提供每個部門內，每位員工的薪水排序結果。  
    使用 `dense_rank()` 函式進行排序，`dense_rank()` 會以`連續數字`的方式給予排序結果 `rn`。如下情況
```
+------------+----------+--------+--------------+  
| Department | Employee | Salary | dense_rank() |  
+------------+----------+--------+--------------+  
| IT         | Max      | 90000  | 1            |  
| IT         | Joe      | 85000  | 2            |  
| IT         | Randy    | 85000  | 2            |  
| IT         | Will     | 70000  | 3            |  
| Sales      | Henry    | 80000  | 1            |  
| Sales      | Sam      | 60000  | 2            |  
+------------+----------+--------+--------------+
```
*   查詢 `employee_info` 作為主要表格，過濾條件以找出 `employee_info.rn < 4` 的 record set，即找出每個部門，收入排名前三高的員工。

解決方案
```sql
with  
employee_info as (  
    select   
        Department.id as departmentId , Department.name as Department ,   
        Employee.name as Employee, Employee.Salary as Salary,  
        dense_rank() over(partition by Department.id order by Employee.Salary desc ) rn  
    from Employee  
    join Department on Department.id = Employee.departmentId   
)  
  
select   
    employee_info.Department, employee_info.Employee, employee_info.Salary  
from employee_info  
where employee_info.rn < 4
```
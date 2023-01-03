---
title: '[leetcode][Database][Hard] 569. Median Employee Salary'
date: '2022-12-06T16:48:49.861Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

題目

Table: `Employee`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| id           | int     |  
| company      | varchar |  
| salary       | int     |  
+--------------+---------+  
id is the primary key column for this table.  
Each row of this table indicates the company and the salary of one employee.
```
Write an SQL query to find the rows that contain the median salary of each company. While calculating the median, when you sort the salaries of the company, break the ties by `id`.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Employee (id int, company varchar(255), salary int)  
Truncate table Employee  
insert into Employee (id, company, salary) values ('1', 'A', '2341')  
insert into Employee (id, company, salary) values ('2', 'A', '341')  
insert into Employee (id, company, salary) values ('3', 'A', '15')  
insert into Employee (id, company, salary) values ('4', 'A', '15314')  
insert into Employee (id, company, salary) values ('5', 'A', '451')  
insert into Employee (id, company, salary) values ('6', 'A', '513')  
insert into Employee (id, company, salary) values ('7', 'B', '15')  
insert into Employee (id, company, salary) values ('8', 'B', '13')  
insert into Employee (id, company, salary) values ('9', 'B', '1154')  
insert into Employee (id, company, salary) values ('10', 'B', '1345')  
insert into Employee (id, company, salary) values ('11', 'B', '1221')  
insert into Employee (id, company, salary) values ('12', 'B', '234')  
insert into Employee (id, company, salary) values ('13', 'C', '2345')  
insert into Employee (id, company, salary) values ('14', 'C', '2645')  
insert into Employee (id, company, salary) values ('15', 'C', '2645')  
insert into Employee (id, company, salary) values ('16', 'C', '2652')  
insert into Employee (id, company, salary) values ('17', 'C', '65')
```
解題思考

*   題目要求計算並且輸出每間公司的薪資中位數。
```
+----+---------+--------+  
| id | company | salary |  
+----+---------+--------+  
| 5  | A       | 451    |  
| 6  | A       | 513    |  
| 12 | B       | 234    |  
| 9  | B       | 1154   |  
| 14 | C       | 2645   |  
+----+---------+--------+
```
*   做子表 `a`查詢，每間公司的員工總數 `cnt` ，並利用 `row_numbner()`依據每間公司員工薪水 `salary` 升序，標註排序結果 `row_num`。
*   利用子表 `a` 作為主要表格，過濾`row_num`以找出介於 `cnt div 2`和 `(cnt div 2)+1` 的 record set，即每間公司的新增中位數。

解決方案
```sql
select   
    a.id, a.company, a.salary  
from (  
    select  
        *,  
        count(id) over(PARTITION  by company) as cnt,  
        row_number() over(PARTITION  by company  order by salary) as row_num  
    from employee  
) a  
where a.row_num between a.cnt div 2 and (a.cnt div 2)+1
```
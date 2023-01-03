---
title: '[leetcode][Database][Hard] 618. Students Report By Geography'
date: '2022-12-06T14:08:14.796Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Student`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| name        | varchar |  
| continent   | varchar |  
+-------------+---------+  
There is no primary key for this table. It may contain duplicate rows.  
Each row of this table indicates the name of a student and the continent they came from.
```
A school has students from Asia, Europe, and America.

Write an SQL query to [pivot](https://en.wikipedia.org/wiki/Pivot_table) the continent column in the `Student` table so that each name is `sorted alphabetically` and displayed underneath its corresponding continent. The output headers should be `America`, `Asia`, and `Europe`, respectively.

The test cases are generated so that the student number from America is not less than either Asia or Europe.

SQL Schema
```sql
Create table If Not Exists Student (name varchar(50), continent varchar(7))  
Truncate table Student  
insert into Student (name, continent) values ('Jane', 'America')  
insert into Student (name, continent) values ('Pascal', 'Europe')  
insert into Student (name, continent) values ('Xi', 'Asia')  
insert into Student (name, continent) values ('Jack', 'America')
```
## 解題思考

*   題目要求輸出樞紐表格，該樞紐表格需要使用 `America` 、 `Asia` 和 `Europe` ，並符合該大洲的學生名字 `name` 列表。
```
+---------+------+--------+  
| America | Asia | Europe |  
+---------+------+--------+  
| Jack    | Xi   | Pascal |  
| Jane    | null | null   |  
+---------+------+--------+
```
*   使用 `left join` 便能簡單的達成題目要求。  
    需要著手處理的問題點 : 如何讓學生名字並存在同一個 row 。  
    利用 `row_number()` 函式，先將學生依據大洲分類，並給予每個大洲內的學生列表標記排序數字，最後透過選擇學生人數最多的大洲作為主要表格，依次 `left join` 其他大洲的學生列表，並以 `row_number()` 的標記數字做為表格關聯的條件。

## 解決方案
```sql
with  
from_america as (select name, row_number() over() as rn from student where continent = 'America' order by name asc),  
from_asia as (select name, row_number() over() as rn from student where continent = 'Asia' order by name asc),  
from_europe as (select name, row_number() over() as rn from student where continent = 'Europe' order by name asc)  
  
select  
    a.name as America, b.name as Asia, c.name as Europe  
from from_america a  
left join from_asia b using(rn)  
left join from_europe c using(rn)
```
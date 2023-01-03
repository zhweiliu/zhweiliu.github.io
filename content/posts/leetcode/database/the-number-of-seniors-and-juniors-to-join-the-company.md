---
title: >-
  [leetcode][Database][Hard] 2004. The Number of Seniors and Juniors to Join the
  Company
date: '2022-12-05T09:07:42.185Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Candidates`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| employee_id | int  |  
| experience  | enum |  
| salary      | int  |  
+-------------+------+  
employee_id is the primary key column for this table.  
experience is an enum with one of the values ('Senior', 'Junior').  
Each row of this table indicates the id of a candidate, their monthly salary, and their experience.
```
A company wants to hire new employees. The budget of the company for the salaries is `$70000`. The company's criteria for hiring are:

1.  Hiring the largest number of seniors.
2.  After hiring the maximum number of seniors, use the remaining budget to hire the largest number of juniors.

Write an SQL query to find the number of seniors and juniors hired under the mentioned criteria.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Candidates (employee_id int, experience ENUM('Senior', 'Junior'), salary int)  
Truncate table Candidates  
insert into Candidates (employee_id, experience, salary) values ('1', 'Junior', '10000')  
insert into Candidates (employee_id, experience, salary) values ('9', 'Junior', '10000')  
insert into Candidates (employee_id, experience, salary) values ('2', 'Senior', '20000')  
insert into Candidates (employee_id, experience, salary) values ('11', 'Senior', '20000')  
insert into Candidates (employee_id, experience, salary) values ('13', 'Senior', '50000')  
insert into Candidates (employee_id, experience, salary) values ('4', 'Junior', '40000')
```

## 解題思考

*   樞紐表要求輸出 `Senior`雇員和 `Junior`雇員的可招聘人數。  
    利用子查詢功能，分別帶出 `Senior`和 `Junior`的招聘結果，以作為樞紐表的欄位值。
*   利用 `with clause` 建立 `Senior`雇員、 `Junior`雇員的薪水排序表。  
    因題目要求必須先招聘盡可能多的 Senior 雇員，再利用剩餘預算朝聘盡可能多的 Junior 雇員
*   薪水排序表中，用 `row_number()` 標註排序結果，排序主要條件為薪資 `salary` 升序；同時排序結果可做為第幾位雇員，即可招聘的雇員數量
*   薪水排序表中，依據薪資 `salary` 和雇員編號 `employee_id` 累加雇員薪資；同時累加雇員薪資可做為已消耗的預算 `budget`
*   查詢招聘 `senior` 花費的預算 `cumulate_budget` 。  
    取 `cumulate_budget <= budget` 的 record set，並從中找出 `max(rn)` 與對應的 `senior.cumulate_budget`
*   計算可分配給`Junior`招聘的剩餘預算 `remaining = budget - senior.cumulate_budget` ，取 `junior.cumulate_budget <= emaining` 的 record set，並從中找出 `max(rn)` 與對應的 `junior.cumulate_budget`

## 解決方案
```sql
with  
seniors_salary_rank as (  
    select  
        employee_id, salary,  
        row_number() over(order by salary, employee_id) as rn,  
        sum(salary) over(order by salary, employee_id) as cumulate_budget  
    from candidates  
    where experience = 'senior'  
    order by salary  
),  
juniors_salary_rank as (  
    select  
        employee_id, salary,  
        row_number() over(order by salary, employee_id asc) as rn,  
        sum(salary) over(order by salary, employee_id) as cumulate_budget  
    from candidates  
    where experience = 'junior'  
    order by salary  
),  
hire_seniors as (  
    select  
        b.rn, 70000 - b.cumulate_budget as remain_budget  
    from (  
        select  
            max(cumulate_budget) as cumulate_budget  
        from seniors_salary_rank  
        where cumulate_budget <= 70000  
    ) a   
    join seniors_salary_rank b using(cumulate_budget)  
      
),  
hire_junior as (  
    select  
        b.rn, ifnull((select remain_budget from hire_seniors limit 1),70000) - b.cumulate_budget as remain_budget  
    from (  
        select  
            max(cumulate_budget) as cumulate_budget  
        from juniors_salary_rank  
        where cumulate_budget <= ifnull((select remain_budget from hire_seniors limit 1),70000)  
    ) a  
    join juniors_salary_rank b using(cumulate_budget)  
)  
  
select  
    distinct(experience) as experience,  
    case   
        when experience = 'Senior' then ifnull((select rn from hire_seniors limit 1 ),0)  
        when experience = 'Junior' then ifnull((select rn from hire_junior limit 1 ),0)  
    end as accepted_candidates  
from candidates a
```
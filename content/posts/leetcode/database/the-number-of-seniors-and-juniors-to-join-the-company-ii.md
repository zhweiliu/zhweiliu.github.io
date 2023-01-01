---
title: >-
  [leetcode][Database][Hard] 2010. The Number of Seniors and Juniors to Join the
  Company II
date: '2022-12-17T00:11:43.958Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

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
The salary of each candidate is guaranteed to be unique.
```
A company wants to hire new employees. The budget of the company for the salaries is `$70000`. The company's criteria for hiring are:

1.  Keep hiring the senior with the smallest salary until you cannot hire any more seniors.
2.  Use the remaining budget to hire the junior with the smallest salary.
3.  Keep hiring the junior with the smallest salary until you cannot hire any more juniors.

Write an SQL query to find the ids of seniors and juniors hired under the mentioned criteria.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Candidates (employee_id int, experience ENUM('Senior', 'Junior'), salary int)  
Truncate table Candidates  
insert into Candidates (employee_id, experience, salary) values ('1', 'Junior', '10000')  
insert into Candidates (employee_id, experience, salary) values ('9', 'Junior', '15000')  
insert into Candidates (employee_id, experience, salary) values ('2', 'Senior', '20000')  
insert into Candidates (employee_id, experience, salary) values ('11', 'Senior', '16000')  
insert into Candidates (employee_id, experience, salary) values ('13', 'Senior', '50000')  
insert into Candidates (employee_id, experience, salary) values ('4', 'Junior', '40000')
```

## Idea

The query result format is in the following example.
```
+-------------+  
| employee_id |  
+-------------+  
| 11          |  
| 2           |  
| 1           |  
| 9           |  
+-------------+
```
Fulfill requirements :

The thinking process like as _[leetcode]_[[Database][Hard] 2004. The Number of Seniors and Juniors to Join the Company](https://medium.com/@zhengwei-liu/leetcode-database-hard-2004-the-number-of-seniors-and-juniors-to-join-the-company-f80748cea908?source=your_stories_page-------------------------------------).

So we can calculating the cumulative salary, sorted the salary of each employee and partition by different experience at first.

Next, finding the max cumulative salary which less than budget `$70,000` and take `employee_id` which are fitting the above condition. Also, finding the junior employees by remaining budget from hired senior employees.

## Solution
```sql
with  
salary_cumulative as (  
    select  
        employee_id, experience, salary,  
        sum(salary) over(partition by experience order by salary) as cumulative_salary  
    from Candidates  
),  
senior_hiring as (  
    select -1 as employee_id, 0 as cumulative_salary  
    union  
    select  
        employee_id, cumulative_salary  
    from salary_cumulative  
    where experience = 'Senior'  
        and 70000 - cumulative_salary >=0  
),  
junior_hiring as (  
   select employee_id from salary_cumulative  
   join (  
        select   
            70000-max(cumulative_salary) as remaining  
        from senior_hiring  
   ) senior  
   where experience = 'Junior'  
   and  remaining-cumulative_salary>=0  
)  
  
  
select employee_id from senior_hiring where employee_id > 0  
union  
select employee_id from junior_hiring
```
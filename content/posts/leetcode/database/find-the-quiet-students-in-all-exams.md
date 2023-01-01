---
title: '[leetcode][Database][Hard] 1412. Find the Quiet Students in All Exams'
date: '2022-12-13T01:08:00.986Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Student`
```
+---------------------+---------+  
| Column Name         | Type    |  
+---------------------+---------+  
| student_id          | int     |  
| student_name        | varchar |  
+---------------------+---------+  
student_id is the primary key for this table.  
student_name is the name of the student.
```

Table: `Exam`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| exam_id       | int     |  
| student_id    | int     |  
| score         | int     |  
+---------------+---------+  
(exam_id, student_id) is the primary key for this table.  
Each row of this table indicates that the student with student_id had a score points in the exam with id exam_id.
```

A `quiet student` is the one who took at least one exam and did not score the high or the low score.

Write an SQL query to report the students `(student_id, student_name)` being quiet in all exams. Do not return the student who has never taken any exam.

Return the result table `ordered` by `student_id`.

SQL Schema
```sql
Create table If Not Exists Student (student_id int, student_name varchar(30))  
Create table If Not Exists Exam (exam_id int, student_id int, score int)  
Truncate table Student  
insert into Student (student_id, student_name) values ('1', 'Daniel')  
insert into Student (student_id, student_name) values ('2', 'Jade')  
insert into Student (student_id, student_name) values ('3', 'Stella')  
insert into Student (student_id, student_name) values ('4', 'Jonathan')  
insert into Student (student_id, student_name) values ('5', 'Will')  
Truncate table Exam  
insert into Exam (exam_id, student_id, score) values ('10', '1', '70')  
insert into Exam (exam_id, student_id, score) values ('10', '2', '80')  
insert into Exam (exam_id, student_id, score) values ('10', '3', '90')  
insert into Exam (exam_id, student_id, score) values ('20', '1', '80')  
insert into Exam (exam_id, student_id, score) values ('30', '1', '70')  
insert into Exam (exam_id, student_id, score) values ('30', '3', '80')  
insert into Exam (exam_id, student_id, score) values ('30', '4', '90')  
insert into Exam (exam_id, student_id, score) values ('40', '1', '60')  
insert into Exam (exam_id, student_id, score) values ('40', '2', '70')  
insert into Exam (exam_id, student_id, score) values ('40', '4', '80')
```
## Idea

The query result format is in the following example.
```
+-------------+---------------+  
| student_id  | student_name  |  
+-------------+---------------+  
| 2           | Jade          |  
+-------------+---------------+
```
It has easy way to find quiet students by `ranking score` of each exam, `getting best and worst rank` of each exam, `marking best and worst for each student` of each exam if they are best score or worst score at the exam.

Finally, listing the students who `never got best mark and worst mark of all exam`.

## Solution
```sql
with  
rank_score_of_exam as (  
    select  
        a.exam_id, a.student_id, b.student_name,  
        dense_rank() over(partition by a.exam_id order by a.score) as score_rk  
    from exam a  
    left join student b using(student_id)  
),  
rank_of_exam as (  
    select  
        exam_id,  
        max(score_rk) as high,  
        min(score_rk) as low  
    from rank_score_of_exam  
    group by exam_id  
),  
mark_best as (  
    select  
        a.exam_id, a.student_id, a.student_name,  
        if(a.score_rk=b.high, 1, 0) as mark  
    from rank_score_of_exam a  
    left join rank_of_exam b using(exam_id)  
),  
mark_worst as (  
    select  
        a.exam_id, a.student_id, a.student_name,  
        if(a.score_rk=b.low, 1, 0) as mark  
    from rank_score_of_exam a  
    left join rank_of_exam b using(exam_id)  
)  
  
select  
    student_id, student_name  
from (  
    select exam_id, student_id, student_name, mark from mark_best  
    union all  
    select exam_id, student_id, student_name, mark from mark_worst  
) opt  
group by student_id, student_name  
having sum(mark) = 0  
order by student_id
```
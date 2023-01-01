---
title: '[leetcode][Database][Hard] 1767. Find the Subtasks That Did Not Execute'
date: '2022-12-15T17:53:09.861Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Tasks`
```
+----------------+---------+  
| Column Name    | Type    |  
+----------------+---------+  
| task_id        | int     |  
| subtasks_count | int     |  
+----------------+---------+  
task_id is the primary key for this table.  
Each row in this table indicates that task_id was divided into subtasks_count subtasks labeled from 1 to subtasks_count.  
It is guaranteed that 2 <= subtasks_count <= 20.
```

Table: `Executed`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| task_id       | int     |  
| subtask_id    | int     |  
+---------------+---------+  
(task_id, subtask_id) is the primary key for this table.  
Each row in this table indicates that for the task task_id, the subtask with ID subtask_id was executed successfully.  
It is guaranteed that subtask_id <= subtasks_count for each task_id.
```

Write an SQL query to report the IDs of the missing subtasks for each `task_id`.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Tasks (task_id int, subtasks_count int)  
Create table If Not Exists Executed (task_id int, subtask_id int)  
Truncate table Tasks  
insert into Tasks (task_id, subtasks_count) values ('1', '3')  
insert into Tasks (task_id, subtasks_count) values ('2', '2')  
insert into Tasks (task_id, subtasks_count) values ('3', '4')  
Truncate table Executed  
insert into Executed (task_id, subtask_id) values ('1', '2')  
insert into Executed (task_id, subtask_id) values ('3', '1')  
insert into Executed (task_id, subtask_id) values ('3', '2')  
insert into Executed (task_id, subtask_id) values ('3', '3')  
insert into Executed (task_id, subtask_id) values ('3', '4')
```
## Idea

The query result format is in the following example.
```
+---------+------------+  
| task_id | subtask_id |  
+---------+------------+  
| 1       | 1          |  
| 1       | 3          |  
| 2       | 1          |  
| 2       | 2          |  
+---------+------------+
```
Fulfill requirements :

Using the `with recursive` to generate a serial number for `subtasks_id` from `1` to `20`, then finding not executed subtasks with condition `ifnull(task_id)` or `ifnull(subtask_id)` which `subtask_id` records not in table `Executed`.

## Solution
```sql
with  
recursive cte_subtasks_sn as (  
    SELECT 1 AS subtask_id   
    UNION ALL   
    SELECT subtask_id + 1 FROM cte_subtasks_sn WHERE subtask_id < 20   
),  
cte_subtasks_count as (  
    select  
        a.task_id as task_id,  
        b.subtask_id as subtask_id  
    from Tasks a, cte_subtasks_sn b  
    where b.subtask_id <= a.subtasks_count  
)  
  
select  
    a.task_id, a.subtask_id  
from cte_subtasks_count a  
left join Executed b using(task_id, subtask_id)  
where ifnull(b.task_id, -1) = -1 or ifnull(b.subtask_id, -1) = -1  
order by task_id, subtask_id
```
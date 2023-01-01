---
title: '[leetcode][Database][Hard]1972. First and Last Call On the Same Day'
date: '2022-12-06T05:39:34.595Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Calls`
```
+--------------+----------+  
| Column Name  | Type     |  
+--------------+----------+  
| caller_id    | int      |  
| recipient_id | int      |  
| call_time    | datetime |  
+--------------+----------+  
(caller_id, recipient_id, call_time) is the primary key for this table.  
Each row contains information about the time of a phone call between caller_id and recipient_id.
```
Write an SQL query to report the IDs of the users whose first and last calls on `any day` were with `the same person`. Calls are counted regardless of being the caller or the recipient.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Calls (caller_id int, recipient_id int, call_time datetime)  
Truncate table Calls  
insert into Calls (caller_id, recipient_id, call_time) values ('8', '4', '2021-08-24 17:46:07')  
insert into Calls (caller_id, recipient_id, call_time) values ('4', '8', '2021-08-24 19:57:13')  
insert into Calls (caller_id, recipient_id, call_time) values ('5', '1', '2021-08-11 05:28:44')  
insert into Calls (caller_id, recipient_id, call_time) values ('8', '3', '2021-08-17 04:04:15')  
insert into Calls (caller_id, recipient_id, call_time) values ('11', '3', '2021-08-17 13:07:00')  
insert into Calls (caller_id, recipient_id, call_time) values ('8', '11', '2021-08-17 22:22:22')
```
## 解題思考

*   建立 `user_calls`的 `with clause`，以提供最後輸出的表格使用。  
    `user_calls`將每筆 `calls` 中的通話紀錄拆分成兩筆 record ，即該筆通話紀錄的兩位 user 分別以自己的角度，紀錄該筆通話的時間以及通話的對象  
    `A ← communicate → B` : `A → B` and `B → A`
*   建立 `rank_calls` 的 `with clause` ，對 `user_calls` 中的通話紀錄進行排序。  
    對每個 `user_id` 的所有 `call_time` 通話時間做 升序 和 降序，以便找出第一筆通話 `first call` 和最後一筆通話 `last call` 。
*   對 `rank_calls` 每個 `user_id` 的 `first call` 和 `last call` 進行統計，若不重複的通話對象只有一位，則可以找出 `first call` 和 `last call` 都是同一人的 `user_id`

## 解決方案
```sql
with  
user_calls as (  
    select caller_id as user_id, call_time, recipient_id from calls  
    union  
    select recipient_id as user_id, call_time, caller_id as recipient_id from calls  
),  
rank_calls as (  
    select  
        user_id, recipient_id,  
        date(call_time) as day,  
        dense_rank() over(partition by user_id, date(call_time) order by call_time asc) as rn,  
        dense_rank() over(partition by user_id, date(call_time) order by call_time desc) as rk  
    from user_calls  
)  
  
select distinct  
    user_id  
from rank_calls  
where rn=1 or rk=1  
group by user_id, day  
having count(distinct recipient_id) = 1
```
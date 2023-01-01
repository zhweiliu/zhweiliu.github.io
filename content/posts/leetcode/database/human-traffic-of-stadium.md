---
title: '[leetcode][Database][Hard] 601. Human Traffic of Stadium'
date: '2022-12-06T14:55:01.530Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Stadium`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| id            | int     |  
| visit_date    | date    |  
| people        | int     |  
+---------------+---------+  
visit_date is the primary key for this table.  
Each row of this table contains the visit date and visit id to the stadium with the number of people during the visit.  
No two rows will have the same visit_date, and as the id increases, the dates increase as well.
```

Write an SQL query to display the records with three or more rows with `consecutive` `id`'s, and the number of people is greater than or equal to 100 for each.

Return the result table ordered by `visit_date` in `ascending order`.

SQL Schema
```sql
Create table If Not Exists Stadium (id int, visit_date DATE NULL, people int)  
Truncate table Stadium  
insert into Stadium (id, visit_date, people) values ('1', '2017-01-01', '10')  
insert into Stadium (id, visit_date, people) values ('2', '2017-01-02', '109')  
insert into Stadium (id, visit_date, people) values ('3', '2017-01-03', '150')  
insert into Stadium (id, visit_date, people) values ('4', '2017-01-04', '99')  
insert into Stadium (id, visit_date, people) values ('5', '2017-01-05', '145')  
insert into Stadium (id, visit_date, people) values ('6', '2017-01-06', '1455')  
insert into Stadium (id, visit_date, people) values ('7', '2017-01-07', '199')  
insert into Stadium (id, visit_date, people) values ('8', '2017-01-09', '188')
```
## 解題思考

*   題目要求輸出`連續三日以上`，拜訪`人數達到 100 以上` 的日期與人數。
```
+------+------------+-----------+  
| id   | visit_date | people    |  
+------+------------+-----------+  
| 5    | 2017-01-05 | 145       |  
| 6    | 2017-01-06 | 1455      |  
| 7    | 2017-01-07 | 199       |  
| 8    | 2017-01-09 | 188       |  
+------+------------+-----------+
```
*   透過 `with clause` 建立 `visit_thresgold` 表格，利用 `lead()`函示擴增`後兩日瀏覽人數`欄位 `next_1`、`next_2`，利用`lag()` 函示擴增`前兩日瀏覽人數`欄位 `prev_1`、`prev_2`，以 `over_threshold` 作為`當日瀏覽人數`欄位。  
    判斷每個欄位`瀏覽人數`是否達到 `100 以上`，  
    若人數達到 `100 以上`，標記為 `1`；  
    若人數`不滿 100`，則標記為 `0`
*   查詢 stadium 作為主要表格並關聯 visit_threshold，若以下三條判斷式有其中一條成立，則該日日期與瀏覽人數需要包含於輸出結果內  
    `prev_2 + prev_1 + over_threshold > 2`  
    `prev_1 + over_threshold + next_1 > 2`  
    `over_threshold + next_1 + next_2 > 2`
*   當看到`連續`這個關鍵字時，我會想到應該使用 `lead()` 函式或 `lag()` 函式。lead() 和 lag() 對應的是當前查詢的 record set ；若在子查詢或子表內使用，則對應子查詢或子表的資料範圍。  
    `lead( column, offset ) over( [partition by column1, column2, …] [order by column1, column2, …])`  
    `lag( column, offset) over( [partition by column1, column2, …] [order by column1, column2, …])`
*   Leetcode 官方對於這題所提供的解決方案是透過 `self join T` 兩次`且不給予關聯條件`，並判斷 `T1.people` 、 `T2.people` 和 `T3.people` 是否都有達到 100 以上。

## 解決方案
```sql
with   
visit_threshold as (  
    select  
        id,   
        if(people >= 100, 1, 0) as over_threshold,  
        if(lead(people, 1) over() >=100, 1,0) as next_1,  
        if(lead(people, 2) over() >=100, 1,0) as next_2,  
        if(lag(people, 1) over() >=100, 1,0) as prev_1,  
        if(lag(people, 2) over() >=100, 1,0) as prev_2  
    from stadium  
)  
  
select   
    A.id, A.visit_date , A.people   
from stadium A  
join visit_threshold B using(id)  
where B.over_threshold+B.next_1+B.next_2 > 2  
or B.prev_1+B.prev_2+B.over_threshold > 2  
or B.prev_1+B.over_threshold+B.next_1 > 2
```
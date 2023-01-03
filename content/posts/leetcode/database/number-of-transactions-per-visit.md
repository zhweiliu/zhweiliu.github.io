---
title: '[leetcode][Database][Hard] 1336. Number of Transactions per Visit'
date: '2022-12-12T16:38:05.305Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## Description

Table: `Visits`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| user_id       | int     |  
| visit_date    | date    |  
+---------------+---------+  
(user_id, visit_date) is the primary key for this table.  
Each row of this table indicates that user_id has visited the bank in visit_date.
```

Table: `Transactions`
```
+------------------+---------+  
| Column Name      | Type    |  
+------------------+---------+  
| user_id          | int     |  
| transaction_date | date    |  
| amount           | int     |  
+------------------+---------+  
There is no primary key for this table, it may contain duplicates.  
Each row of this table indicates that user_id has done a transaction of amount in transaction_date.  
It is guaranteed that the user has visited the bank in the transaction_date.(i.e The Visits table contains (user_id, transaction_date) in one row)
```

A bank wants to draw a chart of the number of transactions bank visitors did in one visit to the bank and the corresponding number of visitors who have done this number of transaction in one visit.

Write an SQL query to find how many users visited the bank and didn’t do any transactions, how many visited the bank and did one transaction and so on.

The result table will contain two columns:

*   `transactions_count` which is the number of transactions done in one visit.
*   `visits_count` which is the corresponding number of users who did `transactions_count` in one visit to the bank.

`transactions_count` should take all values from `0` to `max(transactions_count)` done by one or more users.

Return the result table ordered by `transactions_count`.

SQL Schema
```sql
Create table If Not Exists Visits (user_id int, visit_date date)  
Create table If Not Exists Transactions (user_id int, transaction_date date, amount int)  
Truncate table Visits  
insert into Visits (user_id, visit_date) values ('1', '2020-01-01')  
insert into Visits (user_id, visit_date) values ('2', '2020-01-02')  
insert into Visits (user_id, visit_date) values ('12', '2020-01-01')  
insert into Visits (user_id, visit_date) values ('19', '2020-01-03')  
insert into Visits (user_id, visit_date) values ('1', '2020-01-02')  
insert into Visits (user_id, visit_date) values ('2', '2020-01-03')  
insert into Visits (user_id, visit_date) values ('1', '2020-01-04')  
insert into Visits (user_id, visit_date) values ('7', '2020-01-11')  
insert into Visits (user_id, visit_date) values ('9', '2020-01-25')  
insert into Visits (user_id, visit_date) values ('8', '2020-01-28')  
Truncate table Transactions  
insert into Transactions (user_id, transaction_date, amount) values ('1', '2020-01-02', '120')  
insert into Transactions (user_id, transaction_date, amount) values ('2', '2020-01-03', '22')  
insert into Transactions (user_id, transaction_date, amount) values ('7', '2020-01-11', '232')  
insert into Transactions (user_id, transaction_date, amount) values ('1', '2020-01-04', '7')  
insert into Transactions (user_id, transaction_date, amount) values ('9', '2020-01-25', '33')  
insert into Transactions (user_id, transaction_date, amount) values ('9', '2020-01-25', '66')  
insert into Transactions (user_id, transaction_date, amount) values ('8', '2020-01-28', '1')  
insert into Transactions (user_id, transaction_date, amount) values ('9', '2020-01-25', '99')
```
## Idea

The output requires result table contains 2 columns : `transactions_count` and `visits_count` , but also the column `transactions_count` range between `0` to `max(transactions_count)`. Finally, output the result table ordered by transactions_count.

for example:
```
+--------------------+--------------+  
| transactions_count | visits_count |  
+--------------------+--------------+  
| 0                  | 4            |  
| 1                  | 5            |  
| 2                  | 0            |  
| 3                  | 1            |  
+--------------------+--------------+
```
Fultill requirements:

1.  Generate a serial number table start with `0` by function `row_number()` of `transactions` table due to `max(transactions_count)` should be `less tahn or equals to` the number of records of `transactions` table.  
    Name this `with clause` as `sn` .
2.  Compute the `transactions_count` from records of transactions table, where the records matches the `user_id` from `visits` table, but also the `transaction_date` in `visit_date`.   
    Name this `with clause` as .
3.  Joint `sn` and `cte_1`, where `row_number()` of `sn` should be less than or equals to`max(cte_1.transactions_count)` .   
    It will help to confirm all of `sn.rn` rows where the`sn.rn` less than or equals tothan `max(cte_1.transactions_count)` should be in result table, because the `cte_1` will not include row if the `visits_count` are zero times.  
    Name this `with clause` as `cte_2`.
4.  Finally, named column `visits_count` for counting how many users group by `cte_2.transactions_count` , and output it.

## Solution
```sql
with  
sn as (  
    select row_number() over() -1 rn from transactions   
),  
cte_1 as (  
    select  
        a.user_id,  
        a.visit_date,  
        count(b.transaction_date)as transactions_count  
    from visits a  
    left join transactions b on b.user_id = a.user_id and b.transaction_date = a.visit_date  
    group by a.user_id, a.visit_date  
),  
cte_2 as (  
    select user_id, visit_date, transactions_count from cte_1  
    union all  
    select null, null, rn from sn where rn < (select max(transactions_count) from cte_1)  
)  
  
  
select  
    transactions_count, count(user_id) as visits_count  
from cte_2  
group by transactions_count  
order by transactions_count
```
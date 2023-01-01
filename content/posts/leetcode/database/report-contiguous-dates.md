---
title: '[leetcode][Database][Hard] 1225. Report Contiguous Dates'
date: '2022-12-07T02:38:30.546Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Failed`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| fail_date    | date    |  
+--------------+---------+  
fail_date is the primary key for this table.  
This table contains the days of failed tasks.
```

Table: `Succeeded`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| success_date | date    |  
+--------------+---------+  
success_date is the primary key for this table.  
This table contains the days of succeeded tasks.
```

A system is running one task `every day`. Every task is independent of the previous tasks. The tasks can fail or succeed.

Write an SQL query to generate a report of `period_state` for each continuous interval of days in the period from `2019-01-01` to `2019-12-31`.

`period_state` is _'_`failed'` if tasks in this interval failed or `'succeeded'` if tasks in this interval succeeded. Interval of days are retrieved as `start_date` and `end_date.`

Return the result table ordered by `start_date`.

SQL Schema
```sql
Create table If Not Exists Failed (fail_date date)  
Create table If Not Exists Succeeded (success_date date)  
Truncate table Failed  
insert into Failed (fail_date) values ('2018-12-28')  
insert into Failed (fail_date) values ('2018-12-29')  
insert into Failed (fail_date) values ('2019-01-04')  
insert into Failed (fail_date) values ('2019-01-05')  
Truncate table Succeeded  
insert into Succeeded (success_date) values ('2018-12-30')  
insert into Succeeded (success_date) values ('2018-12-31')  
insert into Succeeded (success_date) values ('2019-01-01')  
insert into Succeeded (success_date) values ('2019-01-02')  
insert into Succeeded (success_date) values ('2019-01-03')  
insert into Succeeded (success_date) values ('2019-01-06')
```

## 解題思考 (Variable 版本)

題目要求輸出 `period_state` 、 `start_date` 和 `end_date` 欄位的表格。  
其中將連續日期都為相同 state 的定義為一個 period，且 `start_date` 和 `end_date` 將表示該 period 的起始日期和結束日期。
```
+--------------+--------------+--------------+  
| period_state | start_date   | end_date     |  
+--------------+--------------+--------------+  
| succeeded    | 2019-01-01   | 2019-01-03   |  
| failed       | 2019-01-04   | 2019-01-05   |  
| succeeded    | 2019-01-06   | 2019-01-06   |  
+--------------+--------------+--------------+
```
*   最開始，我的想法是利用 variable，在每次 `period_state` 變換時，透過變數將該 `period_state`第一個日期作為錨點 `anchor` ，再使用 `rank()` 對 `start_date` 做升序排序 `rn`。
*   透過 `with clause` 建立 `status` 表格，聯合 `Failed` 表格和 `Successed` 表格，篩選出 `fail_date` 、 `success_date` 介於 `2019–01–01` 至 `2019-12-31` 的資料集。
*   透過 `with clause` 建立 `date_anchor` 表格，擴增 `anchor_date`欄位以便後續找出 `end_date` 。  
    `variable current_state` ← 儲存當前 record 的 period_state  
    `variable date_anchor` ← 儲存當前 period 的第一個日期  
    透過子查詢初始化 `variable current_state` 和 `variable date_anchor`
*   查詢 `date_anchor` 以建立子查詢表格 `a` 。  
    子查詢表格中，利用 `rank()` 對 `start_date` 做升序，以標記排序結果 `rn` 。並利用 `max(start_date) over(partition by anchor_date)` 取得每個 period 中最大的 `start_date` ，即該 period 的 `end_date` 。
*   查詢 `a` 作為主要表格，利用 `group by end_date` 和 `having min(rn)` 以篩選出符合輸出條件的資料。

## 解決方案 (Variable 版本)
```sql
with  
status as (  
    select  
        period_state, start_date  
    from (  
        select fail_date as start_date, 'failed' as period_state from Failed  
        union all  
        select success_date as start_date, 'succeeded' as period_state from Succeeded  
    ) tmp  
    where start_date between '2019-01-01' and '2019-12-31'  
    order by start_date  
),  
date_anchor as (  
    select  
        period_state, start_date, anchor_date  
    from(  
        select   
            a.period_state,  
            a.start_date,  
            if(@current_state=a.period_state, @date_anchor, a.start_date) as anchor_date,  
            if(@current_state=a.period_state, @date_anchor, @date_anchor:=a.start_date),  
            @current_state:=a.period_state  
        from status a, (select @current_state:="initial", @date_anchor:="2018-12-31") init  
    ) anchor  
)  
  
select  
    a.period_state, a.start_date, a.end_date  
from (  
    select  
        period_state, start_date,   
        max(start_date) over(partition by anchor_date) as end_date,  
        rank() over(order by start_date) rn  
    from date_anchor  
) a  
group by end_date  
having min(rn)  
order by start_date
```

## 解題思考 (Rank版本)

*   在提交了 variable 版本後，發現連續日期在排序後，其 `rn` 等差為 `1` ，這表示每次 `period_state` 變換時，取第一個日期作為該 `period` 的最小值，則該 `period` 中所有資料都應符合 `start_date-rn=min(start_date)` 的條件，因此又寫了一個 Rank 版本。
*   透過 `with clause` 分別建立 `failed_state` 、 `successeded_state`表格。  
    篩選資料時間介於 `2019–01–01` 至 `2019-12-31` 的資料集，  
    擴增 `period_state` 欄位，  
    利用 `rank() over(order by )` 標註排序結果 `rn` 。
*   做子查詢表格 `opt`，聯合 `failed_state` 和 `successeded_state` 。  
    在 `failed_state` 表格，以 `group by period_state, date_add(fail_date, interval -rn day)` ，分別找出`failed_state` 和 `successeded_state`，每個 period 的 `min(date)` 和 `max(date)` ，作為輸出結果的 `start_date` 和 `end_date` 欄位。

## 解決方案 (Rank版本)
```sql
with  
failed_state as (  
    select  
        'failed' as period_state,  
        fail_date,  
        rank() over(order by fail_date) -1 as rn  
    from Failed  
    where fail_date between '2019-01-01' and '2019-12-31'  
),  
succeeded_state as (  
    select  
        'succeeded' as period_state,  
        success_date,  
        rank() over(order by success_date) -1 as rn  
    from Succeeded  
    where success_date between '2019-01-01' and '2019-12-31'  
)  
  
select   
    period_state, start_date, end_date  
from (  
    select  
        period_state,   
        min(fail_date) as start_date,  
        max(fail_date) as end_date  
    from failed_state  
    group by period_state, date_add(fail_date, interval -rn day)  
    union  
    select  
        period_state,  
        min(success_date) as start_date,  
        max(success_date) as end_date  
    from succeeded_state  
    group by period_state, date_add(success_date, interval -rn day)  
) opt
```
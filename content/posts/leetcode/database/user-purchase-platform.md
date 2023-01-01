---
title: '[leetcode][Database][Hard] 1127. User Purchase Platform'
date: '2022-12-06T13:21:25.529Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Spending`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| user_id     | int     |  
| spend_date  | date    |  
| platform    | enum    |   
| amount      | int     |  
+-------------+---------+  
The table logs the history of the spending of users that make purchases from an online shopping website that has a desktop and a mobile application.  
(user_id, spend_date, platform) is the primary key of this table.  
The platform column is an ENUM type of ('desktop', 'mobile').
```

Write an SQL query to find the total number of users and the total amount spent using the mobile only, the desktop only, and both mobile and desktop together for each date.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Spending (user_id int, spend_date date, platform ENUM('desktop', 'mobile'), amount int)  
Truncate table Spending  
insert into Spending (user_id, spend_date, platform, amount) values ('1', '2019-07-01', 'mobile', '100')  
insert into Spending (user_id, spend_date, platform, amount) values ('1', '2019-07-01', 'desktop', '100')  
insert into Spending (user_id, spend_date, platform, amount) values ('2', '2019-07-01', 'mobile', '100')  
insert into Spending (user_id, spend_date, platform, amount) values ('2', '2019-07-02', 'mobile', '100')  
insert into Spending (user_id, spend_date, platform, amount) values ('3', '2019-07-01', 'desktop', '100')  
insert into Spending (user_id, spend_date, platform, amount) values ('3', '2019-07-02', 'desktop', '100')
```

## 解題思考

*   題目要求統計透過用戶 `user`在使用PC `desktop`、手機APP `mobile` 以及兩者皆有 `both` 的採購金額 `amount` 。  
    分開計算PC `desktop` 和手機APP`mobile` 並不困難，因此需要著手處理的是兩者皆有 `both` 採購紀錄的用戶，並將這些用戶從PC `desktop`和手機APP`mobile` 的統計中分離出來。
*   透過 `with clause` 建立 `p_user_platform` 表格，從 `spending` 表格帶出 `user_id`、 `spend_date`，並透過 `group by user_id, spend_date` 將資料分組為每個用戶 `user_id` 在每個 `spend_date` 的採購資料，並分開加總 `platform="mobile"` 、`platform="desktop"` 的採購數量 `amount`以賦予 `mobild_amount` 和 `desktop_amount` 欄位。
*   透過 `with clause` 建立 `p_user_summary` 表格，判斷 `mobild_amount` 和 `desktop_amount` 的值，將每位用戶在每個 `spend_date` 的採購情形分類成 `desktop`、`mobile`和`both`   
    這個步驟也可以在建立 `p_user_platform` 的過程中進行分類，但我傾向明確每張表格的用途，以便理解每一個 `query statement` 中引用的資料表格與欄位。
*   透過 `with clause` 建立 `p_spend_date` 表格，並在 `spending` 表格擷取所有不重複的 `spend_date` ，並將每個不重複的 `spend_date` 擴展成帶有 `desktop`、`mobile`和`both` 的 record 。  
    這個動作會將每個不重複的 `spend_date` ，從原先的 1 row 擴展成 3 rows。
*   透過查詢 `p_spend_date` 作為主要表格，使用 `left join p_user_summary` 帶出相對應 `spend_date` 、`platform` 的 record set，並利用 `spend_date` 、`platform` 統計出該 `spend_date` 在 `desktop`、`mobile`和`both` 的總採購量 `total_amount` 和總計人數 `total_user` 。

## 解決方案
```sql
with  
p_user_platform as (  
    select   
        user_id, spend_date,  
        sum(case when platform = 'mobile' then amount else 0 end) as mobile_amount,  
        sum(case when platform = 'desktop' then amount else 0 end) as desktop_amount  
    from spending  
    group by user_id, spend_date  
),  
p_user_summary as (  
    select  
        user_id, spend_date,  
        if(mobile_amount > 0, if(desktop_amount > 0, 'both', 'mobile'), 'desktop') as platform,  
        mobile_amount + desktop_amount as amount  
    from p_user_platform  
),  
p_spend_date as (  
    select distinct(spend_date), 'desktop' as platform from spending  
    union  
    select distinct(spend_date), 'mobile' as platform from spending  
    union  
    select distinct(spend_date), 'both' as platform from spending  
)  
  
select  
    a.spend_date, a.platform,  
    sum(ifnull(b.amount,0)) as total_amount,  
    count(b.user_id) as total_users  
from p_spend_date a  
left join p_user_summary b using(spend_date, platform)  
group by a.spend_date, a.platform
```
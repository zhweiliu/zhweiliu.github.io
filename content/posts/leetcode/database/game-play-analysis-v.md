---
title: '[leetcode][Database][Hard] 1097. Game Play Analysis V'
date: '2022-12-06T13:48:34.280Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Activity`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| player_id    | int     |  
| device_id    | int     |  
| event_date   | date    |  
| games_played | int     |  
+--------------+---------+  
(player_id, event_date) is the primary key of this table.  
This table shows the activity of players of some games.  
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on someday using some device.
```
The `install date` of a player is the first login day of that player.

We define `day one retention` of some date `x` to be the number of players whose `install date` is `x` and they logged back in on the day right after `x`, divided by the number of players whose install date is `x`, rounded to `2` decimal places.

Write an SQL query to report for each install date, the number of players that installed the game on that day, and the `day one retention`.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Activity (player_id int, device_id int, event_date date, games_played int)  
Truncate table Activity  
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-01', '5')  
insert into Activity (player_id, device_id, event_date, games_played) values ('1', '2', '2016-03-02', '6')  
insert into Activity (player_id, device_id, event_date, games_played) values ('2', '3', '2017-06-25', '1')  
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '1', '2016-03-01', '0')  
insert into Activity (player_id, device_id, event_date, games_played) values ('3', '4', '2018-07-03', '5')
```
## 解題思考

*   題目要求利用 `Activity` 表格統計安裝用戶數量，以及用戶首日留存率。  
    `定義用戶安裝日` → 用戶在 `Activity` 第一次出現紀錄的 `event_date` 。  
    `定義用戶首日留存` → 用戶在安裝日隔天 `event_date+1` 有 `Activity` record。  
    `定義首日留存率計算方式` → `用戶首日留存數 / 安裝日總用戶人數`
*   透過 `with clause` 建立 `player_install_date` 表格，找出每位用戶的安裝日 `install_dt`
*   透過 `with clause` 建立 `player_day1_back` 表格，統計`用戶首日留存`人數 `cnt_login_back_player` 。  
    選擇 `Activity` 作為主要表格，使用 `left join player_install_date` 帶出所有的安裝日 `install_dt` ，並過濾 Activity 表格中符合條件 `Activity.event_date = player_install_date+1` 的 record set。
*   從 `player_install_date` 帶出 `用戶安裝日` 統計人數，使用 `left join player_day1_back` 帶出`用戶首日留存` 統計人數，便能計算出 `首日留存率` 。

## 解決方案
```sql
with  
player_install_date as (  
    select  
        player_id,  
        min(event_date) as install_dt  
    from activity  
    group by player_id  
),  
player_day1_back as (  
    select distinct  
        b.install_dt,  
        count(a.player_id) over(partition by a.event_date) as cnt_login_back_player  
    from activity a  
    left join player_install_date b using(player_id)  
    where a.event_date = date_add(b.install_dt, INTERVAL 1 DAY)  
)  
  
select   
    a.install_dt,  
    a.installs,  
    round(ifnull(b.cnt_login_back_player,0) / a.installs,2) as Day1_retention  
from (  
    select distinct   
        install_dt,  
        count(player_id) over(partition by install_dt) as installs  
    from player_install_date  
) a  
left join player_day1_back b on b.install_dt = a.install_dt
```
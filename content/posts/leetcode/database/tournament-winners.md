---
title: '[leetcode][Database][Hard] 1194. Tournament Winners'
date: '2022-12-06T09:28:54.784Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Players`
```
+-------------+-------+  
| Column Name | Type  |  
+-------------+-------+  
| player_id   | int   |  
| group_id    | int   |  
+-------------+-------+  
player_id is the primary key of this table.  
Each row of this table indicates the group of each player.
```

Table: `Matches`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| match_id      | int     |  
| first_player  | int     |  
| second_player | int     |   
| first_score   | int     |  
| second_score  | int     |  
+---------------+---------+  
match_id is the primary key of this table.  
Each row is a record of a match, first_player and second_player contain the player_id of each match.  
first_score and second_score contain the number of points of the first_player and second_player respectively.  
You may assume that, in each match, players belong to the same group.
```

The winner in each group is the player who scored the maximum total points within the group. In the case of a tie, the `lowest` `player_id` wins.

Write an SQL query to find the winner in each group.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Players (player_id int, group_id int)  
Create table If Not Exists Matches (match_id int, first_player int, second_player int, first_score int, second_score int)  
  
Truncate table Players  
insert into Players (player_id, group_id) values ('10', '2')  
insert into Players (player_id, group_id) values ('15', '1')  
insert into Players (player_id, group_id) values ('20', '3')  
insert into Players (player_id, group_id) values ('25', '1')  
insert into Players (player_id, group_id) values ('30', '1')  
insert into Players (player_id, group_id) values ('35', '2')  
insert into Players (player_id, group_id) values ('40', '3')  
insert into Players (player_id, group_id) values ('45', '1')  
insert into Players (player_id, group_id) values ('50', '2')  
Truncate table Matches  
insert into Matches (match_id, first_player, second_player, first_score, second_score) values ('1', '15', '45', '3', '0')  
insert into Matches (match_id, first_player, second_player, first_score, second_score) values ('2', '30', '25', '1', '2')  
insert into Matches (match_id, first_player, second_player, first_score, second_score) values ('3', '30', '15', '2', '0')  
insert into Matches (match_id, first_player, second_player, first_score, second_score) values ('4', '40', '20', '5', '2')  
insert into Matches (match_id, first_player, second_player, first_score, second_score) values ('5', '35', '50', '1', '1')
```
## 解題思考

*   樞紐表要求輸出每個 `group` 內總計得分最高的玩家，若遇到平分情況則以 `player_id` 較小的一方獲勝。  
    這題的思考方式和 [[leetcode][Database][Hard]1972. First and Last Call On the Same Day](https://zhengwei-liu.medium.com/leetcode-database-hard-1972-first-and-last-call-on-the-same-day-423e4a8e9c71) 雷同。
*   透過 `with clause` 建立 `max_score_of_player` 表格，並分別加總每位 `player` 的 `score` 。  
    將 `score` 表格中的 `first_player` 和 `second_player` 拆分成兩張子表，並擷取 `first_score` 和 `second_score` 以便進行每位 `player` 的 `score` 加總。
*   透過 `with clause` 建立 `group_player_rank` 表格，選擇 `players` 表格作為查詢主表並關聯 `max_score_of_player` ，並依據 `max_score_of_player.score` 降序和 `players.player_id` 升序的方式，利用 `rank()`函數對`players.group` 進行排名 `rn`。
*   最後，查詢 `group_player_rank.rn` 為 `1` 的資料，便能找出每個 `group` 總計得分最高的 `player` 。

## 解決方案
```sql
with  
max_score_of_player as (  
    select   
        player_id,    
        sum(score) as score  
    from (  
        select first_player as player_id, first_score as score from matches  
        union all  
        select second_player as player_id, second_score as score from matches  
    ) a  
    group by player_id  
),  
group_player_rank as (  
    select   
        a.group_id,  
        a.player_id,  
        rank() over(partition by a.group_id order by b.score desc, a.player_id asc) as rn  
    from players a  
    join max_score_of_player b using(player_id)  
)  
  
select  
    group_id, player_id  
from group_player_rank  
where rn = 1
```
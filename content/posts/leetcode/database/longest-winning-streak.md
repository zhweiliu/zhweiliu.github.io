---
title: '[leetcode][Database][Hard] 2173. Longest Winning Streak'
date: '2022-12-18T21:45:21.994Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## Description

Table: `Matches`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| player_id   | int  |  
| match_day   | date |  
| result      | enum |  
+-------------+------+  
(player_id, match_day) is the primary key for this table.  
Each row of this table contains the ID of a player, the day of the match they played, and the result of that match.  
The result column is an ENUM type of ('Win', 'Draw', 'Lose').
```

The `winning streak` of a player is the number of consecutive wins uninterrupted by draws or losses.

Write an SQL query to count the longest winning streak for each player.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Matches (player_id int, match_day date, result ENUM('Win', 'Draw', 'Lose'))  
Truncate table Matches  
insert into Matches (player_id, match_day, result) values ('1', '2022-01-17', 'Win')  
insert into Matches (player_id, match_day, result) values ('1', '2022-01-18', 'Win')  
insert into Matches (player_id, match_day, result) values ('1', '2022-01-25', 'Win')  
insert into Matches (player_id, match_day, result) values ('1', '2022-01-31', 'Draw')  
insert into Matches (player_id, match_day, result) values ('1', '2022-02-08', 'Win')  
insert into Matches (player_id, match_day, result) values ('2', '2022-02-06', 'Lose')  
insert into Matches (player_id, match_day, result) values ('2', '2022-02-08', 'Lose')  
insert into Matches (player_id, match_day, result) values ('3', '2022-03-30', 'Win')
```
## Idea

The query result format is in the following example.
```
+-----------+----------------+  
| player_id | longest_streak |  
+-----------+----------------+  
| 1         | 3              |  
| 2         | 0              |  
| 3         | 1              |  
+-----------+----------------+
```
Fulfill requirements :

I guess that I have to extract each game result which is not a winner, and convert `_a split timeline_` for each player to help calculating `_longest winning streak_`.

So, I marked each game and sorted by `match_day` of each player, that will help to find `losing` matches, then I can use it as a `split timeline` to calculate with `win` of continuous games.

## Solution
```sql
with  
game_result_rn as (  
    select  
        row_number() over(partition by player_id order by match_day) as rn,  
        player_id, result  
    from Matches  
),  
player_lose_game_rn as (  
    select   
        player_id, rn,  
        ifnull(lead(rn, 1) over(partition by player_id order by rn) ,rn) as next_rn  
    from (  
        select  player_id, 0 as rn from game_result_rn group by player_id  
        union  
        select   
            player_id, rn  
        from game_result_rn  
        where result <> 'Win'  
        union  
        select  player_id, max(rn)+1 as rn from game_result_rn group by player_id  
    ) a  
),  
count_player_win as (  
    select distinct  
        a.player_id,  
        count(result) over(partition by a.player_id, b.next_rn) as longest_streak  
    from game_result_rn a  
    left join player_lose_game_rn b on b.player_id=a.player_id  
    where a.rn between b.rn and b.next_rn and a.result = 'Win'  
)  
  
select  
    a.player_id,   
    ifnull(max(b.longest_streak), 0) as longest_streak  
from (  
    select distinct player_id from Matches  
) a  
left join count_player_win b using(player_id)  
group by a.player_id
```
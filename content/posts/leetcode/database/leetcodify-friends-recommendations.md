---
title: '[leetcode][Database][Hard] 1917. Leetcodify Friends Recommendations'
date: '2022-12-15T23:52:20.337Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## Description

Table: `Listens`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| user_id     | int     |  
| song_id     | int     |  
| day         | date    |  
+-------------+---------+  
There is no primary key for this table. It may contain duplicates.  
Each row of this table indicates that the user user_id listened to the song song_id on the day day.
```

Table: `Friendship`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| user1_id      | int     |  
| user2_id      | int     |  
+---------------+---------+  
(user1_id, user2_id) is the primary key for this table.  
Each row of this table indicates that the users user1_id and user2_id are friends.  
Note that user1_id < user2_id.
```

Write an SQL query to recommend friends to Leetcodify users. We recommend user `x` to user `y` if:

*   Users `x` and `y` are not friends, and
*   Users `x` and `y` listened to the same three or more different songs `on the same day`.

Note that friend recommendations are `unidirectional`, meaning if user `x` and user `y` should be recommended to each other, the result table should have both user `x` recommended to user `y` and user `y` recommended to user `x`. Also, note that the result table should not contain duplicates (i.e., user `y` should not be recommended to user `x` multiple times.).

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Listens (user_id int, song_id int, day date)  
Create table If Not Exists Friendship (user1_id int, user2_id int)  
Truncate table Listens  
insert into Listens (user_id, song_id, day) values ('1', '10', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('1', '11', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('1', '12', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('2', '10', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('2', '11', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('2', '12', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('3', '10', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('3', '11', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('3', '12', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('4', '10', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('4', '11', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('4', '13', '2021-03-15')  
insert into Listens (user_id, song_id, day) values ('5', '10', '2021-03-16')  
insert into Listens (user_id, song_id, day) values ('5', '11', '2021-03-16')  
insert into Listens (user_id, song_id, day) values ('5', '12', '2021-03-16')  
Truncate table Friendship  
insert into Friendship (user1_id, user2_id) values ('1', '2')
```
## Idea

The query result format is in the following example.
```
+---------+----------------+  
| user_id | recommended_id |  
+---------+----------------+  
| 1       | 3              |  
| 2       | 3              |  
| 3       | 1              |  
| 3       | 2              |  
+---------+----------------+
```
Fulfill requirements :

The thinking process like as [[leetcode][Database][Hard] 1892. Page Recommendations II](https://zhengwei-liu.medium.com/leetcode-database-hard-1892-page-recommendations-ii-eb2eb4ab3dc8?source=user_profile---------1----------------------------), so the first step is listing each user and their friends `cte_user_friends` .

To avoid the records that user listen the same song in a day, using the `distinct` to remove duplicates records.

Finding the users listening a song in a day via self join `cte_listen_distinct` , and then confirm the user friendship who in above self join set and remove them.

Finally, counting the `song_id` via group by `day`, `user_id`, `recommended_id`, and filtering the counting `song_id` value large than or equals to `3` after `group by` statement.

## Solution
```sql
with  
cte_user_friends as (  
    select user1_id as user_id, user2_id as friend_id from Friendship  
    union  
    select user2_id as user_id, user1_id as friend_id from Friendship  
),  
cte_listen_distinct as (  
    select distinct   
        user_id, song_id, day  
    from Listens  
)  
  
select distinct  
    a.user_id, b.user_id as recommended_id  
from cte_listen_distinct a  
left join cte_listen_distinct b on b.song_id=a.song_id and a.day=b.day  
left join cte_user_friends c on c.user_id = a.user_id and c.friend_id = b.user_id  
where c.user_id is null and a.user_id <> b.user_id  
group by a.day, a.user_id, b.user_id  
having count(a.song_id) >=3
``` 
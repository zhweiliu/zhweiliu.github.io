---
title: '[leetcode][Database][Hard] 1919. Leetcodify Similar Friends'
date: '2022-12-16T01:08:30.340Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
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

Write an SQL query to report the similar friends of Leetcodify users. A user `x` and user `y` are similar friends if:

*   Users `x` and `y` are friends, and
*   Users `x` and `y` listened to the same three or more different songs `on the same day`.

Return the result table in `any order`. Note that you must return the similar pairs of friends the same way they were represented in the input (i.e., always `user1_id < user2_id`).

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
insert into Friendship (user1_id, user2_id) values ('2', '4')  
insert into Friendship (user1_id, user2_id) values ('2', '5')
```
## Idea

The query result format is in the following example.
```
+----------+----------+  
| user1_id | user2_id |  
+----------+----------+  
| 1        | 2        |  
+----------+----------+
```
Fulfill requirements :

The thinking process like as [[leetcode][Database][Hard] 1917. Leetcodify Friends Recommendations](https://zhengwei-liu.medium.com/leetcode-database-hard-1917-leetcodify-friends-recommendations-d656d741284a?source=user_profile---------1----------------------------), the different is this question asking to find similar friend (i.e. user `x` and user `y` have a pair record in `Friendship`).

So, we can do that with the same logic to find user `x` and user `y` are listened to the same three or more different songs `on the same day or not`.

1.  Listing each user and their friends `cte_user_friends` .
2.  To avoid the records that user listen the same song in a day, using the `distinct` to remove duplicates records.

Finally, counting the `song_id` via group by `day`, `user_id`, `recommended_id`, and filtering the counting `song_id` value large than or equals to `3` after `group by` statement.

## Solution
```sql
with  
cte_user_friends as (  
    select user1_id as user1_id, user2_id as user2_id from Friendship  
    union  
    select user2_id as user1_id, user1_id as user2_id from Friendship  
),  
cte_listen_distinct as (  
    select distinct   
        user_id, song_id , day  
    from Listens  
),  
cte_similar_friend as (  
    select  
        a.user_id as user1_id, b.user_id as user2_id  
    from cte_listen_distinct a # user1  
    left join cte_listen_distinct b on b.song_id=a.song_id and a.day=b.day # user2  
    left join cte_user_friends c on c.user1_id = a.user_id and c.user2_id = b.user_id  
    where a.user_id <> b.user_id and user1_id is not null  
    group by a.day, a.user_id, b.user_id  
    having count(a.song_id) >=3  
)  
  
select distinct  
    b.user1_id, b.user2_id  
from Friendship a  
join cte_similar_friend b on b.user1_id = a.user1_id and b.user2_id = a.user2_id
```
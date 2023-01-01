---
title: '[leetcode][Database][Hard] 1892. Page Recommendations II'
date: '2022-12-15T21:38:41.738Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

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
```

Table: `Likes`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| user_id     | int     |  
| page_id     | int     |  
+-------------+---------+  
(user_id, page_id) is the primary key for this table.  
Each row of this table indicates that user_id likes page_id.
```

You are implementing a page recommendation system for a social media website. Your system will `recommended` a page to `user_id` if the page is `liked` by `at least one` friend of `user_id` and is `not liked` by `user_id`.

Write an SQL query to find all the possible `page recommendations` for every user. Each recommendation should appear as a row in the result table with these columns:

*   `user_id`: The ID of the user that your system is making the recommendation to.
*   `page_id`: The ID of the page that will be recommended to `user_id`.
*   `friends_likes`: The number of the friends of `user_id` that like `page_id`.

Return result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Friendship (user1_id int, user2_id int)  
Create table If Not Exists Likes (user_id int, page_id int)  
Truncate table Friendship  
insert into Friendship (user1_id, user2_id) values ('1', '2')  
insert into Friendship (user1_id, user2_id) values ('1', '3')  
insert into Friendship (user1_id, user2_id) values ('1', '4')  
insert into Friendship (user1_id, user2_id) values ('2', '3')  
insert into Friendship (user1_id, user2_id) values ('2', '4')  
insert into Friendship (user1_id, user2_id) values ('2', '5')  
insert into Friendship (user1_id, user2_id) values ('6', '1')  
Truncate table Likes  
insert into Likes (user_id, page_id) values ('1', '88')  
insert into Likes (user_id, page_id) values ('2', '23')  
insert into Likes (user_id, page_id) values ('3', '24')  
insert into Likes (user_id, page_id) values ('4', '56')  
insert into Likes (user_id, page_id) values ('5', '11')  
insert into Likes (user_id, page_id) values ('6', '33')  
insert into Likes (user_id, page_id) values ('2', '77')  
insert into Likes (user_id, page_id) values ('3', '77')  
insert into Likes (user_id, page_id) values ('6', '88')
```
## Idea

The query result format is in the following example.
```
+---------+---------+---------------+  
| user_id | page_id | friends_likes |  
+---------+---------+---------------+  
| 1       | 77      | 2             |  
| 1       | 23      | 1             |  
| 1       | 24      | 1             |  
| 1       | 56      | 1             |  
| 1       | 33      | 1             |  
| 2       | 24      | 1             |  
| 2       | 56      | 1             |  
| 2       | 11      | 1             |  
| 2       | 88      | 1             |  
| 3       | 88      | 1             |  
| 3       | 23      | 1             |  
| 4       | 88      | 1             |  
| 4       | 77      | 1             |  
| 4       | 23      | 1             |  
| 5       | 77      | 1             |  
| 5       | 23      | 1             |  
+---------+---------+---------------+
```
Fulfill requirements :

The thinking process like as [[leetcode][Database][Hard]1972. First and Last Call On the Same Day](https://zhengwei-liu.medium.com/leetcode-database-hard-1972-first-and-last-call-on-the-same-day-423e4a8e9c71), so the first step is listing each user and their friends `cte_all_users`.

Finding `the pages which are friends likes`, then to find which `pages both user and friends likes`. Finally, filtering not match the condition : `the page user not liked but friends did`.

This solution have same concept with `minus` or `except`, finding the difference set between both user and friends likes pages and only friends like pages.

## Solution
```sql
with  
cte_all_users as (  
    select user1_id as user_id, user2_id as friend from Friendship  
    union  
    select user2_id as user_id, user1_id as friend from Friendship  
)  
  
select distinct  
    a.user_id, b.page_id , count(a.friend) as friends_likes  
from cte_all_users a  
join Likes b on b.user_id = a.friend -- find the pages friend like  
left join Likes c on c.user_id = a.user_id and b.page_id = c.page_id -- find the pages both user and friend like  
where c.page_id is null -- filtering not match the condition : the page user not liked but friends did.   
group by a.user_id, b.page_id
```
---
title: '[leetcode][Database][Hard] 1369. Get the Second Most Recent Activity'
date: '2022-12-13T00:10:04.002Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## Description

Table: `UserActivity`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| username      | varchar |  
| activity      | varchar |  
| startDate     | Date    |  
| endDate       | Date    |  
+---------------+---------+  
There is no primary key for this table. It may contain duplicates.  
This table contains information about the activity performed by each user in a period of time.  
A person with username performed an activity from startDate to endDate.
```
Write an SQL query to show the `second most recent activity` of each user.

If the user only has one activity, return that one. A user cannot perform more than one activity at the same time.

Return the result table in `any` order.

SQL Schema
```sql
Create table If Not Exists UserActivity (username varchar(30), activity varchar(30), startDate date, endDate date)  
Truncate table UserActivity  
insert into UserActivity (username, activity, startDate, endDate) values ('Alice', 'Travel', '2020-02-12', '2020-02-20')  
insert into UserActivity (username, activity, startDate, endDate) values ('Alice', 'Dancing', '2020-02-21', '2020-02-23')  
insert into UserActivity (username, activity, startDate, endDate) values ('Alice', 'Travel', '2020-02-24', '2020-02-28')  
insert into UserActivity (username, activity, startDate, endDate) values ('Bob', 'Travel', '2020-02-11', '2020-02-18')
```
## Idea

The query result format is in the following example.
```
+------------+--------------+-------------+-------------+  
| username   | activity     | startDate   | endDate     |  
+------------+--------------+-------------+-------------+  
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |  
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |  
+------------+--------------+-------------+-------------+
```
Fulfill requirements :

1.  To generate a rank table order by `endDate` of each user via function `rank()`as `rn`. It will help to find user who has at least twice activity records in table `UserActivity`.  
    Name this `with clause` as `rank_end_date`.
2.  Find `_second mostly recent activity_` record of each user from `rand_end_date`. User will be missing who has only one activity record.  
    Name this `with clause` as `activity_twice`.
3.  Find users who has only once activity record of table `UserActivity`. Using left join `activity_twice` to rule out users who has been listing in `twice_record`.  
    Name this `with clause` as `activity_once` .
4.  Finally, `union` the result of `activity_twice` and `twice_record` for output of the query.

## Solution
```sql
with  
rank_end_date as (  
    select  
        rank() over(partition by username order by endDate desc) as rn,  
        username,  
        activity,  
        startDate,  
        endDate  
    from useractivity  
),  
activity_twice as (  
    select  
        rn, username, activity, startDate, endDate  
    from rank_end_date  
    where rn = 2  
),  
activity_once as (  
    select  
        a.rn, a.username, a.activity, a.startDate, a.endDate  
    from rank_end_date a  
    left join activity_twice b using(username)  
    where ifnull(b.rn, 0) = 0  
)  
  
select username, activity, startDate, endDate from activity_once  
union  
select username, activity, startDate, endDate from activity_twice
```
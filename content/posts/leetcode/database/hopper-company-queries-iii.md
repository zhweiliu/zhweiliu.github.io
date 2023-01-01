---
title: '[leetcode][Database][Hard] 1651. Hopper Company Queries III'
date: '2022-12-15T16:48:56.352Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Drivers`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| driver_id   | int     |  
| join_date   | date    |  
+-------------+---------+  
driver_id is the primary key for this table.  
Each row of this table contains the driver's ID and the date they joined the Hopper company.
```

Table: `Rides`
```
+--------------+---------+  
| Column Name  | Type    |  
+--------------+---------+  
| ride_id      | int     |  
| user_id      | int     |  
| requested_at | date    |  
+--------------+---------+  
ride_id is the primary key for this table.  
Each row of this table contains the ID of a ride, the user's ID that requested it, and the day they requested it.  
There may be some ride requests in this table that were not accepted.
```

Table: `AcceptedRides`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| ride_id       | int     |  
| driver_id     | int     |  
| ride_distance | int     |  
| ride_duration | int     |  
+---------------+---------+  
ride_id is the primary key for this table.  
Each row of this table contains some information about an accepted ride.  
It is guaranteed that each accepted ride exists in the Rides table.
```

Write an SQL query to compute the `average_ride_distance` and `average_ride_duration` of every 3-month window starting from `January - March 2020` to `October - December 2020`. Round `average_ride_distance` and `average_ride_duration` to the nearest `two decimal places`.

The `average_ride_distance` is calculated by summing up the total `ride_distance` values from the three months and dividing it by `3`. The `average_ride_duration` is calculated in a similar way.

Return the result table ordered by `month` in ascending order, where `month` is the starting month's number (January is `1`, February is `2`, etc.).

SQL Schema
```sql
Create table If Not Exists Drivers (driver_id int, join_date date)  
Create table If Not Exists Rides (ride_id int, user_id int, requested_at date)  
Create table If Not Exists AcceptedRides (ride_id int, driver_id int, ride_distance int, ride_duration int)  
Truncate table Drivers  
insert into Drivers (driver_id, join_date) values ('10', '2019-12-10')  
insert into Drivers (driver_id, join_date) values ('8', '2020-1-13')  
insert into Drivers (driver_id, join_date) values ('5', '2020-2-16')  
insert into Drivers (driver_id, join_date) values ('7', '2020-3-8')  
insert into Drivers (driver_id, join_date) values ('4', '2020-5-17')  
insert into Drivers (driver_id, join_date) values ('1', '2020-10-24')  
insert into Drivers (driver_id, join_date) values ('6', '2021-1-5')  
Truncate table Rides  
insert into Rides (ride_id, user_id, requested_at) values ('6', '75', '2019-12-9')  
insert into Rides (ride_id, user_id, requested_at) values ('1', '54', '2020-2-9')  
insert into Rides (ride_id, user_id, requested_at) values ('10', '63', '2020-3-4')  
insert into Rides (ride_id, user_id, requested_at) values ('19', '39', '2020-4-6')  
insert into Rides (ride_id, user_id, requested_at) values ('3', '41', '2020-6-3')  
insert into Rides (ride_id, user_id, requested_at) values ('13', '52', '2020-6-22')  
insert into Rides (ride_id, user_id, requested_at) values ('7', '69', '2020-7-16')  
insert into Rides (ride_id, user_id, requested_at) values ('17', '70', '2020-8-25')  
insert into Rides (ride_id, user_id, requested_at) values ('20', '81', '2020-11-2')  
insert into Rides (ride_id, user_id, requested_at) values ('5', '57', '2020-11-9')  
insert into Rides (ride_id, user_id, requested_at) values ('2', '42', '2020-12-9')  
insert into Rides (ride_id, user_id, requested_at) values ('11', '68', '2021-1-11')  
insert into Rides (ride_id, user_id, requested_at) values ('15', '32', '2021-1-17')  
insert into Rides (ride_id, user_id, requested_at) values ('12', '11', '2021-1-19')  
insert into Rides (ride_id, user_id, requested_at) values ('14', '18', '2021-1-27')  
Truncate table AcceptedRides  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('10', '10', '63', '38')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('13', '10', '73', '96')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('7', '8', '100', '28')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('17', '7', '119', '68')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('20', '1', '121', '92')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('5', '7', '42', '101')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('2', '4', '6', '38')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('11', '8', '37', '43')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('15', '8', '108', '82')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('12', '8', '38', '34')  
insert into AcceptedRides (ride_id, driver_id, ride_distance, ride_duration) values ('14', '1', '90', '74')
```
## Idea

The query result format is in the following example.
```
+-------+-----------------------+-----------------------+  
| month | average_ride_distance | average_ride_duration |  
+-------+-----------------------+-----------------------+  
| 1     | 21.00                 | 12.67                 |  
| 2     | 21.00                 | 12.67                 |  
| 3     | 21.00                 | 12.67                 |  
| 4     | 24.33                 | 32.00                 |  
| 5     | 57.67                 | 41.33                 |  
| 6     | 97.33                 | 64.00                 |  
| 7     | 73.00                 | 32.00                 |  
| 8     | 39.67                 | 22.67                 |  
| 9     | 54.33                 | 64.33                 |  
| 10    | 56.33                 | 77.00                 |  
+-------+-----------------------+-----------------------+
```
Fulfill requirements :

Before all, I would like to share a problem for `timeout limit exceeded` of this question when I submit my solution. Because of the pervious solution, I used `subquery` in `select column statement` to calculate average distance and average duration.

So, I’m learned when the query statement meet a large dataset, the subquery will spending more process resource to calculate its.

For the above reason, I modifying the output query used function `lead()` getting next two months distance and duration for each row instead of `subquery`. From the response of submissions, it’s worked and improve the query performance.

## Solution
```sql
with  
cte_month as (  
    select 1 as month union select 2 as month union  
    select 3 as month union select 4 as month union  
    select 5 as month union select 6 as month union  
    select 7 as month union select 8 as month union  
    select 9 as month union select 10 as month union  
    select 11 as month union select 12 as month  
),  
cte_accepted_rides as (  
    select   
        c.month,  
        sum(c.distance) as distance,  
        sum(c.duration) as duration  
    from (  
        select  
            month(b.requested_at) as month,  
            a.ride_distance as distance,  
            a.ride_duration as duration  
        from AcceptedRides a  
        join Rides b using(ride_id)  
        where year(b.requested_at) = 2020  
    ) c  
    group by c.month  
      
),  
cte_ride_info as (  
    select  
        a.month,  
        ifnull(b.distance, 0) as m0_distance,  
        ifnull(b.duration, 0) as m0_duration,  
        ifnull(lead(b.distance, 1) over(order by a.month), 0) as m1_distance,  
        ifnull(lead(b.duration, 1) over(order by a.month), 0) as m1_duration,  
        ifnull(lead(b.distance, 2) over(order by a.month), 0) as m2_distance,  
        ifnull(lead(b.duration, 2) over(order by a.month), 0) as m2_duration  
    from cte_month a  
    left join cte_accepted_rides b on b.month = a.month  
)  
  
  
select  
    distinct(a.month),  
    round((m0_distance + m1_distance + m2_distance ) / 3, 2) as average_ride_distance,  
    round((m0_duration  + m1_duration  + m2_duration  ) / 3, 2) as average_ride_duration  
from cte_ride_info a  
where a.month <= 10
```
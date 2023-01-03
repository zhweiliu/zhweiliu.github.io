---
title: '[leetcode][Database][Hard] 1635. Hopper Company Queries I'
date: '2022-12-14T08:28:10.032Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
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

Write an SQL query to report the following statistics for each month of `2020`:

*   The number of drivers currently with the Hopper company by the end of the month (`active_drivers`).
*   The number of accepted rides in that month (`accepted_rides`).

Return the result table ordered by `month` in ascending order, where `month` is the month's number (January is `1`, February is `2`, etc.).

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
+-------+----------------+----------------+  
| month | active_drivers | accepted_rides |  
+-------+----------------+----------------+  
| 1     | 2              | 0              |  
| 2     | 3              | 0              |  
| 3     | 4              | 1              |  
| 4     | 4              | 0              |  
| 5     | 5              | 0              |  
| 6     | 5              | 1              |  
| 7     | 5              | 1              |  
| 8     | 5              | 1              |  
| 9     | 5              | 0              |  
| 10    | 6              | 0              |  
| 11    | 6              | 2              |  
| 12    | 6              | 1              |  
+-------+----------------+----------------+
```
Fulfill requirements:

The output, it apparently `show the computed result` of `each month`. So, we can generate a monthly table that month range between 1 (Jan) to 12 (Dec) as a main table in the query statement, then `cumulating the active drivers` and `counting the accepted rides` of each month by `left join`.

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
cte_drivers as (  
    select   
        driver_id, join_date,  
        if(year(join_date)<2020, 2020, year(join_date)) as join_year,  
        if(year(join_date)<2020, 1, month(join_date)) as join_month  
    from Drivers  
    where year(join_date) <= 2020  
),  
cte_accepted_rides as (  
    select  
        distinct(month(b.requested_at)) as month,  
        count(ride_id) over(partition by month(b.requested_at)) as accepted_rides   
    from AcceptedRides a  
    join Rides b using(ride_id)  
    where year(b.requested_at)=2020  
)  
  
select  
    distinct(a.month) as month,   
    count(b.driver_id) over(order by a.month) as active_drivers,  
    ifnull(c.accepted_rides, 0) as accepted_rides  
from cte_month a  
left join cte_drivers b on b.join_month = a.month  
left join cte_accepted_rides c on c.month = a.month
```
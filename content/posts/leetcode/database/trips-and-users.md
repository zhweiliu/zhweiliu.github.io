---
title: '[leetcode][Database][Hard] 262. Trips and Users'
date: '2022-12-06T17:08:27.973Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Trips`
```
+-------------+----------+  
| Column Name | Type     |  
+-------------+----------+  
| id          | int      |  
| client_id   | int      |  
| driver_id   | int      |  
| city_id     | int      |  
| status      | enum     |  
| request_at  | date     |       
+-------------+----------+  
id is the primary key for this table.  
The table holds all taxi trips. Each trip has a unique id, while client_id and driver_id are foreign keys to the users_id at the Users table.  
Status is an ENUM type of ('completed', 'cancelled_by_driver', 'cancelled_by_client').
```

Table: `Users`
```
+-------------+----------+  
| Column Name | Type     |  
+-------------+----------+  
| users_id    | int      |  
| banned      | enum     |  
| role        | enum     |  
+-------------+----------+  
users_id is the primary key for this table.  
The table holds all users. Each user has a unique users_id, and role is an ENUM type of ('client', 'driver', 'partner').  
banned is an ENUM type of ('Yes', 'No').
```

The `cancellation rate` is computed by dividing the number of canceled (by client or driver) requests with unbanned users by the total number of requests with unbanned users on that day.

Write a SQL query to find the `cancellation rate` of requests with unbanned users (`both client and driver must not be banned`) each day between `"2013-10-01"` and `"2013-10-03"`. Round `Cancellation Rate` to `two decimal` points.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Trips (id int, client_id int, driver_id int, city_id int, status ENUM('completed', 'cancelled_by_driver', 'cancelled_by_client'), request_at varchar(50))  
Create table If Not Exists Users (users_id int, banned varchar(50), role ENUM('client', 'driver', 'partner'))  
Truncate table Trips  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('1', '1', '10', '1', 'completed', '2013-10-01')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('2', '2', '11', '1', 'cancelled_by_driver', '2013-10-01')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('3', '3', '12', '6', 'completed', '2013-10-01')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('4', '4', '13', '6', 'cancelled_by_client', '2013-10-01')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('5', '1', '10', '1', 'completed', '2013-10-02')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('6', '2', '11', '6', 'completed', '2013-10-02')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('7', '3', '12', '6', 'completed', '2013-10-02')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('8', '2', '12', '12', 'completed', '2013-10-03')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('9', '3', '10', '12', 'completed', '2013-10-03')  
insert into Trips (id, client_id, driver_id, city_id, status, request_at) values ('10', '4', '13', '12', 'cancelled_by_driver', '2013-10-03')  
Truncate table Users  
insert into Users (users_id, banned, role) values ('1', 'No', 'client')  
insert into Users (users_id, banned, role) values ('2', 'Yes', 'client')  
insert into Users (users_id, banned, role) values ('3', 'No', 'client')  
insert into Users (users_id, banned, role) values ('4', 'No', 'client')  
insert into Users (users_id, banned, role) values ('10', 'No', 'driver')  
insert into Users (users_id, banned, role) values ('11', 'No', 'driver')  
insert into Users (users_id, banned, role) values ('12', 'No', 'driver')  
insert into Users (users_id, banned, role) values ('13', 'No', 'driver')
```
## 解題思考

*   題目要求輸出 `2013-10-01` 至 `2013-10-03` ，每日的`搭乘請求取消率`，且被納入計算的`有效搭乘請求`必須是`未停權乘客` 和 `未停權駕駛`，`若有一方被停權`，則該筆搭乘請求視為`無效資料`。  
    `搭乘請求取消率` ← `該日有效搭乘請求且該請求被拒絕的資料數` / `該日總有效搭乘請求`
```
+------------+-------------------+  
| Day        | Cancellation Rate |  
+------------+-------------------+  
| 2013-10-01 | 0.33              |  
| 2013-10-02 | 0.00              |  
| 2013-10-03 | 0.50              |  
+------------+-------------------+
```
*   透過 `with clause` 分別建立未停權乘客 `legal_client`、未停權駕駛 `legal_driver`，以及`2013-10-01` 至 `2013-10-03` 的搭乘資料 `legal_trips`。
*   查詢 `legal_trips` 作為主要表格，關聯`legal_client` 和 `legal_driver` ，以計算每日的 `搭乘請求取消率` 。

## 解決方案
```sql
with   
legal_client as ( select * from Users where role = 'client' and banned = 'No'),  
legal_driver as ( select * from Users where role = 'driver' and banned = 'No'),  
legal_trips as (select * from Trips where request_at between '2013-10-01' and '2013-10-03' )  
  
select   
    legal_trips.request_at as Day,  
    round(        count(if(legal_trips.status='completed', NULL, legal_trips.id)) / count(legal_trips.id),   
            2  
        ) as 'Cancellation Rate'  
from legal_trips  
join legal_client on legal_client.users_id = legal_trips.client_id  
join legal_driver on legal_driver.users_id = legal_trips.driver_id  
group by legal_trips.request_at
```
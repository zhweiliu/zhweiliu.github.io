---
title: '[leetcode][Database][Hard] 2153. The Number of Passengers in Each Bus II'
date: '2022-12-05T08:34:19.919Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Buses`
```
+--------------+------+  
| Column Name  | Type |  
+--------------+------+  
| bus_id       | int  |  
| arrival_time | int  |  
| capacity     | int  |  
+--------------+------+  
bus_id is the primary key column for this table.  
Each row of this table contains information about the arrival time of a bus at the LeetCode station and its capacity (the number of empty seats it has).  
No two buses will arrive at the same time and all bus capacities will be positive integers.
```

Table: `Passengers`
```
+--------------+------+  
| Column Name  | Type |  
+--------------+------+  
| passenger_id | int  |  
| arrival_time | int  |  
+--------------+------+  
passenger_id is the primary key column for this table.  
Each row of this table contains information about the arrival time of a passenger at the LeetCode station.
```

Buses and passengers arrive at the LeetCode station. If a bus arrives at the station at a time `tbus` and a passenger arrived at a time `tpassenger` where `tpassenger <= tbus` and the passenger did not catch any bus, the passenger will use that bus. In addition, each bus has a capacity. If at the moment the bus arrives at the station there are more passengers waiting than its capacity `capacity`, only `capacity` passengers will use the bus.

Write an SQL query to report the number of users that used each bus.

Return the result table ordered by `bus_id` in `ascending order`.

SQL Schema
```sql
Create table If Not Exists Buses (bus_id int, arrival_time int, capacity int)  
Create table If Not Exists Passengers (passenger_id int, arrival_time int)  
Truncate table Buses  
insert into Buses (bus_id, arrival_time, capacity) values ('1', '2', '1')  
insert into Buses (bus_id, arrival_time, capacity) values ('2', '4', '10')  
insert into Buses (bus_id, arrival_time, capacity) values ('3', '7', '2')  
Truncate table Passengers  
insert into Passengers (passenger_id, arrival_time) values ('11', '1')  
insert into Passengers (passenger_id, arrival_time) values ('12', '1')  
insert into Passengers (passenger_id, arrival_time) values ('13', '5')  
insert into Passengers (passenger_id, arrival_time) values ('14', '6')  
insert into Passengers (passenger_id, arrival_time) values ('15', '7')
```

## 解題思考

*   乘客 `passenger` 的抵達時間需要在公車 `bus` 抵達之前，計算每班公車 `bus_id` 抵達時，可能潛在的乘客 `passenger_id` 總數量。  
    由於公車 `bus_id` 的運載能力 `capacity` 可能無法滿足當前等待中的所有乘客 `passenger_id` ， 因此先行計算每班公車可能需要負荷的乘客運載量
*   承上，比較當前公車 `bus_id`運載能力 `capacity` 與等待搭乘的乘客數量，並從兩者中取最小值，表達搭乘該班次公車 `bus_id` 的實際乘客數量
*   `可能存在的乘客數 - 累積的乘客數 = 實際搭乘的乘客數量`
*   建立暫存表初始化 `mysql variable` ，用以 `暫存實際搭乘的乘客數量` 和 `可能存在的乘客數`

## 解決方案
```sql
with  
people_possiable_take_bus as (  
    /*  
        Finding the possible passengers with each bus regardless of passenger catch the bus or not,  
        that will cumulate all of passengers whoes previous arrival  
    */  
    select  
        a.bus_id, a.arrival_time, a.capacity, count(b.passenger_id) as possible_passenger_cnt  
    from buses a  
    left join passengers b on b.arrival_time <= a.arrival_time  
    group by a.bus_id  
    order by a.arrival_time  
),  
alloc_people_take_bus as (  
    /*  
        Calculating how many passengers can loaded by each bus,   
        and accumulate passengers regardless of passenger catch bus or not,   
        due to caulse `people_possiable_take_bus` pre-calculate the possible passengers with each bus,   
        and the situation for the passengers whoes cannot catch currently or pervious bus   
        includes of the statement `least(capacity, possible_passenger_cnt-@accum_passengers)`,   
        that's why we can directly to calculate `possible_passenger_cnt-@accum_passengers`    
        and compare bus's capacity and `possible_passenger_cnt-@accum_passengers` to take the least value  
    */  
    select  
        bus_id, passengers_cnt  
    from (  
        select  
            a.bus_id,  
            @passengers_cnt := least(capacity, possible_passenger_cnt-@accum_passengers) as passengers_cnt,  
            @accum_passengers := @accum_passengers + @passengers_cnt  
        from people_possiable_take_bus a, (select @passengers_cnt := 0, @accum_passengers :=0) b  
    ) output  
)  
  
select   
    bus_id, passengers_cnt   
from alloc_people_take_bus  
order by bus_id
```
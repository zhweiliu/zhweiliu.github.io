---
title: '[leetcode][Database][Hard] 1159. Market Analysis II'
date: '2022-12-06T12:27:16.979Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Users`
```
+----------------+---------+  
| Column Name    | Type    |  
+----------------+---------+  
| user_id        | int     |  
| join_date      | date    |  
| favorite_brand | varchar |  
+----------------+---------+  
user_id is the primary key of this table.  
This table has the info of the users of an online shopping website where users can sell and buy items.
```

Table: `Orders`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| order_id      | int     |  
| order_date    | date    |  
| item_id       | int     |  
| buyer_id      | int     |  
| seller_id     | int     |  
+---------------+---------+  
order_id is the primary key of this table.  
item_id is a foreign key to the Items table.  
buyer_id and seller_id are foreign keys to the Users table.
```

Table: `Items`
```
+---------------+---------+  
| Column Name   | Type    |  
+---------------+---------+  
| item_id       | int     |  
| item_brand    | varchar |  
+---------------+---------+  
item_id is the primary key of this table.
```

Write an SQL query to find for each user whether the brand of the second item (by date) they sold is their favorite brand. If a user sold less than two items, report the answer for that user as no. It is guaranteed that no seller sold more than one item on a day.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Users (user_id int, join_date date, favorite_brand varchar(10))  
Create table If Not Exists Orders (order_id int, order_date date, item_id int, buyer_id int, seller_id int)  
Create table If Not Exists Items (item_id int, item_brand varchar(10))  
Truncate table Users  
insert into Users (user_id, join_date, favorite_brand) values ('1', '2019-01-01', 'Lenovo')  
insert into Users (user_id, join_date, favorite_brand) values ('2', '2019-02-09', 'Samsung')  
insert into Users (user_id, join_date, favorite_brand) values ('3', '2019-01-19', 'LG')  
insert into Users (user_id, join_date, favorite_brand) values ('4', '2019-05-21', 'HP')  
Truncate table Orders  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('1', '2019-08-01', '4', '1', '2')  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('2', '2019-08-02', '2', '1', '3')  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('3', '2019-08-03', '3', '2', '3')  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('4', '2019-08-04', '1', '4', '2')  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('5', '2019-08-04', '1', '3', '4')  
insert into Orders (order_id, order_date, item_id, buyer_id, seller_id) values ('6', '2019-08-05', '2', '2', '4')  
Truncate table Items  
insert into Items (item_id, item_brand) values ('1', 'Samsung')  
insert into Items (item_id, item_brand) values ('2', 'Lenovo')  
insert into Items (item_id, item_brand) values ('3', 'LG')  
insert into Items (item_id, item_brand) values ('4', 'HP')
```
## 解題思考

*   題目要求判斷每位 `seller` 賣出的第二項 `item` 是否為該 seller 的喜愛品牌，並輸出 `No` 和 `Yes` 作為每位 `seller` 的分類結果。若某位 `seller` 只賣出一項 `item`，則結果應為 `No`。
*   透過 `with clause` 建立 `order_info` 表格，利用 `rank()`函式並依據賣出日期 `orders.order_date` 為每一筆銷售資料進行排序 `rank_sell_item`。  
    同時，為 `order_info` 表格關聯 `users`表格以帶出每位 `user`的喜愛品牌 `users.favorite_brand`。
*   透過 with clause 建立 `second_sell_brand` 表格，並判斷 `second_sell_brand.sell_brand = second_sell_brand.seller_fav_brand` 以輸出 `Yes` 和 `No` 。
*   透過查詢 `users` 作為主要表格，使用 `left join second_sell_brand` 帶出每位 `seller` 賣出第二項 `item` 是否為自己的喜愛品牌結果。  
    在建立 `second_sell_brand` 表格時，由於篩選條件 `rank_sell_item=2` 會過濾掉只賣出過一次的 `user` 資訊；而在 `orders` 表格中也存在某些 user 沒有賣出的紀錄。  
    因此需要透過 `left join second_sell_brand` 保證每位 user 都包含於最終的輸出結果中。

## 解決方案
```sql
with  
order_info as (  
    select  
        a.order_id, a.order_date, c.item_brand as sell_brand,  
        a.seller_id, b.favorite_brand as seller_fav_brand,  
        rank() over(partition by a.seller_id order by a.order_date) as rank_sell_item  
    from orders a  
    join users b on b.user_id = a.seller_id  
    join items c on c.item_id = a.item_id  
    order by a.order_id, a.order_date  
),  
second_sell_brand as (  
    select   
        seller_id,   
        if(sell_brand=seller_fav_brand, "yes", "no") as 2nd_item_fav_brand   
    from order_info  
    where rank_sell_item = 2  
)  
  
select  
    a.user_id as seller_id,  
    ifnull(b.2nd_item_fav_brand,"no") as 2nd_item_fav_brand  
from users a  
left join second_sell_brand b on b.seller_id = a.user_id
```
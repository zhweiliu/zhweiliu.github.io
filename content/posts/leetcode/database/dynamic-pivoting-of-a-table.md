---
title: '[leetcode][Database][Hard] 2252. Dynamic Pivoting of a Table'
date: '2022-12-05T08:12:40.016Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## 題目

Table: `Products`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| product_id  | int     |  
| store       | varchar |  
| price       | int     |  
+-------------+---------+  
(product_id, store) is the primary key for this table.  
Each row of this table indicates the price of product_id in store.  
There will be at most 30 different stores in the table.  
price is the price of the product at this store.
```

`Important note:` This problem targets those who have a good experience with SQL. If you are a beginner, we recommend that you skip it for now.

Implement the procedure `PivotProducts` to reorganize the `Products` table so that each row has the id of one product and its price in each store. The price should be `null` if the product is not sold in a store. The columns of the table should contain each store and they should be sorted in `lexicographical order`.

The procedure should return the table after reorganizing it.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Products (product_id int, store varchar(7), price int)  
Truncate table Products  
insert into Products (product_id, store, price) values ('1', 'Shop', '110')  
insert into Products (product_id, store, price) values ('1', 'LC_Store', '100')  
insert into Products (product_id, store, price) values ('2', 'Nozama', '200')  
insert into Products (product_id, store, price) values ('2', 'Souq', '190')  
insert into Products (product_id, store, price) values ('3', 'Shop', '1000')  
insert into Products (product_id, store, price) values ('3', 'Souq', '1900')
```
## 解題思考

*   題目要求輸出每個產品 `product` 在每間商店的售價
```
+------------+----------+--------+------+------+  
| product_id | LC_Store | Nozama | Shop | Souq |  
+------------+----------+--------+------+------+  
| 1          | 100      | null   | 110  | null |  
| 2          | null     | 200    | null | 190  |  
| 3          | null     | null   | 1000 | 1900 |  
+------------+----------+--------+------+------+
```
*   使用 `group_concat()` 組合樞紐表欄位的 sql statement，因樞紐表需依據 `store` 列出欄位，而 `store` 的數目是不固定的。
*   使用 `prepare statement` 執行包含 `group_concat()` 預先組合好的 sql statement
*   使用 `group product_id` 對樞紐表進行總計，因樞紐表要求依據 `product` 統計在不同 `store` 中的售價 `price`

## 解決方案
```sql
CREATE PROCEDURE PivotProducts()  
BEGIN  
    -- Override GROUP_CONCAT length which has a default limit of 1024  
    SET SESSION group_concat_max_len = 1000000;  
  
    -- Store case statement for dynamically generated columns in a variable ie case_stmt  
    SET @case_stmt = NULL;  
    SELECT GROUP_CONCAT(DISTINCT CONCAT('SUM(CASE WHEN store = "', store, '" THEN price END) AS ', store))  
    INTO @case_stmt  
    FROM products;  
  
    -- Insert above statement (@case_stmt) in the following main query to frame final query   
    SET @sql_query = CONCAT('SELECT product_id, ', @case_stmt, ' FROM products GROUP BY product_id');  
  
    -- Execute final query  
    PREPARE final_sql_query FROM @sql_query;  
    EXECUTE final_sql_query;  
    DEALLOCATE PREPARE final_sql_query;  
END
```
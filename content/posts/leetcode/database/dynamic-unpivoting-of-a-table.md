---
title: '[leetcode][Database][Hard]2253. Dynamic Unpivoting of a Table'
date: '2022-12-16T21:13:39.338Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## Description

Table: `Products`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| product_id  | int     |  
| store_name1 | int     |  
| store_name2 | int     |  
|      :      | int     |  
|      :      | int     |  
|      :      | int     |  
| store_namen | int     |  
+-------------+---------+  
product_id is the primary key for this table.  
Each row in this table indicates the product's price in n different stores.  
If the product is not available in a store, the price will be null in that store's column.  
The names of the stores may change from one testcase to another. There will be at least 1 store and at most 30 stores.
```
`Important note:` This problem targets those who have a good experience with SQL. If you are a beginner, we recommend that you skip it for now.

Implement the procedure `UnpivotProducts` to reorganize the `Products` table so that each row has the id of one product, the name of a store where it is sold, and its price in that store. If a product is not available in a store, do `not` include a row with that `product_id` and `store` combination in the result table. There should be three columns: `product_id`, `store`, and `price`.

The procedure should return the table after reorganizing it.

Return the result table in `any order`.

SQL Schema
```sql
Truncate table Products  
insert into Products (product_id, LC_Store, Nozama, Shop, Souq) values ('1', '100', 'None', '110', 'None')  
insert into Products (product_id, LC_Store, Nozama, Shop, Souq) values ('2', 'None', '200', 'None', '190')  
insert into Products (product_id, LC_Store, Nozama, Shop, Souq) values ('3', 'None', 'None', '1000', '1900')
```
## Idea

The query result format is in the following example.
```
+------------+----------+-------+  
| product_id | store    | price |  
+------------+----------+-------+  
| 1          | LC_Store | 100   |  
| 1          | Shop     | 110   |  
| 2          | Nozama   | 200   |  
| 2          | Souq     | 190   |  
| 3          | Shop     | 1000  |  
| 3          | Souq     | 1900  |  
+------------+----------+-------+
```
Refer [bofeng07](https://leetcode.com/bofeng07/)'s MySQL solution, it a grate idea!

Getting the column names of table `Products` from [information_schema.columns](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html), using `group_concat` to combian each column values of each row of table `Products` by `union` .

## Solution
```sql
CREATE PROCEDURE UnpivotProducts()  
BEGIN  
  
    set session group_concat_max_len = 1000000;  
  
    set @macro = null;  
      
    select group_concat(  
        concat(  
            'select product_id, "', column_name, '" as store, ', column_name, ' as price ',  
            'from Products ',  
            'where ', column_name, ' is not null'  
        ) separator ' union '  
    )  
    into @macro  
    from information_schema.columns   
    where table_schema='test' and table_name='Products' and column_name != 'product_id';  
  
    prepare sql_query from @macro;  
    execute sql_query;  
    deallocate prepare sql_query;  
   
END
```
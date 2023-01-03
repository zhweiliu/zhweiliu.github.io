---
title: '[leetcode][Database][Hard] 571. Find Median Given Frequency of Numbers'
date: '2022-12-06T16:31:02.761Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
showToc: true
TocOpen: true
---

## 題目

Table: `Numbers`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| num         | int  |  
| frequency   | int  |  
+-------------+------+  
num is the primary key for this table.  
Each row of this table shows the frequency of a number in the database.
```
The [`median`](https://en.wikipedia.org/wiki/Median) is the value separating the higher half from the lower half of a data sample.

Write an SQL query to report the `median` of all the numbers in the database after decompressing the `Numbers` table. Round the median to `one decimal point`.

SQL Schema
```sql
Create table If Not Exists Numbers (num int, frequency int)  
Truncate table Numbers  
insert into Numbers (num, frequency) values ('0', '7')  
insert into Numbers (num, frequency) values ('1', '1')  
insert into Numbers (num, frequency) values ('2', '3')  
insert into Numbers (num, frequency) values ('3', '1')
```
## 解題思考

*   題目要求輸出 `Numbers` 表格的中位數 `median`。  
    本題要求的邏輯為 :   
    1\. 先將 `Numbers` 內的數字 `num` 以頻次 `frequency` 展開為等長的數列  
    2\. L ← 將所有 `num` 展開的數列依 `num` 大小有序串接   
    3\. 求出 L 的中位數 `median`，並將中位數作為輸出結果
```
Input:   
Numbers table:  
+-----+-----------+  
| num | frequency |  
+-----+-----------+  
| 0   | 7         |  
| 1   | 1         |  
| 2   | 3         |  
| 3   | 1         |  
+-----+-----------+  
Output:   
+--------+  
| median |  
+--------+  
| 0.0    |  
+--------+  
Explanation:   
If we decompress the Numbers table,   
we will get [0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3],   
so the median is (0 + 0) / 2 = 0.
```
*   中位數的定義 → 在一組數列 `_S_` 中，有一半數字個數會小於中位數，另外一半數字個數會大於中位數。假設  
    `_L_` ← 數列 `_S_` 中，小於等於 `Numbers.num` 的數字個數  
    `_R_` ← 數列 `_S_` 中，大於等於 `Numbers.num` 的數字個數  
    `_N_` ← 數列 `_S_` 中，`Numbers.num` 的數字個數  
    則數列 `_S_` 的數字個數應等同於 `_L + N + R_` ， 對 `Numbers` 中的所有 `num` ，該條件都成立
*   考慮當 `_L_` ≠ `_R_` 的情況，若`Numbers.num`是中位數，當 `_N_` 加入至個數較少的`短邊`時 ( `_L_` 或 `_R_` ) ，被 `_N_` 加入的`短邊`會成為`長邊`:  
    當 `_L < R_`，`_L + N > R  
    _`當 `_R < L_`，`_R + N > L  
    _`若`Numbers.num`不是中位數，當 `_L < R_ 時`，即使把 `_N_` 放到短邊，仍會得到 `_L + N < R_` 的結果，這表示 `_N_` 不在數列中間
*   考慮當 `_L = R_` 的情況，根據定義已知 `Numbers.num`是中位數

## 解決方案
```sql
select    
  round(avg(n.num),1) median  
from Numbers n  
where   
  n.Frequency >= abs(  
                  (select sum(Frequency) from Numbers where num<=n.num) -  
                  (select sum(Frequency) from Numbers where num>=n.num)  
                  )
```
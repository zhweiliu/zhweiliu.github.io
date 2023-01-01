---
title: '[leetcode][Database][Hard] 2118. Build the Equation'
date: '2022-12-16T23:57:37.348Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Terms`
```
+-------------+------+  
| Column Name | Type |  
+-------------+------+  
| power       | int  |  
| factor      | int  |  
+-------------+------+ 

power is the primary key column for this table.  
Each row of this table contains information about one term of the equation.  
power is an integer in the range [0, 100].  
factor is an integer in the range [-100, 100] and cannot be zero.
``` 

You have a very powerful program that can solve any equation of one variable in the world. The equation passed to the program must be formatted as follows:

*   The left-hand side (LHS) should contain all the terms.
*   The right-hand side (RHS) should be zero.
*   Each term of the LHS should follow the format `"<sign><fact>X^<pow>"` where:  
    `<sign>` is either `"+"` or `"-"`.  
    `<fact>` is the `absolute value` of the `factor`.  
    `<pow>` is the value of the `power`.
*   If the power is `1`, do not add `"^<pow>"`.  
    For example, if `power = 1` and `factor = 3`, the term will be `"+3X"`.
*   If the power is `0`, add neither `"X"` nor `"^<pow>"`.  
    For example, if `power = 0` and `factor = -3`, the term will be `"-3"`.
*   The powers in the LHS should be sorted in `descending order`.

Write an SQL query to build the equation.

SQL Schema
```sql
Create table If Not Exists Terms (power int, factor int)  
Truncate table Terms  
insert into Terms (power, factor) values ('2', '1')  
insert into Terms (power, factor) values ('1', '-4')  
insert into Terms (power, factor) values ('0', '2')
```

## Idea

The query result format is in the following example.
```
+-----------------+  
| equation        |  
+-----------------+  
| -4X^4+1X^2-1X=0 |  
+-----------------+
```
Fulfill requirements :

For generating the equation by records of table `Terms` , it can be reorganize to a few parts of a term `{factor sign}{absolute factor value}{power of X sign}{power value}`   
For example :  
- Term will be present `-4X` when the `record.factor=-4` , `record.power=1`   
- Term will be present `+2x^4` when the `record.factor=2` , `record.power=4`   
- Term will be present `+1` when the `record.factor=1` , `record.power=0`

Finally, using the function `group_concat()` combainating all of the terms from reorganize.

## Solution
```sql
with  
builder as (  
    select 0 as LHS, -1 as rk, '=0' as e  
    union  
    select  
        0 as LHS,  
        row_number() over(order by power) rk,  
        concat(  
            if(factor >0, '+', '-'), -- factor sign  
            abs(factor), -- remove factor sign of the value  
            if(power=0, '', if(power=1, 'X', 'X^') ), -- power of X  
            if(power<2, '', power) -- show power text when power large than 2  
        ) e  
    from Terms  
)  
  
select   
    group_concat( e order by rk desc separator '' ) as equation  
from builder  
group by LHS
```
---
title: '[leetcode][Database][Hard] 2199. Finding the Topic of Each Post'
date: '2022-12-18T20:54:39.019Z'
categories: ['leetcode']
keywords: ['database', 'mysql']
---

## Description

Table: `Keywords`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| topic_id    | int     |  
| word        | varchar |  
+-------------+---------+  
(topic_id, word) is the primary key for this table.  
Each row of this table contains the id of a topic and a word that is used to express this topic.  
There may be more than one word to express the same topic and one word may be used to express multiple topics.
```

Table: `Posts`
```
+-------------+---------+  
| Column Name | Type    |  
+-------------+---------+  
| post_id     | int     |  
| content     | varchar |  
+-------------+---------+  
post_id is the primary key for this table.  
Each row of this table contains the ID of a post and its content.  
Content will consist only of English letters and spaces.
```
Leetcode has collected some posts from its social media website and is interested in finding the topics of each post. Each topic can be expressed by one or more keywords. If a keyword of a certain topic exists in the content of a post (`case insensitive`) then the post has this topic.

Write an SQL query to find the topics of each post according to the following rules:

*   If the post does not have keywords from any topic, its topic should be `"Ambiguous!"`.
*   If the post has at least one keyword of any topic, its topic should be a string of the IDs of its topics sorted in ascending order and separated by commas `','`. The string should not contain duplicate IDs.

Return the result table in `any order`.

SQL Schema
```sql
Create table If Not Exists Keywords (topic_id int, word varchar(25))  
Create table If Not Exists Posts (post_id int, content varchar(100))  
Truncate table Keywords  
insert into Keywords (topic_id, word) values ('1', 'handball')  
insert into Keywords (topic_id, word) values ('1', 'football')  
insert into Keywords (topic_id, word) values ('3', 'WAR')  
insert into Keywords (topic_id, word) values ('2', 'Vaccine')  
Truncate table Posts  
insert into Posts (post_id, content) values ('1', 'We call it soccer They call it football hahaha')  
insert into Posts (post_id, content) values ('2', 'Americans prefer basketball while Europeans love handball and football')  
insert into Posts (post_id, content) values ('3', 'stop the war and play handball')  
insert into Posts (post_id, content) values ('4', 'warning I planted some flowers this morning and then got vaccinated')
```
## Idea

The query result format is in the following example.
```
+---------+------------+  
| post_id | topic      |  
+---------+------------+  
| 1       | 1          |  
| 2       | 1          |  
| 3       | 1,3        |  
| 4       | Ambiguous! |  
+---------+------------+
```
Fulfill requirements :

I guess that will maybe use the functions `SUBSTRING` or `SUBSTRING_INDE` when most people, include me, seem this question at first. But I would like to solve this question via function `INSTR` .

As we can find the definition for `INSTR` in MySQL [official documentation](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_instr).

Returns the position of the first occurrence of substring substr in string str.   
This is the same as the two-argument form of LOCATE(),   
except that the order of the arguments is reversed.  
```  sql
mysql > SELECT INSTR('foobarbar', 'bar');  
        -> 4  
mysql > SELECT INSTR('xbar', 'foobar');  
        -> 0
```
So, in the first step, an easy way for adding two space characters as prefix and suffix for a post content, it’s help to find the first/last words are keyword or not in a post content via function `INSTR` to use `{SPACE_SYMBOL}{keyword}{SPACE_SYMBOL}` as a condition.

`INSTR` will return a position(or an index) of the content if a keyword in this content, but also return `0` if a keyword not in this content.

Finally, using the table `Posts` as a main table in query, and using `left join` to associate the result from `INSTR` to map if topic(s) is/are in a post content. Also, replacing the topic to `Ambiguous!` if here haven’t a topic in.

## Solution
```sql
with  
cte_post as (  
    select post_id, concat(' ', content, ' ') as content from Posts -- The easy way for INSTR() to find keyword  
),  
cte_match_topics as (  
    select  
        a.post_id,  
        group_concat(distinct b.topic_id separator ',') as topic  
    from cte_post a, Keywords b  
    where INSTR(a.content, concat(' ', b.word, ' ')) > 0 -- find keyword position   
    group by a.post_id  
)  
  
select   
    distinct(a.post_id),  
    ifnull(b.topic, 'Ambiguous!') as topic  
from Posts a  
left join cte_match_topics b using(post_id)  
order by 1
```
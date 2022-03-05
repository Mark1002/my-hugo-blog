---
title: SQL cross join 的語法差異
date: 2019-08-04 17:14:06
categories: sql
---
今天遇到了一個 sql join 的語法問題，首先第一個 sql 如下：
```sql=
SELECT c.customer_id, o.order_id, o.order_date 
FROM customers as c JOIN orders as o
ON c.customer_id=o.customer_id;
```
很明顯的這是一個簡單的 inner join 任務，選擇出符合條件``c.customer_id=o.customer_id``的共同紀錄。

然而 sql 有另一種不用明確指定使用 `join` 的方式也可以達到一樣的結果，第二個 sql 如下：
```sql=
SELECT c.customer_id, o.order_id, o.order_date 
FROM customers as c, orders as o
WHERE c.customer_id=o.customer_id;
```
兩種語法都可以那應該本質上差不多吧？**我錯了，大錯特錯。**
本來一直天真的以為 ``FROM TABLE1, TABLE2`` 背後應該也是 inner join 的語法糖吧？結果我後來[網路查證](https://stackoverflow.com/questions/334201/why-isnt-sql-ansi-92-standard-better-adopted-over-ansi-89/47720615#47720615)得到的結果是當 sql 表示成 ``FROM TABLE1, TABLE2`` 會對這兩張表進行 **cross join**，等同於 ``FROM TABLE1 CROSS JOIN TABLE2 ``。
而什麼是 cross join 呢？其實就是對這兩張表的各個紀錄做笛卡兒積 (cartesian product)，如下圖所示：
![](https://i.imgur.com/HQKLEMc.png)
所以如果 table1、table2 各有 N、M 筆記錄，那經過 cross join 後就會有 N * M 筆記錄 !
這實在是與 inner join 依據條件取出交集的行為天差地遠。以下用另一張圖表示各個 join 的關係：

![](https://i.imgur.com/OVK6KhR.png)

可以看出 cross join 與其他 join 的行為明顯不同。與 inner join 減少選擇資料記錄相比，**cross join 則是會大幅度增加資料記錄!**

因此，可以想像如果不小心使用了第二個 sql 會導致不必要的資料表 cross join 的動作，在資料量大的環境時很容易就增加資料庫效能的負擔。即使最後有用 WHERE 篩選條件讓結果看起來一樣，但是前面光是 cross join 取出的記錄筆數就實在與直接下 inner join 差太多了！

而為什麼會有第二種寫法呢？我經過網路查證發現其實第二種寫法是源自資料庫標準協定 ANSI-89 的舊式寫法，那是一個還沒有 join 語法的舊協定，要表示成 inner join 只能用 ``FROM TABLE1, TABLE2`` 這種 implicit 的方式，新的協定 ANSI-92 才有 join 的寫法。這次的心得就是**要好好地善用 join 以及認真的再把資料庫複習一遍！**


reference:
1. https://www.techonthenet.com/sql/joins_try_sql.php
2. https://stackoverflow.com/questions/334201/why-isnt-sql-ansi-92-standard-better-adopted-over-ansi-89
3. http://www.sqlcourse2.com/joins.html
4. https://stackoverflow.com/questions/11861417/what-is-the-difference-between-cartesian-product-and-cross-join
5. https://stackoverflow.com/questions/17946221/sql-join-and-different-types-of-joins
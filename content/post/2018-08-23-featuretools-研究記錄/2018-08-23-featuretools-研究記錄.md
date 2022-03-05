---
title: featuretools 研究記錄
date: 2018-08-23 22:06:18
categories: data science
---
最近 survey 到了一個很方便的特徵工程工具，叫 featuretools，其特色是專門針對關聯式資料庫的資料表做自動特徵生成，其強大之處在於可以一次關聯到好幾張表，並且可指定多種特徵公式來自動產生特徵，以下來介紹我的學習心得。

首先一樣透過 pip 來安裝如下： 
```
pip install featuretools
```
featuretools 裡面有個非常重要的概念叫 **DFS (Deep Feature Synthesis)**，中文叫深度特徵生成，一個很潮的名詞。其實概念很簡單，就是藉由著資料表間的主鍵與外鍵的關聯去自動產生關聯的特徵。看看下圖的官方例子 ``Customers`` 這張表就是透過 ``Customer id`` 與 ``Purchases`` 產生關聯，並且利用 ``primitive`` 去產生特徵，所謂的 ``primitive`` 就好像各種不同的特徵產生公式，featuretools 已內建很多種可供選擇。這裏選用 ``Max`` 這一個 ``primitive``，對 ``Purchase Amount`` 這個欄位計算其最大值，如此就產生以下結果了。
![](https://i.imgur.com/wExSv54.png)

再來是簡單的程式使用例子，首先載入需要用到的程式模組，featuretools 也內建了方便的 demo 用資料集可以用來快速展現 DFS 的功能。
```python=
import featuretools as ft

datas = ft.demo.load_mock_customer()

sessions_df = datas['sessions']
customers_df = datas['customers']
transactions_df = datas['transactions']
```
sessions、customers、transactions 分別代表彼此有關聯的資料表，這裏皆以 pandas dataframe 的形式儲存。在 featuretools 定義裡這些表就是所謂的 **entity**，可以作為 DFS 的執行目標單位。以下為這三張表的部分記錄。
```python=
customers_df.head()
```
![](https://i.imgur.com/yu7kcJk.png)
```python=
sessions_df.head()
```
![](https://i.imgur.com/2eYACmL.png)
```python=
transactions_df.head()
```
![](https://i.imgur.com/ITVjiQD.png)
可以發現這些表彼此間有主鍵外鍵的關係，也因此我們也必須明確定義給 featuretools 知道，如以下所示。
```python=
# 定義 relationship
relationships = [
    ("sessions", "session_id", "transactions", "session_id"),
    ("customers", "customer_id", "sessions", "customer_id")
]
```
也別忘了定義 entity，除了要指名 entity 的 dataframe 之外，也要指定欄位做為主鍵。
```python=
# 定義 entity
entities = {
    "customers" : (customers_df, "customer_id"),
    "sessions" : (sessions_df, "session_id", "session_start"),
    "transactions" : (transactions_df, "transaction_id", "transaction_time")
}
```
當 entity 與 relationships 都指定好了話，就可以執行看看 DFS 了，程式碼如下，第 4 行表示要被作用的目標 entity，第 5 行 ``agg_primitives`` 這裡我採用了內建的 ``Max`` primitive，第 6 行 ``max_depth`` 表示 DFS 要關聯的深度，即表示從當前的表中依照外鍵往回追朔到其他表的深度，例如 ``customers`` 要接到 `` transactions `` 要接續關聯 2 次，就像 ``customers -> sessions -> transaction``。 
```python=
feature_matrix_customers, features_defs = ft.dfs(
    entities=entities,
    relationships=relationships,
    target_entity="customers",
    agg_primitives=["Max"],
    max_depth=2
)
```
最後的結果如下，可以看到新的特徵欄位成功的出現在原來 ``customers`` 表上了，多了 ``Max`` 以及還有其他的時間欄位，時間欄位像是 `` MONTH, DAY, YEAR `` 是預設從 ``join_date`` 出來的，也是一種 primitive。總之，featuretools 還有非常多種參數可以調校，用得好的話真的事可以加快分析流程啊。
![](https://i.imgur.com/fScp3Bo.png)

reference:
1. https://www.featuretools.com/
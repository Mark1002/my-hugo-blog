---
title: Mutable Default Arguments 的陷阱
date: 2019-02-10 20:27:01
categories: python
---
最近因緣際會看到當 python 函式的預設參數為 mutable 物件時會有意想不到的行為故記錄之。考慮以下的程式，有一個帶有預測參數為``list``，的函式並且呼叫了兩次。一般預期的結果一定是 ``[3]、[4]`` 對吧？


```python=
def foo(el, targets=[]):
    targets.append(el)
    return targets

foo(3)
foo(4)
```
直接看執行結果，其實不然！執行第二次的時候發現上一次的 ``list`` targets 仍然存留著，導致 4 接著 append 上去。
```
[3]
[3,4]
```
為什麼會這樣呢？函式的預設參數照理應該是區域變數啊！其實這與 python 程式語言的機制有關。

因為 python 的函式也是物件，因此預設參數也算是函式物件的屬性之一，當 python 函式在定義階段時，預設參數這個屬性就被定義完成了，之後無論函式被呼叫幾次是不會再重新定義的，所以如果把預設參數定義成 mutable 物件就會發生上述的結果。

所以最好不要將預設參數直接指定為 mutable 物件以防產生不預期行為，若有需要則將程式改成以下即可避免。
```python=
def foo(el, targets=None):
    if targets is None:
        targets = []
    targets.append(el)
    return targets
```
這樣子 targets 這個 ``list`` 變數就會隨著函式每次呼叫而重新定義了。

reference
1. https://docs.python-guide.org/writing/gotchas/
2. http://effbot.org/zone/default-values.htm

---
title: python generator 學習紀錄
date: 2018-07-31 23:09:43
categories: python
---
``generator`` 是 python 程式語言中一項強大的訪問集合元素的一種方式，基本上要定義一個 ``generator`` 只要在 function 內使用 ``yield`` 關鍵字就可以了。最簡單的方式就如以下定義：
```python=
def foo():
    yield "hello world!"
```
一個 ``generator`` 就完成了，可以看看呼叫的結果會回傳一個表示``generator``的物件。而且要注意的是**呼叫變成``generator``的 function 時並不會執行 function body 中的程式邏輯。**
```
foo()
<generator object foo at 0x103ee07d8>
```
要真正的觸發``generator``內部的程式邏輯，必須要使用迭代器的操作方式，像是使用 ``next``、迴圈等，才會觸發程式邏輯。因為``generator`` 本身就是個迭代器。就像如下的例子，要依序走訪 ``generator`` 呼叫 python 的內建函數 ``next``就可以了。
```python=
f_iter = foo()
next(f_iter)
```
呼叫的結果不意外地 "hello world!" 就被印出來了。
```
next(f_iter)
'hello world!'
```
要稍微注意的是迭代器只能往前不會後退的特性，若元素皆走訪完在呼叫一次 next 時會噴出 StopIteration 錯誤。

接下來介紹 ``generator`` 的實作關鍵 ``yield`` 的使用方式，``yield`` 一開始乍看之下會以為與一般 function 的 ``return ``很像，但事實上卻很不同。 
首先，當 function 內部使用到 ``yield``時，這個 function 就被視為 ``generator`` 了，只有在迭代階段才會執行程式邏輯。而在迭代階段執行程式碼到 ``yield`` 那行時，``yield`` 會使當下的函式運行暫停下來並保存當前所有的的運行狀態，接著 ``yield`` 會回傳當前的結果給迭代器，到下一個迭代階段程式才會從方才``yield`` 中斷的地方回復執行。舉一個例子如下：
```python=
def alternate_zero_and_one():
    while True:
        state = 1
        yield state
        state = 0
        yield state

zero_one_gen = alternate_zero_and_one()
```
這是一個會不停交替產生出 1 與 0 的 ``generator`` 函式，來看看以下執行 next 迭代的結果。
```
next(zero_one_gen)
1
next(zero_one_gen)
0
next(zero_one_gen)
1
```
當第一次迭代執行到第一個 ``yield`` 時，由於之前 ``state = 1`` 所以 ``yield`` 回給迭代器的值就是 1，之後到了第二個迭代，程式並不會從 function body 的第一行從頭執行，**而是會從剛才第一個 ``yield`` 的中斷處往下開始執行**。所以接著執行 ``state = 0``，之後碰到第二個 ``yield`` 以此類推，暫停並回傳當前結果，到下一迭代再從暫停處回復執行。這樣不停的 0 與 1 迭代功能就完成了。

然而使用 ``generator`` 有什麼好處呢？以我的角度來看 ``generator`` 可以在一些需產生大量集合元素的任務派上用場。看看以下第一種用 list 實作產生大量集合元素的方法。可以看到裡面有一個區域變數 `result_list`，會一直將產生的字串加入到該 list 中，加完之後回傳整個結果 list。 
```python=
def foo_list(num):
    count = 0
    result_list = []
    while count < num:
        count += 1
        result_list.append("yo!")
    return result_list

for m in foo_list(9999999):
    print(m)
```
很明顯的這段程式碼最大的問題就是當指定的 ``num``很大時，`result_list` 每次都儲存的結果會吃掉大量的記憶體資源，後果極可能造成程式 crash 掉。所以``generator`` 就派上用場了，改寫程式如下。
```python=
def foo_gen(num):
    count = 0
    while count < num:
        count += 1
        yield "yo!"

for m in foo_gen(9999999):
    print(m)
```
由於 ``generator`` 在迭代階段才運行程式邏輯的特性，所以根本不需要額外的區域變數來儲存每次迭代運算的結果，大大的降低記憶體資源的使用量，同時也可以達到與 list 方法一樣的結果。

最後 ``generator`` 的強大之處不只如此，python 的 concurrency 非同步 IO 基礎就是基於
``generator`` 而實現的，之後就待我研究完在分享吧!

reference:
1. http://www.runoob.com/python3/python3-iterator-generator.html
2. https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do

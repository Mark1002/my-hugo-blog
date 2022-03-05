---
title: Duck typeing in python
date: 2018-02-10 16:44:09
comments: true
categories: python
---
duck typeing 是程式語言的一種不需要用到繼承就可以有多型的特性。像 python 這種弱型別語言就有 duck typeing，以下程式碼展示了 duck typeing 的狀況：

```python=
class Duck:
    def scream(self):
        print("Cuack!")

    def walk(self):
        print("Walking like a duck...")

class Person:
    def scream(self):
        print("Ahhh!")

    def walk(self):
        print("Walking like a human...")

def activate(duck):
    duck.scream()
    duck.walk()

if __name__ == "__main__":
    Donald = Duck()
    John = Person()
    activate(Donald)
    # this is not supported in other languages, because John
    # is not a Duck object
    activate(John)
```
```
Cuack!
Walking like a duck...
Ahhh!
Walking like a human...
```
程式中函式 activate 接收了 duck 物件當作參數並呼叫其中的方法，但由於 python 是弱型別語言，
可以發現函式 activate 不管接收哪種物件，只要物件中有跟 Duck 類別有一樣同名的方法，這些方法都會呼叫。強型別語言由於要在編譯前就嚴格指定參數的型別因此不會有 duck typeing 這個問題。
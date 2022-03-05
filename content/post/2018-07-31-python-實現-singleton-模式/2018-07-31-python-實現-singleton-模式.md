---
title: python 實現 singleton 模式
date: 2018-07-31 23:01:04
categories: design pattern
---
singleton 模式指的是一個類別所實體化的物件永遠是唯一的，或者說此類別只可以實體化物件一次，不多也不少。因此，從程式任何地方所存取此類別的物件都是同一個物件，很適合運用在資料庫連線等唯一物件的情況。而且 singleton 模式不需要宣告成全域變數就可以達到全局存取的效果，是個非常好用的設計模式。而 python 實現 singleton 模式的模式非常多元，以下來介紹實現 singleton 的三種方式。

**1. 標準方式**
以下是類似 Java 的標準 singleton 模式，有一個表示私有的類別變數 ``_instance`` 儲存 singleton 模式的物件，並用一個靜態方法 ``get_instance()`` 來負責物件的取得與物件實體化。只不過 python 沒有真正私有的機制，所以還是可以存取與改變內部類別的屬性。
```python=
class SingleTon:
    # 加底線表示為私有變數
    _instance = None

    @staticmethod
    def get_instance():
        if SingleTon._instance is None:
            SingleTon()
        return SingleTon._instance
    # 此處應為私有的建構子
    def __init__(self):
        if SingleTon._instance is not None:
            raise Exception('only one instance can exist')
        else:
            self._id = id(self)
            SingleTon._instance = self
    
    def get_id(self):
        return self._id

```
**2. 利用 python 模組全域特性**
這招算是最簡單的一個技巧可以快速實現 singleton 模式，其利用 python 模組本身就是全域特性，先在一個模組裡先實體化一個物件，之後程式其他任何地方要用到這個物件那就只要 import 這個模組裡的物件就好了。
```python=
# singleton_module.py
class _SingleTon:
    def __init__(self):
        self._id = id(self)
    
    def get_id(self):
        return self._id

SingleTonObject = _SingleTon()
# from singleton_module import SingleTonObject
```

**3. 使用 metaclass 改變類別定義行為**
這招算是很進階的寫法，利用 metaclass 來預先改變定義類別的行為來達到 singleton 模式的效果。在 python 的世界裡，**類別也是一種物件**，而所謂的 **metaclass 就是定義類別這個物件的類別**。聽起來非常抽象，這篇[參考](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)有對 metaclass 做了一個詳盡的介紹。以下就是 metaclass 的寫法。第 13 行 ``metaclass=SingletonMetaclass`` 指定了 Singleton 這個類別定義的方式。裡面的關鍵是在第 6 行 ``__call__`` 裡面，這個 ``__call__`` 可以用來定義類別的建立與初始化行為，其中注意此處的``self.__instance`` 的 self 指的可是被定義的類別本身實體，而不是類別實體化的物件。 
```python=
class SingletonMetaclass(type):
    def __init__(self, *args, **kwargs):
        self.__instance = None
        super().__init__(*args, **kwargs)
    # 決定若使用類別來建構物件時，該如何進行物件的建立與初始
    def __call__(self, *args, **kwargs):
        if self.__instance is None:
            self.__instance = super().__call__(*args, **kwargs)
            return self.__instance
        else:
            return self.__instance

class Singleton(metaclass=SingletonMetaclass):
    def __init__(self):
        self._id = id(self)

    def get_id(self):
        return self._id
```
接下來展示一下三種方式的測試結果。這裡簡單定義兩個 python module，裡面都呼叫了相應的 singleton，來看看在不同的 module 裡是否都是指向同一個物件。
```python=
# module1.py and moudle2.py
from singleton_module import SingleTonObject
from singleton_meta import Singleton
from singleton_standard import SingleTon

def execute():
    print("standard method: {}".format(SingleTon.get_instance().get_id()))
    print("module method: {}".format(SingleTonObject.get_id()))
    print("metaClass method: {}".format(Singleton().get_id()))
```
執行以下主程式來查看結果。
```python=
import module1
import module2

if __name__ == '__main__':
    print('module1:')
    module1.execute()
    print('module2')
    module2.execute()
```
可以發現這三種方法都成功了實現 singleton 模式，module1 與 module2 的物件都是指向同一個物件。
```
module1:
standard method: 4301794776
module method: 4301894488
metaClass method: 4301795168
module2
standard method: 4301794776
module method: 4301894488
metaClass method: 4301795168
```
**2019/05/26 補充：**
經過多個月後發現其實 python3 的內部機制就可以辦到了....Orz，關鍵就是利用 ``__new__`` 來去改變物件的建構行為。**注意: python 真正建構物件的方法其實是 ``__new__``, ``__init__`` 則是負責對建構後的物件值初始化，所以在 ``__init__`` 之前物件已經建立了。** 而且``__new__``本身就是 static 方法，所以像之前方法一的方式可以完全被取代。
```python
class SingleTonNew: 
    _instance = None 
    def __new__(cls, *args, **kwargs): 
        if cls._instance is None: 
            cls._instance = super().__new__(cls) 
        return cls._instance 
         
    def __init__(self, a, b): 
        self.a = a 
        self.b = b

s1 = SingleTonNew(a=11, b=21)
s2 = SingleTonNew(a=11, b=21)
id(s1)
id(s2)
```
一樣可以實現單例模式，且更簡潔。
```
4468430664
4468430664
```


reference:
1. https://juejin.im/post/5a64255c51882573432d42e0
2. https://gist.github.com/pazdera/1098129
3. https://gist.github.com/werediver/4396488
4. https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python
5. https://www.code-learner.com/how-to-use-python-__new__-method-example/
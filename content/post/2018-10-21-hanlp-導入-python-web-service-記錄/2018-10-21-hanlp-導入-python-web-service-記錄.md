---
title: hanlp 導入 python web service 記錄
date: 2018-10-21 20:51:18
categories: python
---
[hanlp](http://hanlp.com/) 是一個由大陸開發非常強大的中文自然語言處理工具，由 Java 開發，目前也有提供 python 的介面 [pyhanlp](https://github.com/hankcs/pyhanlp) 了。之前的任務為要使用 flask 並結合 hanlp 寫成一個 API 的 web service，果然遇到一些導入上的問題，以下來記錄我成功串接的過程。

首先，要將 hanlp 打包成 python 的 web service 用一般 python 載入模組的方法像 ``import pyhanlp`` 是行不通的，在一般的 py 執行檔是可以使用，但用在 python web 框架 flask 上服務會 crash 掉。原因好像為 ``pyhanlp`` 跟底層 Java 的溝通方式無法運用在 web 環境有關。因此第一個關鍵為要改使用可讓 python 直接溝通 java 的套件 ``jpype``，來直接取用 hanlp 的 jar 檔。程式碼如以下所示。
```python=
hanlp_lib_path = "./lib/"
java_class_path = hanlp_lib_path + 'hanlp-1.6.8.jar' + ':' + hanlp_lib_path
startJVM(getDefaultJVMPath(), '-Djava.class.path=' + java_class_path, '-Xms1g', '-Xmx1g')
```
可以看到前面指定了 jar 檔的位置並加入給 JVM 執行，這裡我在同一個專案目錄下的 ``lib`` 資料夾結構如下。

```
lib/
    data/
    hanlp-1.6.8.jar
    hanlp.properties
    hanlp.properties.in
```
``data`` 就是辭典的部分，共 600 多 MB，``hanlp-1.6.8.jar`` 就是 hanlp jar 檔本體了，而``hanlp.properties`` 用來配置辭典的路徑，這裡要在``hanlp.properties``中最前面第一行加上 ``root=./lib/``表示作用的根目錄。

再來第二重要點就是由於 API 呼叫為多執行緒的環境，因此也必須在 java class 執行之前加上以下程式來讓 JVM 支援多執行緒，確保 API 服務不會掛掉。
```python=
if not jpype.isThreadAttachedToJVM():
    jpype.attachThreadToJVM()
```
完整的程式碼如下所示，這裡我把 hanlp 呼叫寫成了一個類別，這樣就可以在 flask 或 Django 等 API 框架直接 import 我這個 hanlp 類別而不會報錯了。
```python=
import logging
import jpype
from jpype import startJVM, JClass, getDefaultJVMPath

logger = logging.getLogger(__name__)
hanlp_lib_path = "./lib/"
java_class_path = hanlp_lib_path + 'hanlp-1.6.8.jar' + ':' + hanlp_lib_path
startJVM(getDefaultJVMPath(), '-Djava.class.path=' + java_class_path, '-Xms1g', '-Xmx1g')

class NerService:
    def __init__(self):
        self.raw_text = None
    
    def set_raw_text(self, raw_text):
        self.raw_text = "".join(raw_text.split())
        logger.info(self.raw_text)
    
    def perform_ner(self):
        # 啟動支援 thread
        if not jpype.isThreadAttachedToJVM():
            jpype.attachThreadToJVM()
        PerceptronLexicalAnalyzer = JClass('com.hankcs.hanlp.model.perceptron.PerceptronLexicalAnalyzer')
        analyzer = PerceptronLexicalAnalyzer()
        s = str(analyzer.analyze(self.raw_text))
        self.hanlp_result_str = s
        analyze_list = s.split(" ")
        return analyze_list
```
reference
https://www.jianshu.com/p/d7e7cc747e56
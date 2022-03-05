---
title: python log 檔限制大小問題
date: 2018-05-27 16:25:55
categories: python
---
**學習如果一知半解是件非常可怕的事**。之前對於 python log 的使用就是這樣，導致程式寫完出了不少的包，因此我必須有必要重新理解 python logging 套件的使用機制。

遇到一個問題就是為什麼有加入了限制 log 檔大小的屬性 ``maxBytes`` 指定大小不超過 5 MB，但為什麼 log 檔案大小還是會無限制的增加呢?
以下是我原先的 django log 設定：
```python=
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '[ %(asctime)s; %(filename)s:%(lineno)d ] %(levelname)s:%(name)s: %(message)s'
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'log/debug.log',
            'maxBytes': 5*1024*1024,
            'formatter': 'standard',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'standard',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```
logging 負責寫入 log 檔的 handler 是由 `RotatingFileHandler` 這個類別來處理的。這個類別會將記錄存到 log 檔中，若裏頭存放的紀錄即將超過 `maxBytes` 的設定範圍時，則會備份舊的記錄到備份檔中，並且依照 `backupCount` 給定的數目來依序產生備份檔。像是有個 log 檔 `app.log`，若 `backupCount=3` 則將會依序產生 `app.log.1`, `app.log.2`, `app.log.3` 等備份檔，若這些備份都滿了，則會回到 `app.log` 從頭開始將新的記錄蓋過舊的記錄，所以故名思義真得跟 rotating ㄧ樣。

![](https://i.imgur.com/4yoYmNe.png)


經過一方折騰終於發現問題出在哪了。以下是取自[官方文件說明](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.RotatingFileHandler)：
*Rollover occurs whenever the current log file is nearly maxBytes in length; **but if either of maxBytes or backupCount is zero, rollover never occurs**, so you generally want to set backupCount to at least 1, and have a non-zero maxBytes.*
原因就是我沒有指定 `backupCount`， 而這個參數預設就是 0，沒有備份檔根本就不能將舊的記錄寫入，所以很自然的 log 檔就會越長越大了。
所以非常重要的一點是必須要加上 `backupCount` 這個參數，且值至少為 1，`RotatingFileHandler` 才可以根據 `maxBytes` 設定來將舊紀錄寫入備份檔。
再來值得注意的是，如果 log 是寫在 django 的話，則 django develop server 所對應的執行指令必須要加上 `--noreload` 參數，`RotatingFileHandler` 的設定才會有用，原因是 django 再 develop server 模式下會有兩個 thread，一個是執行 django 服務的主體，另一個則是監聽程式碼有沒有被修改，有修改則 reload 服務。所以若不把 reload 服務的 thread 關掉，則 log 檔就會被這個 thread 占住了，無法讓 ``RotatingFileHandler`` 對 log 檔進行大小限制處理，詳細指令如以下所示:
```
python manage.py runserver --noreload
```
所以我在 16 行加上了 `backupCount
` 並指定為 1，如此就可以解決 django 中 log 檔無限制增加的問題了。
```python=10
'file': 
    { 
        'level': 'INFO', 
        'class':'logging.handlers.RotatingFileHandler', 
        'filename': 'log/debug.log', 
        'maxBytes': 5*1024*1024,
        'backupCount': 1,
        'formatter': 'standard', 
    }
```

reference:
1. https://stackoverflow.com/questions/40088496/how-to-use-pythons-rotatingfilehandler
2. https://stackoverflow.com/questions/46401571/python-3-logger-with-rotatingfilehandler-excedes-maxbytes-limit
---
title: 解決 docker 執行 crontab 環境變數遺失問題
date: 2020-10-03 14:25:17
categories: docker
---
今天遇到了在 docker 執行 crontab 環境變數消失的問題。明明在 docker 裡已經有設環境變數了，但 crontab 實際在執行時**並不會考慮所有的環境變數**。

試了很多方法都沒有用，最後參考了一種在 crontab 執行前先將 docker 內所有設好的環境變數都寫入一個 shell script 中，如此就可以在 crontab 執行真正排程指令前執行此 shell script 載入環境變數。


```dockerfile=
FROM python:3.7-slim-buster
ENV TZ=Asia/Taipei
WORKDIR /dump
COPY . .
RUN apt-get update && apt-get install -y curl cron && \
    pip install -r requirements.txt

ENTRYPOINT /dump/run-cron.sh
```
dockerfile 如上圖所示，其中的關鍵是在 ``run-cron.sh``，程式如下：

```bash=
#!/bin/bash

ENV_VARS_FILE="/root/.env.sh"

echo "Dumping env variables into ${ENV_VARS_FILE}"
printenv | sed 's/^\(.*\)$/export \1/g' > ${ENV_VARS_FILE}
chmod +x ${ENV_VARS_FILE}

echo "Applying crontab"
crontab /dump/crontab.txt

echo "Running crontab"
cron -f
```
裡面的第 6 行，用 ``printenv`` 印出 docker 環境變數再用 ``sed`` 指令取出每一行變數組合成 ``export VARIABLE_NAME=VALUE`` 形式在寫入 sh 檔，crontab 則以如下設定來執行。
```
0 */3 * * * . /root/.env.sh; /usr/local/bin/python /dump/dumper.py -days=7 >> /var/log/cron.log 2>&1
```
在真正要執行的指令之前，再加上 ``. /root/.env.sh;`` 來去確實載入所有環境變數。

ref:
1. https://gist.github.com/athlan/b6f09977e2f5cf20840ef61ca3cda932
2. https://www.sitepoint.com/a-comprehensive-crash-course-into-cronjobs/

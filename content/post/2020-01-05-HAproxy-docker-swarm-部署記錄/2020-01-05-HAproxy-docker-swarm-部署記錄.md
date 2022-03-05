---
title: HAproxy docker swarm 部署記錄
date: 2020-01-05 21:45:57
categories: loading balancing
---
久聞 Haproxy 是一個非常知名的負載均衡 (Load Balancing) 軟體，最近心血來潮，想來研究在分散式 docker-swarm 的環境來用 HAproxy 模擬負載均衡。部署的內容很簡單，先起多個不直接對外的 web server 服務，然後再啟動一個 Haproxy 做反向代理 (reverse proxy) 來為這些背後的 web server 做負載均衡以及對外的窗口。

首先，我用 python 寫了一個簡單的 http server 程式來模擬真正的 web server。

```python=
import os
from http import server

class RequestHandler(server.SimpleHTTPRequestHandler):

    def do_GET(self): 
        self.send_response(200, 'ok')
        self.end_headers()
        response = f'hostname: {os.uname()[1]}'
        self.wfile.write(response.encode('utf-8'))
        return

if __name__ == '__main__':
    with server.HTTPServer(('', 8000), RequestHandler) as http_server:
        print('start simple web server...')
        http_server.serve_forever()
```
程式的內容為 client 向 server 發出請求後，會在瀏覽器頁面上印出該 server 的 hostname，如此就可以很方便的觀察之後負載均衡的行爲。

先來看看本次 docker swarm 的部署編排，`docker-stack.yml`如下。
```yaml=
version: '3.7'

services:
  web:
    image: simple_web
    deploy:
      replicas: 3
      endpoint_mode: dnsrr 
    networks:
      - haproxy-practice
  haproxy:
    image: haproxy:2.1
    depends_on:
      - web
    volumes:
      - "./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg"
    ports:
      - "48686:8000"
    networks:
      - haproxy-practice  

networks:
  haproxy-practice:
    driver: overlay
```
這裡我打算起三個 web server 以及一個 HAproxy，並將 `endpoint_mode` 設為 `dnsrr(DNS Round Robin)` 而不用預設的 `vip(virtual IP)`，因為 docker swarm 也有自己內部的負載均衡機制叫 [routing mesh](https://docs.docker.com/network/overlay/)，會轉傳外部的請求到其中一個作用的 container。如果是預設的 `vip` 最後只會回應一個虛擬的 ip，無法得知其他 container replicas 的 ip，造成的結果就是 HAproxy 只會偵測到一台 web server。因此如果要使用外部的負載均衡就必須指定 `dnsrr` 模式才可跳過  routing mesh 取得多個 container ip。


然後就是關鍵的 HAproxy 設定部分了，HAproxy 的設定檔為 `haproxy.cfg`，內容如下所示。
```
global
    daemon
    maxconn 256

resolvers docker
    nameserver dns1 127.0.0.11:53
    resolve_retries 3
    timeout resolve 1s
    timeout retry   1s
    hold other      10s
    hold refused    10s
    hold nx         10s
    hold timeout    10s
    hold valid      10s
    hold obsolete   10s

defaults
    timeout connect 10s
    timeout client 30s
    timeout server 30s
    mode http

frontend  haproxyPractice
    bind *:8000
    use_backend stat if { path -i /my-stats }
    default_backend web_service 

backend web_service
    balance roundrobin
    server-template web- 3 web:8000 check resolvers docker init-addr libc,none

backend stat
    stats enable
    stats uri /my-stats
    stats refresh 15s
    stats show-legends
    stats show-node
```
設定檔中有很多不同的區塊，在此不一一贅述[這篇文章](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/)有非常詳盡的介紹。這裡挑幾個重點區塊設定來說明，首先是 `resolvers` 區塊，docker 有自己內部的 dns 服務，可以允許在同一個 docker 網路中的服務用自己的服務名稱來連線，這裡使用 docker 內部的 dns 服務 `127.0.0.11:53` 來讓 HAproxy 自動解析真實 container ip。

再來就是 `backend` web_service 區塊設定，`server-template` 參數可以快速的建立多個 server 設定而不用個別慢慢下。`web- 3`中的`web-` 表示 server 名稱的前綴，`3` 表示 server 名稱的總數，因此會產生 web-1、web-2、web-3 共三個 server，剛好與前面我設定的`replicas: 3` 對應。如果實際的 replicas 比較少，則空下來的 server 會自動 disable 掉。

同時也要指定 `resolvers docker` 讓 HAproxy 知道要使用哪個 dns resolver，`init-addr libc,none` 則指示 HAproxy 再啟動時就執行服務發現，並且在無任何 wed server 也繼續執行。

最後到了啟動的步驟了，輸入以下指令來啟動 docker swarm 服務。
```
docker stack deploy -c=docker-stack.yml haproxy-practice
```
接著不停的重整瀏覽器可以觀察每次的 hostname 都會受到負載均衡而有所不同。

![](https://i.imgur.com/RI1H5vs.png)

也可以瀏覽 `/my-stats` 來查看 HAproxy 的管理頁面看看個別服務的情形。

![](https://i.imgur.com/7RVaRM1.png)

這次部署的相關程式碼也同步於 [github](https://github.com/Mark1002/docker-haproxy-practice) 上。

reference:
1. https://www.haproxy.com/blog/haproxy-on-docker-swarm-load-balancing-and-dns-service-discovery/
2. https://www.digitalocean.com/community/tutorials/an-introduction-to-haproxy-and-load-balancing-concepts

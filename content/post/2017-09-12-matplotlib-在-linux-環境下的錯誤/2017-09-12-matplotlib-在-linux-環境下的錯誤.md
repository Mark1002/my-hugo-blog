---
title: matplotlib 在 linux 環境下的錯誤
date: 2017-09-12 22:30:26
comments: true
categories: python
---
遇到的問題為使用 python 的繪圖套件-matplotlib 在 linux 作業系統 unbantu 16.04 下所遇到的錯誤。錯誤畫面如下：
{% asset_img error1.png 奇怪的錯誤 %}
一開始以為是 race condition 的問題，但後來想想目前程式皆是用 single thread 去寫的，所以應該不是這個問題。
後來又測到了如以下的錯誤：
{% asset_img error2.png Segmentation fault! 這套件一定有問題..... %}
很明顯的可以發現繪圖套件本身極可能有問題。因此朝了此方向去 google 得知由於 matplotlib 預設是在有圖形使用者介面的環境運作的，需要有 X11 connection，然而很多的 web server 初始就設置 X11 connection ，因此直接轉移到伺服器環境常常會照成一些錯誤。參考了官網解法，加入以下程式碼即可解決。

{% codeblock lang:python %}
# do this before importing pylab or pyplot
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

{% endcodeblock %}

# 方法更新
目前的解法雖然程序不會當掉，每個request都有回應，**但是指細檢查每個 request 回傳的內容還是發現裡面還是有 error 的！！！**

以下壓力測試以 jmeter 測量，Django 不加上 --nothreading 參數啟動server，同時開 100 thread 發送 request 給 Data Chart API ，django 雖然不會掛掉但可以發現結果圖形很不穩，

![](https://i.imgur.com/8uZ5U4T.png)

在看檢視結果樹每個 request 的回傳結果是有錯誤的！！！

![](https://i.imgur.com/Liki7cx.png)

這就是表示雖然有加了 matplotlib 設定檔來指定 Agg backend，但還是沒有解決 thread safe 的問題，每一個 request 同時顯示圖片會發生 race condition 問題！！導致繪圖套件圖片會出錯！

因此，我在繪圖的程式碼區塊上下加上了 lock 來建立 critical section 來解決 race condition 問題

```python=
import threading

lock.acquire()
# critical section
# 這裡是被保護的繪圖程式碼
lock.release()
```

加上之後再重新執行 jmeter，發現不但結果圖形變穩定，100 個 request 都有成功的回應！！！

![](https://i.imgur.com/nG3M6oR.png)

每個 request 都有成功畫出圖片

![](https://i.imgur.com/vIsBi24.png)

最後是執行 1000 個 thread 結果，加上 Lock 後服務變得很穩定

![](https://i.imgur.com/08Ec68i.png)


因此加上 Lock 限制或許才是最終的解決之道。

參考資料：
https://matplotlib.org/faq/howto_faq.html
https://www.lookfor404.com/%E8%BF%90%E8%A1%8Cggplot%E5%87%BA%E7%8E%B0%E9%97%AE%E9%A2%98no-display-name-and-no-display-environment-variable/
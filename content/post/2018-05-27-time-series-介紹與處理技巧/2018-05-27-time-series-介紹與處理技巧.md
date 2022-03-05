---
title: time series 介紹與處理技巧
date: 2018-05-27 17:17:27
categories: data science
---
時間序列資料 (time series) 與一般資料最大的不同處在於時間序列的前後每一筆記綠都有相依性，不可以將每一筆紀錄當成空間中獨立的點來看。要了解時間序列資料 (time series) 必須先從以下的特性開始。
### time series 分解
首先，時間序列可以分解成以下三種特性：
1. **stationary**
中文譯為穩健性，表示時間序列資料的標準差以及平均值不會隨著時間推移而變化，而是保持著一個固定數值的狀態。stationary 是時間序列中非常著要的性質，有些預測的方法 (ARIMA) 根本上是基於此特性來去預測的。另一種 stationary 的說法為時序資料去除掉 Seasonality 與 trend 就會有 stationary 的性質了。
2. **seasonality**
表示時間序列中有相似的模式並週期性的循環出現。
3. **trend**
表示隨著時間的推移，時間序列有著明顯方向的增長或遞減。

而時間序列又可以分解成 ``additive`` 與 ``multiplicative`` 兩種不同的性質，兩者分解的公式如下：

* additive:
**yt = St + Tt + Et**

* multiplicative:
**yt = St x Tt x Et**

根據我對[參考資料](https://www.otexts.org/fpp/6/1)的理解，additive 特性的時間序列通常是代表有固定起伏、固定週期模式的線性時間序列。然而 multiplicative 則是相反，時間序列的起伏、週期會隨著時間不同成長幅度隨之變化，屬於非線性的時間序列，股票的時序資料多屬於此類。

利用以下 python 程式可以分解出 `seasonality`, `trend`, `residual(stationary)` 等時間序列不同部分。
```python=
from statsmodels.tsa.seasonal import seasonal_decompose

# Decompose time series
result = seasonal_decompose(etf_df['收盤價(元)'].values, 'multiplicative', freq=20)
result.plot()
plt.show()
```
![](https://i.imgur.com/QaF3ktm.png)

### time series data 相關性
時序資料也可以從過去幾筆間的資料來去看之間的相關性來做之後的為資料分析評估, 像是衡量`` Xt, Xt-1, Xt-2,...Xt-n ``間的關係，此性質叫 **autocorrelation**，表示時間序列資料與前筆資料的相關性程度。而最常用來檢視時間序列記錄關係性的功能就是 **ACF (autocorrelation function)** 了，用 ACF 可以來判斷這個時間序列資料是否有 stationarity 或 seasonality 的性質。


* ACF plot:
ACF 值介於 -1~1 間，越正相關越接近 1 ，反之越負相關則接近 -1，無相關則為 0，以下為 ACF 的示意圖，X 軸為 laged value，代表過去的時間記錄，Y 軸為 ACF 關係係數。這張圖表示該時間序列資料前 24 個 laged value 都高於 statistically significant 定值，有高度相關的關係，也因為有著高度相關的關係所以該時間序列為 non-stationary。

![](https://i.imgur.com/omPW45F.png)

了解了以上的時間序列資料特性後就可以做時間序列預測 (time series forecasting) 的任務了，一些著名的統計預測方法，像是 ARIMA 就是基於以上的時間序列特性的，而時間序列要如何預測詳細待下回分解。

### 程式處理技巧
由於自己常常在處理 IOT sensor 的時間序列資料，也記錄一下常用的技巧。
* time series data 區間間隔標準化
sensor 資料的收集往往都會遇到網路延遲的問題，所以難保每筆記錄的時間間隔都是一樣的，pandas 提供了方便的技巧可以處理這類問題，如下程式碼所示。
```python=
# regular time series interval to 1 min 且用最近的一筆記錄補值
train_series_regular = train_series.resample('1T').bfill()
# 間隔標準化後可能會產生缺值，因此在做內插
train_series_regular = train_series_regular.interpolate(method='time', limit_direction='both')
```

參考:
1. https://machinelearningmastery.com/feature-selection-time-series-forecasting-python/
2. http://www.dummies.com/programming/big-data/data-science/autocorrelation-plots-graphical-technique-for-statistical-data/
3. https://towardsdatascience.com/preprocessing-iot-data-linear-resampling-dde750910531
4. https://datascopeanalytics.com/blog/unevenly-spaced-time-series/
5. https://www.analyticsvidhya.com/blog/2016/02/time-series-forecasting-codes-python/
6. https://medium.com/open-machine-learning-course/open-machine-learning-course-topic-9-time-series-analysis-in-python-a270cb05e0b3
7. https://www.otexts.org/fpp/6/1
8. https://www.analyticsvidhya.com/blog/2018/02/time-series-forecasting-methods/
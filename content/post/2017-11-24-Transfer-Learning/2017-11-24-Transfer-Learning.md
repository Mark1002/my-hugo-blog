---
title: Transfer Learning 學習紀錄
date: 2017-11-24 22:32:56
comments: true
categories: deep learning
---
Transfer learning 是一個在機器學習領域的研究議題，大意是指將一個在某一種任務上學習 model 的知識可以轉移給其他的 model ，來去解不同但相近的任務，藉此來降低重新學習的成本。本人看到這種方法第一個聯想就好像是武俠小說中，一般人吃了25年功力的大還丹就變成絕世高手一樣.....

其中，Transfer learning 在深度學習的 CNN 更是用的火熱。因為深度學習的方法要訓練一個完好的模型所要花費的時間成本、資料量是非常貴的，以影像分類任務來說至少也要收集幾十萬張相關照片，來訓練個好幾天才會有好的結果，然而現實中根本不可能有那麼多大量的資料，因此在實務上深度學習的應用已很少完全重頭訓練模型，而是利用 transfer learning 來做 pre-training 任務。

Transfer learning 的應用上共有兩種：
1. Feature extractor
此應用將整個預訓練的 CNN 視為一個 feature extractor，去掉後面的 fully-connected layer，保留前面的捲積層部分當作固定的特徵萃取器，圖片經過捲積層部分的萃取過程後得到的新特徵叫做 ``CNN codes``，之後在接上一個線性的分類器來做分類。
2. Fine-tune
此方法除了也將預訓練 CNN 模型後面的 fully-connected layer 去掉以外，也對前面的一部分捲積層做權重的微調，其他捲積層則固定權重不變，進而重新訓練模型來更新權重，微調的網路層深淺視使用情境而定，一般而言較深的網路層會有較高層次抽象的特徵，離資料集本身的特性越接近，而淺的網路層則是比較泛化的特徵，像是線條、顏色等等。

而使用 Transfer learning 的情境，會受到兩個關鍵因素所影響，分別是**資料數量**與**資料相似性**，因此根據這兩個關鍵因素共可分成以下四種使用策略，如以下這張圖所示：
![](https://i.imgur.com/tk7QnUf.png)

1. **New dataset is large and similar to the original dataset**
首先是圖中第一象限的情況，資料量多而且性質非常相似，這可以說是最好的情況了，由於資料量很多，因此不怕有 overfitting ，可以採取 fine-tune 對一部分捲積層做權重的微調來重新訓練整個網路。
2. **New dataset is large and very different from the original dataset**
第二象限資料量大且很不同，這種情況其實就可以直接從無到有訓練自己的CNN了，但實際上使用預訓練的網路模型來當作初始的權重還是有助益的，因此這種情況的訓練策略為使用預訓練模型的權重當作初始權重，之後再重新訓練整個網路。
3. **New dataset is small and similar to original dataset**
第三象限由於資料量極少，若採取 fine-tune 方式重新訓練整個網路會有 overfitting 的問題，然而訓練資料與預訓練模型的訓練資料非常相近，因此我們預期深的網路層的抽象特徵與資料本身特性夠接近，採取的作法為 feature extractor，得到 CNN codes 後在其後面接上一個線性分類器，並只訓練這個線性分類器來分類。
4. **New dataset is small but very different from the original dataset**
最後第四象限，資料量不但少且與預訓練模型的訓練資料差異極大，這時候由於資料量少，因此不適合使用 fine-tune 重新訓練整個網路，而將預訓練模型當成 feature extractor 呢？這裏要注意的是由於資料性質差異大，因此就不能使用後面網路的深層特徵了，因為這些特徵只與預訓練模型的資料性質很接近，但與要訓練的新資料特性可是天差地遠了，所以要做 feature extractor 的話，要抽取網路層前面較淺且較泛化的 CNN codes 當特徵，後面在接上線性分類器來訓練。

除了以上兩種應用與四種情境以外，使用 Transfer learning 訓練需注意的另一點為要使用較小的 learning rate，這是因為預訓練模型卷積層的權重已有較好的調校，若使用過大的 learning rate 反而會把本來準確的權重給破壞了。

參考資料:
https://towardsdatascience.com/transfer-learning-using-keras-d804b2e04ef8
http://cs231n.github.io/transfer-learning/#tf

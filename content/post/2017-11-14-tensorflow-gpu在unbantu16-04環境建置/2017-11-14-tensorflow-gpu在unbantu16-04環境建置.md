---
title: tensorflow-gpu在unbantu16.04環境建置
date: 2017-11-14 21:46:09
comments: true
categories: tensorflow
---
這是我 tensorflow 在 unbantu 16.04 的安裝紀錄。

安裝套件版本：
tensorflow 1.3.0
cuda 8.0.61
cudnn 6.0 for cuda 8

從 [nvidia](https://developer.nvidia.com/cuda-toolkit-archive)下載 cuda 以及 cudnn 之後，在安裝前請重新開機進入 BIOS 設定把預設顯示卡功能調整成用內顯，並調整螢幕插頭位置到內顯插槽上，之後按下 `Ctrl-Alt-F1` 進入 tty1 介面，並輸入以下指令
```
sudo service lightdm stop
```
根據[這篇](http://city.shaform.com/blog/2016/10/31/install-tensorflow-with-cuda.html)的說法這條指令的作用是用來達到顯示卡與內顯區別的目的，如果不打這條指令就裝 cuda 的話，重新開機後 unbantu 會畫面全黑或是卡在系統 loading 畫面。
(ps:本人就是這樣才重新裝第二次!)

之後就可以開始安裝 cuda 了，輸入以下安裝指令來安裝。這裡有一點要注意的是要指定 `cuda=version` 來安裝，不指定的話預設裝上的是 cuda9
```
$ sudo dpkg --install cuda-repo-ubantu1604_8.0.61-1_amd64.deb
$ sudo apt-get update
$ sudo apt-get install cuda=8.0.61-1
```
裝完後到 `.bashrc` 貼上以下環境變數路徑，之後記得先重開機看看能不能順利入系統
```
export PATH=$PATH:/usr/local/cuda-8.0/bin
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64
```
接著開始來安裝 cudnn，輸入以下指令
```
$ tar -xvzf cudnn-8.0-linux-x64-v6.0.tgz
$ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
$ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
$ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```
以上都裝好的話可以打以下指令來 check 顯卡是否有裝好
```
$ nvidia-smi
```

之後就是裝 tensorflow 了，基本上都跟[官網](https://www.tensorflow.org/install/install_linux)一樣，沒什麼不同。
本人採用的是 Anaconda 的安裝環境，創一個虛擬的環境後打以下指令

```
$ pip install https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.3.0-cp35-cp35m-linux_x86_64.whl
```

到這邊就差不多裝好囉！之後就可以在裝 keras, pandas, numpy 等等其他資料分析常用的套件啦！

參考資料
http://city.shaform.com/blog/2016/10/31/install-tensorflow-with-cuda.html
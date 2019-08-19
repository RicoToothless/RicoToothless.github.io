+++
draft= true
author = "Rico"
title = "COSCUP 2019"
date = "2019-8-19"
description = ""
tags = [
    "COSCUP",
    "CNTUG",
]
categories = [
    "event",
]
archives = "2019-8-19"
+++

# 活動開始前

在搶票之前我就先花了 2500 元買了"[算我一票] COSCUP 個人贊助票"，後來又發現有賣"2019！開源久酒！"的活動票，能跟大家聚聚何樂不為呢？

因為"[算我一票] COSCUP 個人贊助票"已經包含 COSCUP 入場的資格了，所以真是辛苦那些搶票的人。

# 2019！開源久酒！

是在 COSCUP 前一天的星期五夜晚小酌聚會，離我的公司蠻近的，走路就到。位於 4 樓頂樓，周圍都被高聳又明亮的建築包圍著，導致這是個特別的夜晚。

搭配著有點節奏音樂一起聊著最近工作上的趣事，而且也認識了厲害的新朋友，我們這一桌吃了不少零食和喝了不少啤酒（真的不少）。

# COSCUP 第一天

我選的議程幾乎是 Cloud Native 社群舉辦的，只能說其他議程真的離我的工作範疇真的太遠了，尤其是 Golang 的議題是兩天都有，可惜不會寫 Go ，不然這次的 COSCUP 應該會收穫很多。

因為時間的關係，基本上在 COSCUP 裡面所有的議程都只能帶到皮毛，幸好講師基本上都會把更多的參考連結附上。

### 議程一：和我一起用全裸容器網路改變世界
[共筆連結](https://hackmd.io/DqGFkz-SSoKIoSFObo1UUA?both)
[講者blog](https://blog.pichuang.com.tw/)
[Slide Link](https://speakerdeck.com/pichuang/20190817-container-bare-metal-for-networking)
[Multus-CNI](https://github.com/intel/multus-cni)

簡單介紹一下 Kubernetes Networking 、 CNI 和 CNI Plugin 。

主要講述 Kubernetes 目前主流的 sulotion 不外乎就是一個 pod 一個 network interface 。

除了這個之外，也是有大型公司有 VNF (Virtual Network Function) & CNF (Cloud native Network Function) 想要混合著用的需求。

所以就來介紹一下這個 Multus-cni 這個專案，讓一個 pod 可以有兩個 network interface ，可以一次跟兩個不同的 CNI 串接。

* 實體網路和虛擬網路
如果是要跟 Bare Metal Network Device 對接的話，可以使用參照 SR-IOV 標準的 CNI Plugin 對接。

又或者是用 Open vSwitch 可以使用 DPDK CNI Plugin。

### 議程二：Web Browser Extension Workshop (Group 2)
[共筆連結](https://hackmd.io/hHklSJ7oSAOZX_Fwke9zJQ?both)
[PPT url](https://goo.gl/hgoiVp)
[講者github](goo.gl/M2DUsg)

是真的蠻有趣的，而且教材設計不錯讓人很有成就感，稍嫌可惜的地方就是有些步驟在 PPT 上面沒有特別截圖出來。



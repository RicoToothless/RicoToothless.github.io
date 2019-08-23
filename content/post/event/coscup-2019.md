+++
draft = false
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
archives = "2019-08"
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

議程真的是很多，所以我只撰寫我有聽到的部分。

### 議程一：和我一起用全裸容器網路改變世界
* [共筆連結](https://hackmd.io/DqGFkz-SSoKIoSFObo1UUA?both)
* [講者blog](https://blog.pichuang.com.tw/)
* [Slide Link](https://speakerdeck.com/pichuang/20190817-container-bare-metal-for-networking)
* [Multus-CNI](https://github.com/intel/multus-cni)

簡單介紹一下 Kubernetes Networking 、 CNI 和 CNI Plugin 。

主要講述 Kubernetes 目前主流的 sulotion 不外乎就是一個 pod 一個 network interface 。

除了這個之外，也是有大型公司有 VNF (Virtual Network Function) & CNF (Cloud native Network Function) 想要混合著用的需求。

所以就來介紹一下這個 Multus-cni 這個專案，讓一個 pod 可以有兩個 network interface ，可以一次跟兩個不同的 CNI 串接。

#### 實體網路和虛擬網路
如果是要跟 Bare Metal Network Device 對接的話，可以使用參照 SR-IOV 標準的 CNI Plugin 對接。

又或者是用 Open vSwitch 可以使用 DPDK CNI Plugin。

### 議程二：Web Browser Extension Workshop (Group 2)
* [共筆連結](https://hackmd.io/hHklSJ7oSAOZX_Fwke9zJQ?both)
* [PPT url](https://goo.gl/hgoiVp)
* [講者github](goo.gl/M2DUsg)

是真的蠻有趣的，而且教材設計不錯讓人很有成就感，稍嫌可惜的地方就是有些步驟在 PPT 上面沒有特別截圖出來。

了解一些 Chrome & Firefox 的小差異，發現 Firefox 有些東西是相容於 Chrome 的，不過我最近都用 [Brave](https://brave.com/) 這個 Browser 。

其實比起開發的部分，最有感的部分在於講解 Permission 的部分，套件哪些權限可以對 Browser 使用特定的 API ，套件哪些權限是對 Host 的（可惜我對 Host Permission 還沒有很深刻的認識）。簡單講就是之後在安裝套件的時候可以看一下 Permission 是不是有點過大，覺得有疑慮再去衡量一下風險要不要使用。

### 議程三：HA Prometheus solution - Thanos
* [共筆連結](https://hackmd.io/0s_GosawScOF1q9RKcjTaw?both)
* [Thanos](https://thanos.io/)

主要在於 Prometheus 在 HA mode 下為了資料的一致性做了效能上的犧牲，所以才有 Thanos 這樣的工具出現，也支援不同的 storage providers 。

有很多功能像是壓縮舊資料、更大時間軸的 query 來加速 query 和兼具 StoreAPI 的 sidecar 。

### 議程四：Quick and Solid - Baremetal on OpenStack
* [共筆連結](https://hackmd.io/@coscup/ByOKk7eVS)

講師的名字跟我一樣是 Rico ，所以特別記得。

只能說在座的各位對 Openstack 比較陌生一點，之前在資策會學過的也忘了差不多了，幸好之前有先聽講師 Rico 在 Cloud Native Meetup 講過一次，有稍微了解到近期 Openstack 的趨勢。

Openstack 跟 Kubernetes 是會常常被拿來做比較的 orchestration system ，但其實兩者之間是可以一起使用與互補，畢竟 Kubernetes 最大的罩門還是在於多租戶的管理。

還有近期的 Openstack 不要求把所有的 Component 全部裝完，反而依照需求安裝幾個就夠了。而且 deploy 的工具也可以使用十分方便的 Helm ， Openstack & Kubernetes 一次滿足。

### 議程五：Rancher Pipeline 使用經驗分享
* [共筆連結](https://hackmd.io/@coscup/HJsYy7eVH)
* [PPT](https://drive.google.com/file/d/1FyuAaI4BsNAVBr29lIf25BzbE2WqrZ-a/view)

Rancher 可說是開箱即用的工具，他們直接使用 RKE（ Rancher 自己的 kubernetes ）。而使用 Rancher 本身的 pipeline 是因為原生且該有的都有之外， Rancher UI 做得不錯。

改善之後的流程圖可以明顯看到， git push 之後到 Rancher pipeline 會 trigger 到三個不同的地放做事，但能夠在 Rancher 裡面看到可以做到集中化的管理。

### 議程六：Experience on gRPC rate limiting with Istio
* [共筆連結](https://hackmd.io/@coscup/rJJc17gNr)
* [PPT](https://speakerdeck.com/miyachen/experience-on-grpc-rate-limiting-with-istio)

最近也在摸索 Service Mesh 這一塊，所以還無法領悟到其中的精髓。

這次探討的是 rate limiting ，能夠 config 的地方有兩個 part ，分別是 Mixer & Client ，並且詳細介紹到可以實際操作的程度了。

以後有機會使用 Istio 我應該會再回來好好細讀一番。（雖然我還在想要用 Consul or Istio ）

### 議程七：如何在两年内从初学者成长为流行开源项目维护者和技术书作者？
* [共筆連結](https://hackmd.io/@coscup/SJ6B0MxNS)
* [PPT](http://greyli.com/slides/coscup2019/#/)

比較偏向勵志的議程，講解真的十分幽默之外台風也很足，現場人非常非常多，給大家的新手任務也很接地氣，不知道走出這扇門後有多少人完成新手任務了呢？

**新手任務**
* 在部落格寫 100 篇文章
* 在 Stack Overflow 上寫 50 個回答
* 在 GitHub 上創建 50 個 PR

### 議程八：Service Mesh and Observability around it
* [共筆連結](https://hackmd.io/@coscup/rJrqJXg4r)
* [tetrate](https://www.tetrate.io)
* [skywalking](https://skywalking.apache.org/)

又回到 Cloud Native 的教室了，這次的議程也是 Service Mesh 相關的主題，而且也是使用 Istio 再作延伸。

有帶到 Service Mesh 比起傳統在 OSI Layer 3 對 service 做隔離的管理（ DNS 或者防火牆等等），Service Mesh 是從 OSI Layer 7 方面做管理，可以更貼近開發人員和更為聰明的管理方式。

主要談到 Istio Mixer 有效能上的堪憂，所以講者提供的 solution 就是把 Mixer 拿掉，之後再靠什麼東西補足這部分我還真有點忘了。

### BoF (Birds of a feather) 同樂會
* [BoF表單](https://docs.google.com/document/d/e/2PACX-1vRjNzk6wT2-UJCILujDX_9eq4C_jUfxIDxsAwzI_km_n2GdiKzAg9y6l7gyIzEvZL_1yzZMSGI8HIXz/pub)

第一天 COSCUP 結束之後參加了 Cloud Native 的 BoF ，只能說大家都很 high ，玩個兩個小遊戲考驗大家的技術到哪裡，每個人都有他厲害的地方，真的大開眼界。

然後表單裡面某個聚會居然高達 100 多人！

---

# COSCUP 第二天

因為台科大供電系統有問題，早上緊急轉移到台科大邊緣沒有受到影響的「研揚大樓」，還完 Ubike 走過去真的好遠啊。

### 議程一：網域名稱、DNS 技術解析 BoF
* [共筆連結](https://hackmd.io/@coscup/rJvQkQg4B)

講述了蠻多大公司競爭頂級域名的一些故事，討論的範圍也非常的廣，我這邊把 COSCUP 對於這個議程的介紹就直接複製貼上了。

**討論議題(包含但不限於)：網域名稱、DNS 技術、DNS 紀錄、DNSSEC、常見 DNS 操作(所有權人/管理人/技術聯絡人/帳務聯絡人)、域名移轉、註冊商與經銷商、域名爭議、域名投資、中文域名。**

另外也有不願具名的資深工程師分享在中國管理 DNS 的故事，曾經被要求要在一個小時內把所有跟特定公司相關的 DNS record 全部清除。

### 議程二：Wikidata x OpenStreetMap 新手編輯工具箱
* [共筆連結](https://hackmd.io/tuED2BT5SE6FKVfjjym6ww?both)
* [PPT](https://www.slideshare.net/planetoid/wikidata-164489690)
* [Markdown](https://hackmd.io/X9vwu6UpS8q8teWPgUb_Eg?view)

真的是很有心的講者，可以感受到他想要讓大家多多貢獻，只要回去翻他的教材都可以輕易上手，甚至都直接點出新手可能要注意的點和難以跨過的關卡是什麼。

可惜對這一塊我真的沒什麼熱忱啊。

### 議程三：開源教育 BoF
* [共筆連結](https://hackmd.io/@coscup/HklVkXeES)

講了許多教育界在推廣開源軟體或硬體上的心酸血淚，也稍微了解到在這個議題裡真的有很多面向值得探討。

### 議程四：自己的鍵盤自己做 - Ergodox 教學
* [共筆連結](https://hackmd.io/@coscup/rkoMxmeEH)

歡迎加入他們的 fb 社團「鍵人谷」，只能說會軟體又會硬體的人很厲害，而且還有很酷炫的燈呢！

如果需要長時間的 coding 或許真的很需要這種鍵盤，來延長工程師的生涯。

### 議程五：打造古早味的輝光管時鐘
* [共筆連結](https://hackmd.io/QcExaSlpRJ-gNPN_0z1pBg?both)
* [作者blog](https://yodalee.blogspot.com/2018/10/nixie-clock0-introduction.html)

這個是我這兩天 COSCUP 看到最新奇的議程了，認識到了 Nixie Tube 是什麼東西，稍微複習了一下電學，也講解了要玩 Nixie Tube 真的是門檻相對高的娛樂。

像是 Nixie Tube 難以入手且價格昂貴，電壓要自己調到 180V ，聽到這邊我覺得還是在軟體界 build 東西就好了，至少不會電到自己。

結束之後幾乎全班跑去講台前面看 Nixie Tube ，真是熱烈。

### 議程六：網站開發框架的正確學習姿勢
* [共筆連結](https://hackmd.io/@coscup/SJPiCfxVB)

新手要怎麼選擇語言？

為何要學習 framework ？

新手常卡關的點。

其實蠻受用的，畢竟很多時候新手就只差最關鍵的開關有沒有被打開，一旦打開了之後，他就會自己去尋找答案了。

最後因為講師是 JetBrains 傳教士，所以拿了不少 JetBrains 的貼紙，覺得開心。

### 議程七：Lightning Talks
* [共筆連結](https://hackmd.io/BDbWDlCrRoCIGdXgA1DfkQ?both)

每一個 talk 真的都很有特色，每一個都不冷場，不知道我上去講會不會一樣吸引大家的目光呢？

最令我印象深刻的就是在台上敲鍵盤的講者了，因為一直打錯字導致娛樂性十足。

# 結語

這期間也晃了很多攤位，發現開源社群真的蓬勃發展，拿了很多貼紙和贈品真的很開心。（但卻被室友嫌棄是垃圾）

希望明年能夠參與更多。

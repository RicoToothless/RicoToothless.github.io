+++
author = "Rico"
title = "ECS migrate to EKS part 2"
date = "2020-02-09"
description = "How to plan the migration plan?"
tags = [
    "ecs",
    "eks",
    "docker",
    "kubernetes",
    "cdk",
    "aws",
]
categories = [
    "tech",
]
archives = "2020-02"
+++

# Software Development Life Cycle (SDLC)

這一次的 Migration 基本上可以套用 SDLC 的概念，但是迭代更快，為了增加效率決定把 task 細分，每個 task 都跑一次 SDLC 。其實 SDLC 每個階段的劃分大家都不太一樣，而我的 SDLC 的流程大致如下：

1. Project Initiation
2. Requirements Gathering
3. Analysis
4. Design
5. Development
6. Testing
7. Deployment
8. Maintenance

# Project Initiation

在這裡我們需要站在高處上考慮到整體，什麼問題需要解決、手中資源有多少、時程、里程碑、商業價值等等。

[上一篇文章]({{< relref "/post/tech/ecs-to-eks-1" >}})，因為公司內部日趨複雜的專案越來越多，原本簡單的維運方式已經無法負荷，所以從 ECS migrate to EKS 。

# Requirements Gathering

原本在 SDLC 裡 Requirements Gathering 是幫助定義是否有這需求，不過呢，基本上原本在 ECS 的優點都是必須的，幸運的是這些優點都可以在 EKS 上實現，我們能做的就是排定優先順序完成。

# Analysis

1. 分析是否可以把整個 Project 切成小的 task 。
2. 分析每個 task 若要達成的話需要哪些要求，例如資料要先備份，有依賴性的服務要先把依賴關係搞清楚。
3. 分析失敗的風險是要可以控制且可以預測的，基本上只要資料有備份好，使用 container 切換很快所以還可以。
4. 分析這個 Project 是否可以對不同職位的人都容易上手，我個人的想法是呈現給他們的都是最簡單的一面，實際使用的時候不用知道太多背後的原理。

# Design

因為每一個 task 實際怎麼達成目標的方法都不太一樣，以下幾點是 Design 的核心原則：

1. 要做出好的 Design 必須深入了解所有技術的細節，你的 application 能力、工具的能力、團隊能力等等。（雖然整個 migrate 過程只有我一個人就是了）
2. 所有的 Design 計畫必須要給有相關的人員檢視過。
3. 若相關人員給出建議之後再整合到 Design 計畫裡。

在真正開始 migrate 之前，必須要先了解工具現有的 migrate solution ，像是資料庫 migrate 和 credential migrate ，這些工具都會有 migrate 的功能，不只了解還得實際演練一次，畢竟任何資料都要小心謹慎處理。

# Development

接下來就是真正要實作的部分，一樣有幾個核心原則：

1. 先測試環境之後正式環境。
2. 先無狀態 application 之後有狀態 application 。
3. 先簡單之後困難
   
第一點應該沒什麼需要補充的，而第二點其實跟第三點的概念十分相似，自己本身雖然有學過 Kubernetes 但畢竟實際實踐是兩回事，所以本質上是從頭開始學起，所以先站穩腳步再提及跑步是妥當的做法，再加上有些工具都要跟著換會更好，例如 CICD 的工具 [Kubernetes plugin for Jenkins](https://github.com/jenkinsci/kubernetes-plugin)。

# Testing

Testing 是非常重要的一環，必須要把功能各方面都保護好，除了要測是否有 bug 也要測是不是完全符合需求。基本上測試的人也是我，所以沒有特別跑 [Bug lifecycle](http://tryqa.com/what-is-a-defect-life-cycle/)，因為是我自己跑那個 lifecycle 。 Testing 核心原則如下：

1. 不要有任何推測、假想、幻想某些看似基本或微小的功能是正常的。
2. 測試發現跟預期不一樣也不能對功能有妥協，功能當初設計要 80 分，驗收的時候就該是 80 分以上 。
3. 如果真的需要妥協，像是當初 Design 就有問題，就 feedback 並回到 Design 的階段。

# Deployment

Deployment 的原則和 Development 差不多，每個 task 重要程度不一，所以 Deployment 也要看花多少資源準備。當我測完之後，就會開始 migrate ，不論是哪種 task ，在這之前都要準備好備份。當然還有一點就是要稍微選擇一下停機時間，盡量不影響他人是美德。

# Maintenance

接下來就是準備好接受真實 user 的 feedback 了，不論 user 是客戶或是坐在隔壁的開發人員都必須好好面對，畢竟他們給的 feedback 才是最有價值處理的事情。

# Conclusion

下一篇會講更多實踐的技術細節，這邊先做一些總結：

1. Project Initiation 要確認 migrate 價值。
2. Requirements Gathering 要確認是否有開啟 migrate 的需求。
3. Analysis 要分析我們能做到什麼程度，風險是什麼。
4. Design 要深入了解技術後再 Design ，且必須持續整合其他人的 feedback 。
5. Development 要扎扎實實且一步一步的完成。
6. Testing 要務實、堅守立場最後真的有疑慮再做 feedback 。
7. Deployment 要隨時有 B 計畫，並且盡量讓服務不停機。
8. Maintenance 要廣納接受所有 feedback 。
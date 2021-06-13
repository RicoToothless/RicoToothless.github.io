+++
author = "Rico"
title = "Jenkins Configuration as Code (JCasC)"
date = "2021-06-13"
description = "Make Jenkins configuration more consist"
tags = [
    "jenkins",
    "config",
    "jcasc",
    "job-dsl",
]
categories = [
    "tech",
]
archives = "2021-06"
+++

## Why we need Jenkins Configuration as Code?

你曾經是否有在 Jenkins GUI 上設定不太記得上一次自己設定了什麼，或者當初是誰設定的也不清楚，又或者不小心更動設定自己卻渾然不知呢？

[Jenkins Configuration as Code (a.k.a. JCasC) Plugin](https://github.com/jenkinsci/configuration-as-code-plugin) 基本上就是解決上述的情境，把 GUI 上的設定轉化為 yaml key-value 的形式，目前大部分的設定都有支援，但還是有些冷門的 plugin 設定 JCasC 沒有支援，可以參考 [JCasC support dashboard](https://issues.jenkins.io/secure/Dashboard.jspa?selectPageId=18341)。

如果每一次的設定都從 source control management（現今大部分是以 git 為主）出發，再設定部署到 Jenkins 之前就可以走一般的開發流程：

1. Jenkins 管理人員想要更改設定而提出 Pull Request
2. 提出 Pull Request 後會做 CI 檢查，像是 lint 或 test
3. 讓其他 Jenkins 管理人員做 code review
4. Merged 之後會再做一次設定的 CI 檢查後就 CD 到 Jenkins 裡

這樣的好處可以減少人為疏失、增加設定時的信心、設定內容的一致性、有效率的開發週期以及最重要的是減少維護的成本。

## JCasC without magic

JCasC 有完整個文件和很多的範例可以參考之外，最實際的方法就是可以直接把目前的設定匯出 JCasC yaml 格式，詳請參考 [Exporting configurations](https://github.com/jenkinsci/configuration-as-code-plugin/blob/master/docs/features/configExport.md)，雖然文件裡面有說匯出的設定不完全都可以使用，但我個人的心得是 95% 以上的設定直接複製下來使用都沒問題，當然還是得看 Jenkins 裝了多少 plugin 而定，雖然冷門的 plugin 還是會把設定匯出來，但實際寫進 JCasC 部署之後就會噴不支援的 error。

也就是說，假如不知道怎麼寫 JCasC 的話，可以先在測試環境的 Jenkins 用熟悉的 GUI 設定一遍，再把設定匯出來複製貼上即可。

不過 JCasC 也是有缺點的，和 Infrastructure as Code（IaC）一樣有實際情況和紀錄的狀態不一致的問題。一般來說，IaC 部署之後都會把 Infrastructure 目前有的 resources 紀錄起來，但假如有人意外的有權限刪除某個 resource 的話，當初儲存的紀錄並不會知道有 resource 被刪除掉了。目前市面上的 IaC solution 並沒有像 Kubernetes 一樣的有 controller 的機制時時刻刻監看著實際情況和紀錄的狀態。

同理，JCasC 也是一樣，**正式使用 JCasC 之後就不應該再使用 GUI 來做設定**。

## JCasC review

目前對 JCasC 的使用心得是，在規劃大量 user permission 很好用，我們使用的是 [Role-based Authorization Strategy](https://plugins.jenkins.io/role-strategy/)，如果是按照平常 GUI 的方式做設定會因為人數增加而增加維護成本。

如果是既有的 Jenkins 導入 JCasC 幾乎沒有痛點，因為使用 JCasC 跟在 GUI 上設定的本質是一模一樣的，所以 JCasC 可以覆蓋舊有的設定，也可以跟其他還沒有使用 JCasC 的設定一起共存。另外，即使 JCasC 設定有錯也不會導致 crash，頂多只會有大量的 error log。

JCasC 也有支援 [Jenkins Job DSL](https://plugins.jenkins.io/job-dsl/)，因為 Job DSL 也需要另外再寫一篇做深入的探討，簡單來說就是把所有的 Jenkins job 以程式碼的方式做定義，跟本篇想要減少維護成本的中心思想一樣。

但本篇文章還是先得稍微介紹一下 JCasC & Job DSL 一起使用的用法。總體來說，我們當然可以把所有 Job DSL 全部寫在 JCasC 裡，但比較推薦的做法是使用 JCasC 把 Job DSL 需要的 seed job 建立出來，再靠 seed job 去部署剩下的 Job DSL。

使用官方 Jenkins helm chart 的話，預設是 helm 有更新 JCasC 的部分就會靠 side car container 做 [auto reload](https://github.com/jenkinsci/helm-charts/tree/main/charts/jenkins#config-as-code-with-or-without-auto-reload)，Jenkins controller （舊名：Jenkins master）本身就可以不用重啟 pod 來更新 JCasC 裡的設定，提升 JCasC 的使用體驗。

## Conclusion

* 使用 JCasC 減少人為疏失、增加設定時的信心、設定內容的一致性、有效率的開發週期以及減少維護的成本
* JCasC 可以直接參考匯出的 yaml 設定
* 使用 JCasC 之後不應該再使用 GUI 做設定
* JCasC 可以無痛導入在既有的 Jenkins
* Jenkins helm chart 可以讓 JCasC 有更好的使用體驗
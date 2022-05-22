+++
author = "Rico"
title = "Cosmos SDK chains monitoring"
date = "2022-05-21"
description = "How to monitor our different Cosmos SDK chains"
tags = [
    "cosmos",
    "monitor",
    "prometheus",
    "alertmanager",
    "grafana",
]
categories = [
    "tech",
]
archives = "2022-05"
+++

## What is Cosmos SDK?

SDK 是 Software Development Kit 的縮寫，中文稱「軟體開發套件」，而 Cosmos SDK 這個開發套件可以讓工程師客製自己想要的區塊鏈，以 Cosmos SDK 開發的區塊鏈著名的有 Cosmos Hub、Binance Chain、Terra、IRIS、Osmosis 和 Juno 等等。

說了這麼多要解決的是塞車問題，就算 PoS 已經比 PoW 的鏈快上很多了，但依舊難解整個鏈塞車的問題，甚至惡意的 request 造成整個鏈無法使用影響所有的使用者，最近的趨勢越來越朝向多鏈發展，優缺點都很值得探討，但這裡不會著墨太多，本篇將會專注在技術層面。

## Cosmos SDK chain's architecture

Cosmos SDK 創建出來的區塊鏈雖說可以客製化到非常獨特，每一條鏈可以差異到非常大，但是大體上的架構如下：

![cosmos-chain-architecture](/cosmos-chain-architecture.png)

最底層為 Tendermint 共識層，其功能確保每一個 node 都同意每一筆 transaction，之後才寫進自己的 local state。

再來是 Cosmos SDK 內建的功能，像是最常用的 staking、account、bank、governance 和 IBC 等等。

最後是開發人員自己需要客製化的部分，例如 DeFi、ICO、GameFi、NFT marketplace 或 DAO 等等。

## How to monitor Cosmos SDK chain?

身為 validator operator 除了基本的硬體資訊和程式狀態會看以外，也會搜集 Cosmos SDK 架設的鏈裡面的 metrics，因為硬體正常或程式有在跑不代表程式邏輯本身是否預期的。因為 Cosmos SDK 採用了 Prometheus solution，所以整合非常容易，要看硬體和程式狀態的話，就可以直接跟常見的 [node exporter](https://github.com/prometheus/node_exporter) & [process exporter](https://github.com/ncabatoff/process-exporter) 一起並用。

Cosmos SDK 架設的鏈裡面的 metrics 其實剛好也分成 Tendermint、Cosmos SDK 和客製化功能這三層，設定上來說 Tendermint 和 Cosmos SDK 這兩層不管在哪個鏈都一模一樣，而且內建在設定檔裡，只有客製化功能可能每個鏈會有所差異，可以參考 Agoric 的[設定文件](https://github.com/Agoric/agoric-sdk/blob/master/packages/cosmic-swingset/README-telemetry.md)。

實務上，我們這三層的 metrics 都會收集，不過只有 Tendermint metrics 才會設定 alert rules，這層的 metrics 有 block height metrics 確保自己的 sentry or validator node 是有正常 sync 的，另外也有 validator metrics 確保自己有正常產出 block，確認 node 的 p2p peers 數量是否是預期的，尤其是 sentry node 的架構下，sentry 和 validator 之間的 peers 數量是不是預期的。而 Tendermint 有哪些 metrics 可以參考 Tendermint [官方文件](https://docs.tendermint.com/v0.35/nodes/metrics.html#list-of-available-metrics)。

Cosmos SDK metrics 反而比較適合拿來做網路上的分析或是用 dashboard 的方式呈現，可以參考 [Cosmos SDK metrics list](https://docs.cosmos.network/master/core/telemetry.html#supported-metrics)。

客製化功能 metrics 就真的因鏈而異了，有些鏈會實作 Prometheus metrics，但有些不會，得自己寫個 exporter 呼叫 RPC 做判斷。

## How to monitor remotely?

理想上，我們還是希望監控在不碰到機器的情況下得知 validator 是否正常運作，其原理跟網頁上看到的 scanner 差不多，直接呼叫 RPC 取得鏈上資訊後顯示在網頁上，例如：[MINTSCAN](https://www.mintscan.io/cosmos) 或 [BIG DIPPER](https://cosmos.bigdipper.live/)，所以從頭到尾這些網站都沒有碰到機器就可以知道所有 validators 有沒有在運作。

換句話說，我們其實可以運用 Prometheus exporter 做到類似的事情，幸運的是，社群裡已經有現成的 [solarlabsteam/cosmos-exporter](https://github.com/solarlabsteam/cosmos-exporter)（關鍵字直接輸入 cosmos-exporter 會有很多其他的結果），這個 exporter 基本上適用於所有以 Cosmos SDK 架起來的鏈，只有少部分客製化很高的鏈會讓某些 metrics 無法使用，可以自己 fork repository 然後拿掉或改動程式碼即可。這個 exporter 可以監控整個網路的 validators 或者只要監控特定幾個 wallet or validator addresses。

因為只需要有 RPC 就可以使用就不侷限一定要使用自己的 node，你可以使用別人的 RPC 或者另外自己架設一個 node 專門給 exporter 呼叫讓架構更乾淨。另外需要注意的一點是 `solarlabsteam/cosmos-exporter` 需要 gRPC，外面免費的 RPC 還是以 REST API 為主。

不過 `solarlabsteam/cosmos-exporter` 只有支援到 Tendermint & Cosmos SDK 這兩層而已，如果要支援客製化功能就得另外使用其他工具或者自己寫。

## Conclusion

* Cosmos SDK 可以快速開發出一條完整的區塊鏈，也可以快速站在巨人的肩膀上，例如本文章的主題，監控
* 監控 Cosmos SDK 架設的鏈可以分為 Tendermint 共識、Cosmos SDK 內建的功能、客製化功能這三層
* 收集 metrics 可分為兩種，直接從 node 上取得，或者遠端遙測

## Reference

* [Understanding the value proposition of Cosmos](https://blog.cosmos.network/understanding-the-value-proposition-of-cosmos-ecaef63350d)
* [Tendermint metrics document](https://docs.tendermint.com/v0.35/nodes/metrics.html#list-of-available-metrics)
* [Cosmos SDK metrics documentation](https://docs.cosmos.network/master/core/telemetry.html#supported-metrics)
* [Cosmic SwingSet Telemetry](https://github.com/Agoric/agoric-sdk/blob/master/packages/cosmic-swingset/README-telemetry.md)
* [solarlabsteam/cosmos-exporter](https://github.com/solarlabsteam/cosmos-exporter)

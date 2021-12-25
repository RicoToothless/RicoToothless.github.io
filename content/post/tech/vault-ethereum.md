+++
author = "Rico"
title = "Vault Ethereum Plugin"
date = "2021-12-24"
description = "The Hashicorp Vault plugin for Ethereum wallet"
tags = [
    "hashicorp",
    "vault",
    "ethereum",
    "container",
    "kubernetes",
]
categories = [
    "tech",
]
archives = "2021-12"
+++

![vault-ethereum-opening](/vault-ethereum-opening.png)

## What is Hashicorp Vault?

在介紹最核心的 plugin 之前還是要講一下這個工具的背景，由 [Hashicorp](https://www.hashicorp.com/) 公司開發的產品 [Vault](https://www.vaultproject.io/)，Vault 專門管理任何敏感的資料，而且用法非常非常多元。從常見的儲存 key value 的敏感資料外，動態生產臨時用的敏感資料也算，例如，生產臨時的 AWS IAM、資料庫 username & password、生產 TLS 給 cert-manager 等等，除了內建的使用方法外，Vault 也支援 [plugin 系統](https://www.vaultproject.io/docs/internals/plugins)，使用者可以寫自己的邏輯在裡面。

Vault plugin 跟 Vault 本身是兩支不同且獨立的程式，為了資安考量，plugin 並沒有跟 Vault 使用相同的記憶體，兩支程式溝通是靠 TLS 加密過，如上所述，plugin 壞了也不會影響 Vault 本身的運作。

## Why Vault Ethereum plugin?

Vault Ethereum plugin 作者為美國 Washington, D.C. 和 Baltimore 地區的 HashiCorp User Group 創辦人 Jeff Ploughman 在[這篇文章](https://www.hashicorp.com/blog/using-vault-to-build-an-ethereum-wallet)認為：

雖然使用錢包時是基於密碼學之上，而 public key 可以當作分散式的概念分享給其他人，但 private key 本質上是中心化的概念，因為此概念跟區塊鏈分散式的思想背道而馳，讓作者常常不經思考錢包並不是跟區塊鏈的概念一起設計的，反而像是區塊鏈另外加裝一個錢包功能的 sidecar（白話一點就是作者覺得錢包設計沒有很優秀），這也讓很多大型企業在管理錢包的時候傷透腦筋。

所以作者倚賴 Hashicorp Vault，因為此工具已經提供很多安全的機制，例如文章裡面示範了 Vault 本身的 seal 機制、application 或是使用者身份驗證、MFA 和支援 HSM（需要 Vault Enterprise 才有）。

文章也提到可以拿來串 Github MFA authentication 做 CICD pipeline 來部署 smart contract，當然這還是得看 gas price 現在行情如何。

作者也有受邀到 2018 年的 [HachiCon 演講](https://youtu.be/nIez8LXtefY)，他最後有示範 cold storage 加上 MFA 送 20 美金給觀眾。

## Vault Ethereum plugin usage

[Vault-Ethereum Plugin](https://github.com/immutability-io/vault-ethereum) 專案的 README 已經有個快速的 demo 展示其主要功能，簡單來說他會在 docker-compose 裡創建兩個服務，一個是 vault 一個是 ganache（ganache 是用來個人測試的 Ethereum node，很多測試的功能會預先替使用者準備好）。

![vault-ethereum-architecture](/vault-ethereum-architecture.png)

作者建議把專案放在 `GOPATH` 下，但其實如果只是純粹用 docker-compose 的話其實可以不用。一開始會自己客製 container image 後啟動 docker-compose 環境。

```
$ cd $GOPATH/src/github.com/immutability-io/vault-ethereum
$ make docker-build
$ make run
```

打開另外一個 console 或 terminal 後到專案的 `docker/` 目錄下執行 vault 的身份驗證，之後我們就可以對 Vault 打出一系列的請求了。

```
$ cd $GOPATH/src/github.com/immutability-io/vault-ethereum/docker

# Authenticate
$ source ./local-test.sh auth
$ ./demo.sh > README.md
```

全部的請求有：

1. 以 mnemonic 匯入 account
2. 直接創新的 account
3. 查詢 ETH 餘額
4. 轉錢
5. 簽名
6. 部署 smart contract
7. 查看 smart contract 的 token 餘額

跑完指令後就可以看到執行的結果了，可以從 [docker/README-demo.md](https://github.com/immutability-io/vault-ethereum/blob/master/docker/README-demo.md) 參考他要請求的範例。

---

我實際安裝的時候發現一些情況跟大家分享一下。

先前提到 Vault plugin 架構是 plugin 這個程式是完全跟 Vault 主程式分開，所以 plugin 得靠 `api_addr` 這個設定去互動，其實有點綁手綁腳，所以在設計的時候要注意。

另外 Vault plugin 不比 Terraform plugin 生態系來得豐富，所以網路上的資源相對少蠻多的。

還有 Vault plugin error message 蠻難判斷到底發生什麼問題，例如 plugin binary 檔案沒有執行權限的話只會噴很摸不著頭緒的 error。

```
Unrecognized remote plugin message: 

This usually means that the plugin is either invalid or simply
needs to be recompiled to support the latest protocol.
```

還有需要注意的是，Vault Ethereum plugin 沒有釋出 Alpine Linux binary，所以無法透過 Vault 官方 image 直接下載 plugin binary，要自己客製 container image。

## Conclusion

* 管理敏感資訊一直都很難，到哪裡都一樣
* Vault Ethereum plugin 倚賴 Vault 本身提供的安全功能來保護錢包安全
* Vault Ethereum plugin 涵蓋幾乎所有的鏈上鏈下操作
* Vault plugin 生態系並沒有十分活躍

## Reference

* [Vault Plugin System](https://www.vaultproject.io/docs/internals/plugins)
* [Vault on Kubernetes Deployment Guide](https://learn.hashicorp.com/tutorials/vault/kubernetes-raft-deployment-guide)
* [Vault Helm Chart](https://github.com/hashicorp/vault-helm)
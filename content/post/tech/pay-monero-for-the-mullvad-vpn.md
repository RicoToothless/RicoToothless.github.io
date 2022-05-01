+++
author = "Rico"
title = "Pay Monero for the Mullvad VPN"
date = "2022-01-08"
description = "Purchase VPN service in complete anonymity."
tags = [
    "mullvad-vpn",
    "wireguard",
    "monero",
]
categories = [
    "tech",
]
archives = "2022-01"
+++

## Why we need VPN service?

我相信很多人都知道使用 VPN 可以偽裝 IP 到各個國家周遊列國，有很多服務會看使用者的 IP 當作國家地理上的判斷，但最近也會看信用卡當做依據。

很多人因為生活上有需求才用 VPN，為了看特定地區的影片、優惠折扣、有些國家會擋外國的 IP 又或者要連線公司內部網路，很多人都是有需要才會開 VPN，沒事不會特地掛著 VPN 上網，但我得說：

這根本是**裸奔**

你的電信商知道你上什麼網站、政府也知道你上什麼網站、你的房東厲害一點都知道你上什麼網站、咖啡廳老闆也知道你上什麼網站或者任何網站都知道你真實的 IP 是多少，想想就覺得毛骨悚然。

當然你會說那就直上 [Tor](https://www.torproject.org/) 不是更安全嗎？老實說沒錯，但瀏覽正常網頁很容易被擋，要做鬼打牆的 CAPTCHA 眼睛很累。

所以選擇折衷一點的 VPN 讓隱私跟使用者體驗兩者兼顧。

我想很多文章和影片講解 VPN 的技術怎麼實現的，但我還是用一張圖來替大家做個快速複習。

恩，這張圖不太衛生我知道，但這張圖真的可以闡述 VPN 的精髓。

![vpn-meme](https://i.kym-cdn.com/photos/images/original/001/524/130/f9d.jpg)

圖片來自 [know your meme](https://knowyourmeme.com/photos/1524130)

## Why Mullvad VPN?

市面上很多 VPN 服務商，選擇信任的 VPN 服務商很重要，因為他有能力知道你的真實 IP（public IP）和你要瀏覽什麼網站。

大部分的 VPN 的使用者都是跟其他使用者共用一台 VPN server，所以我有架設過 [Algo VPN](https://github.com/trailofbits/algo)，他就是可以在你喜歡的公有雲上架設 VPN server 就可以不用跟其他人共用機器。

但是在追求隱私的路上我還是覺得不夠，不論是 VPN 服務商或是公有雲註冊要用 email 註冊，而且付錢還是常見的金流系統，就是覺得不夠徹底。

最後我發現網路上有人推薦可以用這家 [ProxyStore](https://proxysto.re/en/index.html) 用 [Monero](https://www.getmonero.org/) 買 [Mullvad VPN](https://mullvad.net/en/) 的序號，引起我的好奇心看看這是怎樣的 VPN 服務商。

Mullvad VPN 最大的特色就是他不用 email 來註冊帳號，他是直接生產一組帳號給你，保管好這組帳號一直使用就可以了。

不過 Mullvad VPN 目前大部分付費的方式還是追蹤的到本人（他們公司也收信封寄的現金，但一樣追蹤的到），所以如果連付費都要匿名的話可以透過 ProxyStore 用 Monero 來買序號，選擇要買幾個月付完 Monero 之後過個 10 分鐘左右就會在頁面上顯示序號了，所以等待的期間，當下網頁的網址一定要記得。

Monero 跟 Bitcoin 和 Ethereum 等等加密貨幣不一樣的是，Monero 是完全追蹤不到交易紀錄的加密貨幣，非常適合注重隱私的人使用，而且都是由社群所推動的，社群的想法也非常注重去中心化。

當然 Mullvad VPN 的價格是真的蠻貴的，一個月 5 歐元，而且不管你是一次買一整年還是一次買十年，價格永遠固定一個月 5 歐元。

---

不論有沒有使用 VPN，一樣可以用 Mullvad 提供的 [check leak](https://mullvad.net/en/check/) 檢查自己的連線是不是安全的，檢查的項目有：

1. 是否有沒有用 Mullvad VPN，自己幫自己打個廣告不為過吧
2. 你目前的真實 IP 是否在黑名單裡，如果有，你可能不能造訪特地網站甚至會遇到更多 CAPCTCHA
3. 你的 DNS 有沒有洩漏你造訪網站的資訊，因為只要你去解析網域，DNS 就會知道你要去什麼網站，也就是說 DNS 也知道你在網路上的行蹤
4. WebRTC 技術是否洩漏出你真實 IP 的資訊，WebRTC 是可以讓使用者跟影音直接連線的技術，也因為如此他的副作用就是洩漏真實 IP 的資訊

Mullvad VPN 可以使用他們的寫的 client 連線，也可以直接使用 OpenVPN 或是 WireGuard 連。

## Conclusion

* VPN 可以讓你在網路上不裸奔
* Tor 雖然很有隱私但正常瀏覽網頁很不方便
* 選擇信任的 VPN 服務商很重要，因為他有能力知道你真正在網路上的行蹤
* Mullvad VPN 沒有直接收 Monero，但可以透過 ProxyStore 辦到
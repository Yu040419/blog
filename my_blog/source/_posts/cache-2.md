---
title: 快取 Cache 是什麼？那些網頁後端的快取（下）
date: 2021-03-14 14:24:28
tags:
  - cache
  - CDN
  - Proxy Server
---

上一篇介紹了什麼是快取，也簡單了解了網頁 Client-side 的快取，想了解的可以點選 [快取 Cache 是什麼？（上）](https://yu040419.github.io/blog/article/cache-1/) 認識。

在這篇當中會簡單談 Proxy Server 代理伺服器的快取，以及網頁 Server-side 的快取。

## Proxy Server

> 代理伺服器，也稱為前向代理

代理伺服器顧名思義就是一個代理人的概念，介於 client 端跟 server 端的中間人，需要由 client 端設定。做的事情包含：

- 代理 client 發送 request
- 連接網際網路
- 將資料存在自己的快取
- 收到 server 的 response 並回覆給 client

也因為 Proxy Server 這個中間人的關係，所以間接形成一個防火牆，當外部要攻擊 client 端時，就會攻擊到 Proxy Server，因為所有請求的 IP 位址都是 Proxy Server。

一般來說常見的架構是，多個 client 對應一個 proxy server。在這樣的狀況下，當 client A 請求過 R 網站，隔天 client B 也想要請求 R 網站時，proxy server 就會依照快取的設定檢查有無過期後，直接從快取拿資料回傳給 cient B，達到降低負載的效果。

但一旦請求太多，proxy server 也是有可能會有應付不來的可能，這時候就可以考慮幫 proxy Server 設置他的上層代理伺服器（類似幫這個 proxy server 再開幾台 proxy server 的概念），詳細可以參考 [鳥哥的文章](http://linux.vbird.org/linux_server/0420squid.php#theory_parent_proxy)。

Proxy Server 特色包含：

- 因為快取加快網頁存取速度、減少網路頻寬浪費
- 提供類似防火牆功能保護 client
- 搭配上層代理伺服器可達成降低負載（Load Balance）的效果

Proxy Server 雖然好用，但有幾個致命的缺點：

- 因為隱藏 client 的 IP 位址，可能會被濫用做壞事
- 由使用者設定，對一般大眾門檻太高。且因設定不易，可能會因為快取機制拿到舊資料

因此就誕生了 CDN。

## CDN

> Content Delivery Network，內容傳遞網路

### Reverse Proxy

在討論 CDN 之前，要先來談談反向代理伺服器（Reverse Proxy），因為 CDN 是反向代理的應用之一。那什麼是反向代理呢？其實很多概念跟 Proxy Server 類似，最大的不同就是，反向代理的人並非 client，而是 Server。

反向代理的特色包含：

- 一般大眾使用門檻低：因為由 server 端進行設定，所以使用者不須做任何設定
- 保護 Server：跟前向代理（proxy server）相反，因為接收 response 的 IP 位址都是反向代理的，所以外部要攻擊 server 的難度提升不少
- 壓縮內容以節省網路頻寬
- 保留前向代理的優點：
  - 提供快取加快使用者存取速度
  - 降低伺服器負載

### 運作特色

CDN 的概念就向是 Server 端將反向代理伺服器放在各個地方，降低原先 client 發送 request 到距離更遠的地方時，可能產生的網路連接及交換問題。

除了 反向代理原本的特色外，CDN 特色還包含：

- 提高傳輸效能：CDN 可以依照使用者的位置連接距離最近、最順暢的反向代理
- 提高系統穩定性：
  - 當某個反向代理伺服器遭到攻擊時，CDN 就可以將 request 連接到其他的反向快取，繼續提供服務
  - Server 當機時，也因為 CDN 的快取機制讓使用者不至於都無法連線

### 注意要點

Server 端在設定 CDN 的快取有效時間，跟 CDN 回覆給 client 端的快取有效時間需要一致，否則會發生資料不同步的事情。可參考 [網路傳輸的加速 - CDN 與 HTTP 緩存](https://mark-lin.com/posts/20190921/)。

## Server-Side

Server-Side 的快取有很多種，這邊主要談的是 Object Cache，類似資料庫的快取，而眾多資料庫快取當中，這邊只會簡單介紹 Redis。

### Redis

Redis 是 in memory 的 key-value 資料庫，位置在 Web Server 跟 DB 中間。因為 DB 的資料都存放在磁碟或 SSD 上，而 Redis 資料都位於記憶體內，有了 Redis 之後，可以省去存取磁碟的麻煩、避免搜尋時間延遲，並在幾微秒的時間內存取資料。

## 資料來源

- [網路傳輸的加速 - CDN 與 HTTP 緩存](https://mark-lin.com/posts/20190921/)
- [反向代理](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)
- [CDN](https://zh.wikipedia.org/wiki/%E5%85%A7%E5%AE%B9%E5%82%B3%E9%81%9E%E7%B6%B2%E8%B7%AF)

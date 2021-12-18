---
title: 快取 Cache 是什麼？那些網頁前端的快取（上）
date: 2021-03-14 14:10:03
tags:
  - cache
  - HTTP
---

中文是快取，中國翻作緩存。簡單來說就是把資料暫時存在一個地方，以快速讀取資料。

快取剛開始其實指的是硬體的 CPU 快取，後來衍伸出只要是類似可快速存取的應用，就可被稱為快取。其中包含但不限於網頁 Client-side 及 Server-side 的快取，以及 Proxy Server 代理伺服器的快取。

因為應用範圍很廣，此篇會先介紹網頁 Client-side 的快取。想了解 Proxy Server 代理伺服器的快取，以及網頁 Server-side 的快取，可以點選 [快取 Cache 是什麼？（下）](https://yu040419.github.io/blog/article/cache-2/)。

## Client-side Cache

前端相關的快取有很多種，這裡介紹的快取因為建立在 HTTP 運輸協定，因此也被稱為 HTTP Caching，都跟瀏覽器相關。

### 種類

依照快取優先順序分為以下四種：

1. Memory Cache：
   - 最優先
   - 部分瀏覽器才有，包含但不限於 Chrome
   - 存放在記憶體的快取
   - 該頁籤關閉就後，快取就會消失
   - F5 刷新網頁時大多都是從這裡快取資料
   - 相較 Disk Cache：
     - 存取速度快
     - 可存放的容量較小
2. Service Worker Cache
   - 僅次 Memory Cache 優先的快取
   - 需要安裝 Service worker 才可使用
   - 只限 HTTPS 協定
   - 即便 Service worker 沒有該資料的快取，但只要透過 Service worker，即便是從之後的 Disk cache 拿，都會顯示是從 Service worker 拿到的快取
   - 快取持續性長，關閉瀏覽器及頁籤後還可持續留存
3. Disk Cache / HTTP Cache
   - 存放在磁碟的快取
   - Client-Side Cache 大多都是 Disk Cache
   - 即便頁籤或是瀏覽器關閉，快取也不會消失
   - 相較 Memory Cache：
     - 存取速度較慢
     - 可存放的容量較大
4. Push Cache
   - 基於 HTTP/2 的 Session
   - Session 結束快取也跟著消失
   - 以上三種都沒有才會被使用
   - （實在是不了解 Push cache QQ

以上的優先順序如下圖（省略了部分瀏覽器才有的 Memory Cache）：
![](https://i.imgur.com/RHGoIVt.png)
[圖片來源](https://web.dev/service-worker-caching-and-http-caching/)

### 使用設定

HTTP Caching 除了 Service Worker 之外，其他都是透過 Server 端的 HTTP Response Headers 來設定，所以其實只要 Server 端寫好設定，瀏覽器就會自動依照設定做相對應的運作。以下設定方式是針對 Memory Cache 及 Desk Cache，並依照三種概念區分，分別是使用與否、有效期限及驗證。

#### 使用與否

- 不要使用：
  - `Cache-Control: no-store`：每次都發 request 到 Server 請求檔案
- 要使用：
  - `Cache-Control: public`：可被瀏覽器及中間的代理伺服器（Proxy Server）或是 CDN 快取
  - `Cache-Control: private`：只可被瀏覽器快取
  - `Cache-Control: no-cache`：每次都確認看資料有沒有變動更新。通常會搭配以下討論到的 `ETag` 或 `Last-Modified` 使用
  - `Pragma: no-cache`：跟 `Cache-Control: no-cache` 效用相同，但基本上不會使用，是古老的 header。

#### 有效期限

- `Cache-Control: max-age=60`：有效期限是收到 response 後 60 秒，假設 30 秒刷新頁面時，並不會跟 server 拿資料，而是從快取。是相對時間。
- `Cache-Control: s-maxage=60`：針對 proxy server 代理伺服器，像是 CDN 所設定的有效期限。如果 30 秒時 CDN 的快取內容有變，瀏覽器也不會發送 request。是相對時間。
- `expires: Wed, 21 Oct 2020 07:28:00 GMT`：絕對時間，該時間後就無效。

三者優先權： `Cache-Control: s-maxage` > `Cache-Control: max-age` > `Expires`。

#### 驗證

當快取過期時，不代表不能用快取裡的資料了，這時候就可以透過以下來跟 Server 確認，快取裡的資料是不是還可以使用。

驗證方法有兩種：

- 依照修改時間：`Last-Modified` 與 `If-Modified-Since`
- 依照檔案有無變動：`Etag` 與 `If-None-Match`

##### 依照修改時間

Server 設定的 Response Headers：

```
Status code: 200
Cache-Control: max-age=60
Last-Modified: Wed, 07 Oct 2020 07:28:00 GMT
// 上次修改時間是 2020-10-07 07:28:00 GMT
```

當一分鐘過後，快取已經過期時，使用者刷新瀏覽器時，瀏覽器會發送以下的 Request Headers：

```
If-Modified-Since: Wed, 07 Oct 2020 07:28:00 GMT
// 上次修改時間是 2020-10-07 07:28:00 GMT 嗎？
```

如果檔案不變時，Server 就會回覆：

```
Status code: 304 // 沒有改變 (Not Modified)
Cache-Control: max-age=60
Last-Modified: Wed, 07 Oct 2020 07:28:00 GMT
```

如果檔案有改變，Server 就會回覆：

```
Status code: 200 // 拿到新的檔案
Cache-Control: max-age=60
Last-Modified: Wed, 07 Oct 2020 07:30:00 GMT
// 換上新的修改時間 2020-10-07 07:30:00 GMT
```

##### 依照檔案有無變動

`Etag` 的概念有點類似 Hash，只要是同樣的輸入，一定會得到相同的輸出，`Etag` 也是。如果檔案內容沒有變動，`Etag` 的值也會是相同的。

Server 設定的 Response Headers：

```
Status code: 200
Cache-Control: max-age=60
Etag: x234dff
```

當一分鐘過後，快取已經過期時，使用者刷新瀏覽器時，瀏覽器會發送以下的 Request Headers：

```
If-None-Match: x234dff
// etag 值還是 x234dff 嗎?
```

如果檔案不變時，Server 就會回覆：

```
Status code: 304 // 沒有改變 (Not Modified)
Cache-Control: max-age=60
Etag: x234dff
```

如果檔案有改變，Server 就會回覆：

```
Status code: 200 // 拿到新的檔案
Cache-Control: max-age=60
Etag: x4524dgg
// 換上新的 Etag 值
```

一般來說，==會使用依照修改檔案與否的 `Etag` 與 `If-None-Match`==。因為有可能檔案從 A 改成 B 又改回 A，這樣依照編輯時間是有被改過的，還是會需要下載一遍相同的檔案。

### 常見使用類型

- 頻繁更動的資料：`cache-control: no-cache` 搭配 `etag`
  - 可以將 etag 的機制實作在 JS 檔案名稱，讓 JS 檔案。在 index.html 當中 `<script src="檔名有變動"></script>`，index.html 的 etag 就會改變，這時就會只需要
- 不常變化的資料：`cache-control: max-age=31536000` （一年）
- 希望檔案有更新，再發 request，可以將樓上兩者搭配使用。比方說在 index.html 設定 `cache-control: no-cache` 搭配 `etag`，而 index.html 裡 `<script src="script-qd3j2orjoa.js"></script>`，將 etag 機制直接放入檔案名稱，檔案一有更改是直接更改檔名。JS 檔設定 `cache-control: max-age=31536000`。
  這樣一來，一旦 JS 檔名更改，index.html 的 etag 也會更改，就可以達成目的。

### 失效

當瀏覽器發出 `PUT`、`POST`、`DELETE` 等請求時（要更改、新增、刪除資料時），原先該 URL 的各種 Client-side 快取就會自動失效。

### 強迫瀏覽器更新資源

- 打開 dev tools 的 Network 標籤，勾選 Disable cache 後，再重新按重新整理
- Ctrl + F5 / Ctrl + 按重新整理

## 資料來源

- [深入理解浏览器的缓存机制](https://www.jianshu.com/p/54cc04190252)
- [循序漸進理解 HTTP Cache 機制](https://blog.techbridge.cc/2017/06/17/cache-introduction/)
- [Web 快取](https://zh.wikipedia.org/wiki/Web%E7%BC%93%E5%AD%98)
- [HTTP Caching -- Summer](https://cythilya.github.io/2018/07/27/http-caching/)
- [HTTP 快取](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-tw)

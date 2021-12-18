---
title: CSS 效能優化
date: 2021-03-19 23:04:23
tags:
  - CSS
  - gzip
  - sprites
---

效能優化一直是個博大精深的主題，這邊簡單討論幾種方式，包含像是資源大小、載入方式及執行方式。

## 資源大小

進行文字資源優化，優化後檔案變小，當然傳輸時間就會縮短。

### Minify 最小化

在原本的 CSS 檔案當中，為了易讀性時常會有空格跟換行，Minify 就是將原本的檔案當中的空格、換行、註解全部拿掉，這樣檔案大小會縮小，但瀏覽器依然可以成功解析。

透過 Sass 內建的指令就可以達成（壓縮過的副檔名通常都是 min.css）：

```
sass style=compressed main.scss main.min.css
sass + 設定 style 是壓縮檔 + 原本的 scss 檔案名稱 + 壓縮過的 min.css 檔案名稱
```

或是透過 VS code 的 Live Sass Compile 也可以達成。

### Gzip

Gzip 就是壓縮，可以想像成一般我們常見的 zip 或是 rar 壓縮檔。在 Modern Web 現代化網頁開發設計中，大多瀏覽器都已經可以自動要求壓縮檔，並在渲染時自動進行解壓縮。若要求不到壓縮檔時才會存取原始檔案。

因此 Server 有沒有設定啟用讓 Gzip 壓縮檔案就很關鍵，像部署的 AWS EC2，預設模式就是會透過 Gzip 壓縮檔案。透過 Gzip 壓縮是最簡單且效率最高的優化方式。

更多文字資源優化可參考 [最佳化文字資產的編碼和傳輸大小 - Google](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer)。

## 載入方式

### Critical Css

把重要的資訊先渲染出來，讓使用者可以先收到一些資訊，其餘的再依序載入。

做法就是直接在 HTML `<head>` 中，直接把重要的 CSS 寫入`<style>`，其餘的再打包成一個檔案引入。這樣當瀏覽器在渲染出 HTML 檔案時，直接套用 CSS 樣式，不需要再發一個 request 到 CSS 檔案。

透過 Critical Css 優化效能最被人詬病的一點，就是不好維護，因為資料散落在不同地方。這時候可以參考 [這篇文章](https://www.w3cplus.com/css/understanding-critical-css.html)，裏頭討論如何透過基於 Node.js 的第三方套件 Grunt 來達成自動化 Critical Css。

### Css Sprites

是圖片最佳化的一種方式，大多運用在 icon 上。因為每個 icon 都是一個圖片檔，當今天一個網站可能有數十個甚至數百個 icon 時，就會有多少個 request，非常耗效能。

不如將圖片全部組合成一張圖片，瀏覽器只要載入一張大圖，再透過 CSS 的 `background-position` 屬性去做切割，只取需要的部分，就可以達成跟原先一樣的效果。

```css
height: icon 高;
width: icon 寬;
background-repeat: no-repeat;
background-position: x 軸, y 軸;
```

gulp 跟 webpack 都有提供將圖片合在一起，變成一張 Sprite 圖的服務。關於 Sprites Css 詳細可參考 [這篇](https://www.cnblogs.com/Ry-yuan/p/7392492.html)，裡面也有討論另一種圖片最佳化 ─ base64。

### Cache

可參考之前寫的 [快取 Cache 是什麼？那些網頁前端的快取（上）](https://yu040419.github.io/blog/article/cache-1/)，以及 [快取 Cache 是什麼？那些網頁後端的快取（下）](https://yu040419.github.io/blog/article/cache-2/)。

## 執行方式

在了解如何從 CSS 的選擇器及屬性渲染進行優化前，先來看 CSS 渲染方式，以及 HTML 是怎麼被渲染的。

### 瀏覽器如何渲染檔案

今天瀏覽器載入 HTML 的時候，會依照 HTML 的標籤元素依序製造出 DOM ( Document Object Model )。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <link href="style.css" rel="stylesheet">
    <title>Critical Path</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="awesome-photo.jpg"></div>
  </body>
</html>
```

假設是上方範例的文件，會是這樣的依序建立：

![](https://i.imgur.com/mTEvooA.png)

而當瀏覽器解析到 `<head>` 裡的 `<link>` 時，就會發 request 給該檔案，瀏覽器收到 response 後，也會開始製作 CSSOM ( CSS Objext Model )。

> 當瀏覽器在製作 CSSOM，別忘了它也同時在渲染 DOM，兩者是並行執行的。

當 DOM 跟 CSSOM 都渲染完後，瀏覽器就會將兩個合併，變成「轉譯樹狀結構（Render tree）」：

![](https://i.imgur.com/9uK9bBP.png)
若今天在某個 div 的 css 上加上 `display: none`，那這個 div 就不會在 Render tree 上呈現。

有了 Render Tree 後就可以開始進行版面計算及配置，並開始繪製網頁跟渲染。步驟如下：

1. 處理 HTML 標記，產生 DOM 樹狀結構。
2. 處理 CSS 標記，產生 CSSOM 樹狀結構。
3. 將 DOM 樹狀結構和 CSSOM 樹狀結構合併為 Render tree。
4. 對 Render tree 進行版面配置，計算每個節點的幾何形狀。
5. 在螢幕上繪製各個節點。

以上圖片及部分文字均源自 [Google](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model?hl=zh-tw)。

### 選擇器

簡單來說就是將低選擇器的複雜度。舉例來說：

```css
/* 瀏覽器可以快速了解是哪個元素 */
.title {
  ...
}
/* 瀏覽器需要先了解整個 dom 結構後並計算，然後再找底下的 .title */
.box:nth-last-child(-n+1) .title {
  ...
}
```

因此 CSS Class 建議以 BEM 的命名方式，這樣可以直接下 `.list__list-item { }` 直接選到該元素。更多可參考 [Google 文章](https://developers.google.com/web/fundamentals/performance/rendering/reduce-the-scope-and-complexity-of-style-calculations?hl=zh-tw)。

### 屬性渲染

其實在前面討論瀏覽器如何渲染頁面時，第四及第五點：

> 4. 對 Render tree 進行版面配置，計算每個節點的幾何形狀。
> 5. 在螢幕上繪製各個節點。

其實還可以更仔細的去區分成五個步驟，這樣的流程被稱為 Brower Rendering Pipeline。步驟分別是：
![](https://i.imgur.com/ztDZI83.jpg)

1. **JavaScript**：利用 JS 來處理視覺變更相關的工作，像是一些動畫、新增 DOM 元素、轉換等。CSS 動畫、轉換和網頁動畫 API 也是從這個階段開始。
2. **Style 樣式計算**：根據符合的選取器，弄清楚 CSS 規則適用於哪些元素。
3. **Layout 版面配置**：排版。計算元素要佔用的空間及在螢幕上的位置。如：width、height、position、margin、padding。
4. **Paint 繪製**：描繪文字、顏色、影像、邊框和陰影，如：box-shadow、border-radius、color、background-color、text-shadow。這階段耗時最長，應儘量避免。
5. **Composite 合成**：合成所有元素及圖層。如：transform、opacity。

當然瀏覽器在第一次載入渲染網頁時，每個步驟都需要跑過，不過後來的像是切換顏色、背景、邊框等不影響版面配置的動作，瀏覽器會跳過前面幾個步驟，直接進行 Paint；切換 transform 或是 opacity 時，也是直接進行 Composite。

因此這就是可以做效能優化的地方了，==當我們可以從第四、第五步驟開始做的時候，就不需要讓瀏覽器從第三步驟開始執行==。

最常見的例子就是當我們希望某個元素可以改變位置時，不要使用 position，因為 position 會影響版面位置，讓瀏覽器必須從第三步開始做起，而是改用 `transform: translate();`，直接做第五步。

## 資料來源：

- [關鍵轉譯路徑 Critical Rendering Path ─ Summer 桑莫](https://cythilya.github.io/2018/07/13/critical-rendering-path/#browser-rendering-pipeline)
- [Browser Rendering Optimization ─ TechBridge](https://blog.techbridge.cc/2016/04/02/Browser-Rendering-Optimization/)
- [Rendering Performance Overview ─ Google](https://developers.google.com/web/fundamentals/performance/rendering/)

## 推薦閱讀

- [圖片最佳化（Image Optimization） ─ Summer 桑莫](https://cythilya.github.io/2018/07/30/image-optimization/)
- [如何做圖片壓縮？ ─ Summer 桑莫](https://cythilya.github.io/2018/08/22/image-compression/)
- [文字資源優化 ─ Summer 桑莫](https://cythilya.github.io/2018/09/05/text-content/)
- [從 CSS sprite 進化到 SVG sprite](https://muki.tw/tech/css-to-svg-sprite/)

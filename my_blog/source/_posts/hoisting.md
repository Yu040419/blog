---
title: 透過範例理解 Hoisting 提升
date: 2021-03-21 12:12:51
tags:
  - JavaScript
  - hoisting
---

在談 Hoisting 之前，我們先測試看看以下程式

```javascript=
console.log(a); // 報錯： a is not defined
```

那換種方式呢？

```javascript
var a = 1;

function test() {
  console.log(a); // undefined
  var a = 2;
}

test();
```

a 居然不是 `1` 而是 `undefined`！為什麼會這樣呢？接下來會從 ECMAScript（ECMAScript 是 JavaScript 的標準）第三版（ES3）來了解一下 JS 引擎最初是如何執行程式的。

這邊要特別注意一下，`var` 跟 `let` `const` 的 hoisting 狀況不太一樣，這邊指的都是 `var`。

## 執行模型

JS 在執行程式前的編譯階段時，會先將整份檔案視為一個 Global Execution Context ，中文名是全域執行環境，簡稱 Global EC，並把 Global EC 放入 stack 中。

接下來開始依序從第一行開始掃，在全域（Global）的環境碰到變數時，會先將變數初始值設成 `undefined` 後放入 Variable Object（以下簡稱 VO）。碰到函式宣告時，也會將函式放入 VO，key 為函式名，value 為 function，如果函式宣告時包含參數，參數會放進該函式類似 VO 的 Activation Object（以下簡稱 AO），並將函式視為一個 Execution Context，push 進 stack 中。

進入函式的 EC 時，一樣從函式第一行開始掃，如果發現有宣告變數，就會放入該函式 AO，一樣將變數初始值設成 `undefined` 。以底下例子為例：

```javascript
var foo = "bar";
var a = 1;
function bar(a) {
  foo = "inside bar";
  var a = 2;
  c = 3;
  console.log(c);
  console.log(d);
}
bar(4);
```

步驟如下

1. 建立 global EC 並 push 進 stack
2. 第一行宣告變數 foo，放入 global EC 的 VO，並給予初始值 undefined
3. 第二行宣告變數 a，放入 global EC 的 VO，並給予初始值 undefined
4. 第三行宣告函式 bar，放入 global EC 的 VO，給予初始值 function。建立 bar EC 並 push 進 stack。發現宣告參數 a，將參數 a 放入 bar EC 的 AO，給予初始值 undefined
5. 第四行沒有發現宣告，不做事
6. 第五行宣告變數 a，但發現 a 已經存在 bar EC 的 AO 裡，因此不做事
7. 第六行沒有宣告，不做事
8. 第七行沒有宣告，不做事
9. 第八行沒有宣告，不做事
10. 第九行沒有宣告，不做事
11. 第十行呼叫函式 bar，且參數是 4，更改 bar EC 的 AO，a 從 undefined 變成 4

編譯後的成果類似下圖：

```javascript
// 模擬 VO 及 AO
global EC = {
  VO: {
    foo: undefined,
    a: undefined,
    bar: function
  }
}

bar EC = {
  AO: {
    a: 4 // 從 undefined -> 4
  }
}
```

```
模擬 stack 圖

|           |
|           |
|   bar EC  |  bar EC 接著 global EC 後被放入 stack
| global EC |  global EC 最早被放進 stack
|___________|
```

特別注意以上都是編譯的結果，程式都還沒執行，程式開始執行時，實際步驟類似如下

```
Line 1： 進入 Global scope，找 Global VO 是否有變數 voo ，有，將 "bar" 賦值給 Global VO 的變數 foo
Line 2： 找 Global VO 是否有變數 a ，有，將 1 賦值給 Global VO 的變數 a
Line 10： 呼叫 bar()
Line 3： 執行 bar() 進入 bar scope，將 4 賦值給 bar AO 的參數 a
Line 4： 找 bar AO 是否有變數 foo，沒有，往上找 global VO，有，將 "inside bar" 賦值給 global VO 的 foo
Line 5： 找 bar AO 是否有變數 a，有，將 2 賦值給 bar AO 的 a，蓋掉原本參數的 4
Line 6： 找 bar AO 是否有變數 c，沒有，往上找 global VO，還是沒有，在嚴格模式下會報錯，一般是會變成全域變數
         因此在 global VO 中新增一變數 c 並賦值 3
Line 7： 找 bar AO 是否有變數 c，沒有，往上找 global VO，有，是 3，印出 3
Line 8： 找 bar AO 是否有變數 d，沒有，往上找 global VO，沒有，報錯 'd is not defined'
```

程式執行後，會變成

```javascript
// 執行後 VO，註解是編譯後的狀態到執行後的演變
global EC = {
  VO: {
    foo: 'inside bar', // undefined -> 'bar' -> 'inside bar'
    a: 1, // undefined -> 1
    bar: function, // 沒變
    c: 3 // 沒有 c 這個變數變成有 c
  }
}

bar EC = {
  AO: {
    a: 2 // 4 -> 2
  }
}
```

因此執行過後結果是：

```javascript
var foo = "bar";
var a = 1;
function bar(a) {
  foo = "inside bar";
  var a = 2;
  c = 3;
  console.log(a); // 2
  console.log(c); // 3
  console.log(d); // d is not defined
}
bar(4);
```

如果第五行跟第七行對調，這時候答案就會變了

```javascript
var foo = "bar";
var a = 1;
function bar(a) {
  foo = "inside bar";
  console.log(a); // 4，因為這時候的 a 的值放的是參數
  c = 3;
  var a = 2;
  console.log(c); // 3
  console.log(d); // d is not defined
}
bar(4);
```

了解執行模型後，回過頭來看剛開始的題目，其實就可以很清楚的了解，為什麼 a 是 `undefined` 而不是報錯 a is not defined。

## 優先順序

從上面可以知道，`var` 宣告的變數、函式及函式內的參數等都會被提升，那當今天這三個都同名時，誰會優先被提升呢？

```javascript
function test(a) {
  console.log(a); // 參數名稱也是 a
  function a() {
    // function 名稱也是 a
  }
  var a = 2; // 變數名稱也是 a
}
test(3); // a() { }，函式會優先被提升

// 把函式刪掉
function test(a) {
  console.log(a); // 參數名稱也是 a
  var a = 2; // 變數名稱也是 a
}
test(3); // 3 ，代表參數提升較變數優先
```

所以優先順序是：==函式 > 參數 > var 宣告的變數==。而當今天有兩個同名的函式時，後者會覆蓋前者

```javascript
function test() {
  console.log(a());
  function a() {
    // function 名稱也是 a
    return "a1";
  }
  function a() {
    // function 名稱也是 a
    return "a2";
  }
}
test(); // a2
```

## 練習

```javascript
var a = 1;
function test() {
  console.log("1.", a);
  var a = 7;
  console.log("2.", a);
  function b() {
    a++;
  }
  var a;
  b();
  inner();
  console.log("4.", a);
  function inner() {
    console.log("3.", a);
    a = 30;
    b = 200;
  }
  console.log("5.", a);
}
test();
console.log("6.", a);
a = 70;
console.log("7.", a);
console.log("8.", b);
```

答案在底下，請先練習

＝＝＝＝＝＝＝　防雷線　＝＝＝＝＝＝＝＝＝

1. undefined
2. 7
3. 8
4. 30
5. 30
6. 1
7. 70
8. 報錯 b is not defined

## TDZ

`var` 相關的 hoisting 都了解後，現在可以來了解前面說跟 `var` 有點不一樣的 `let` `const`。先從範例來看：

```javascript
let a = 1;
function test() {
  console.log(a);
  let a = 2;
}

test(); // 用 node.js 跑 ReferenceError: a is not defined
// 用 devtool 跑：Uncaught ReferenceError: Cannot access 'a' before initialization
```

跑完會報錯，感覺很像是 `let` 沒有提升，不過如果 `let` 真的沒有提升，應該會存取到 global scope 的 `a = 1` 才對。

特別的就在這， `let` 跟 `const` 其實也會被提升，只不過宣告的變數，不會被初始化為 `undefined`，可以想像成 VO 裡有這個變數名稱的 key，但 value 是空的。因此在賦值之前試著存取該變數，都會出現錯誤。

前面有說過 `let` 跟 `const` 都是以大括號的 block 作為作用域，所以只要該 block 中有存取未宣告的變數，從進入 block 到賦值前，就會是 Temporal Dead Zone（TDZ），中文為暫時性死區，是一個為了解釋 `let` 與 `const` 的 hoisting 行為所提出的一個名詞。舉例來說：

```javascript
// function
function test() {
  var a = 1; // c 的 TDZ 開始
  var b = 2;
  console.log(c); // 錯誤
  let c = 10; // c 的 TDZ 結束
  if (a > 1) {
    console.log(a);
  }
}
test();

// if 條件式
let b = 2;
if (b !== null) {
  b = b * 2; // d 的 TDZ 開始
  console.log(d); // 錯誤
  let d = 3; // d 的 TDZ 結束
  b++;
}
```

## 資料來源

- [我知道你懂 hoisting，可是你了解到多深？](https://blog.techbridge.cc/2018/11/10/javascript-hoisting/)

## 推薦閱讀

- [function exprestion 的初始化](https://github.com/Lidemy/mentor-program-3rd-ClayGao/pull/24)

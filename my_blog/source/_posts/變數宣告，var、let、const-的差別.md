---
title: 變數宣告，var、let、const 的差別
date: 2021-03-09 23:25:14
tags: JavaScript
---

變數宣告方式有三種，分別是 `var`、以及 ES6 新增的 `let` 及 `const`。

`var` 跟 `let` `const` 最大的差別在於作用域，也就是變數的生存範圍。

在 ES6 之前 `var` 是以函式 function 為一個作用域，因此稱為 `functional scope`。而 ES6 之後是以 block 為一個作用域（大括號，像是任何迴圈、條件式等），稱之為 `block scope`。比方說：

```javascript
// 用 var 宣告
function test() {
  {
    var a = 10; // a 存在整個 test function 裡
  }
  console.log(a);
}
test(); // 10

// 用 let 宣告
function test() {
  {
    let a = 10; // a 只存在這個 block 中
  }
  console.log(a); // 這邊存取不到
}
test(); // 錯誤訊息：Uncaught ReferenceError: a is not defined
```

## var

如同前面所說，`var` 是 function scope，但如果 `var` 不在 function 裡宣告時，會變成全域變數（global variable），也就是整個檔案都存取的到，沒有作用域之分。

```javascript
// 案例 A，宣告在 function 裡
function test() {
  var a = 1;
  console.log(a);
}
test(); // 1
console.log(a); // 這邊找不到 a，因此噴 Uncaught ReferenceError: a is not defined

// 案例 B，寫在 function 裡但不宣告，會自動被宣告成全域變數
function test() {
  a = 1;
  console.log(a);
}
test(); // 1
console.log(a); // 1

// 案例 B 運作方式有點類似以下寫法
var a;
function test() {
  a = 1;
  console.log(a);
}
test(); // 1
console.log(a); // 1

// 案例 C，宣告在 function 外面成為全域變數
var a = 1;
function test() {
  console.log(a); // 這個 function 裡找不到，於是往外找
}
test(); // 1
console.log(a); // 1

// 案例 D，宣告全域變數後，在 function 內又重新賦值
var a = 20;
function test() {
  a = 10; // a 重新賦值成 10
  console.log(a);
}
test(); // 10
console.log(a); // 因此這裡也會是 10

// 案例 E，宣告全域變數後在 function 內又宣告另一個變數 a
var a = 2;
function test() {
  var a = 1; // 宣告在 function 內的 a 只存在這個 function 裡，不會重新賦值更動到外面的 a
  console.log(a);
}
test(); // 1
console.log(a); // 因此值還是 2

// 案例 F，不用 function 改用 if
var a = 1;
if (true) {
  var a = 2;
}
console.log(a); // 2
```

像案例 C 這樣從 test scope 找 a 變數，找不到往外找找到 global scope 的方式，被稱為 scope chain（範圍鍊）。

向案例 F 這樣不使用 function，而是使用條件式時，還記得之前說過 `var` 除了使用 function 時才有作用域的區別，因為這樣改到 global 原先值的情況，被稱為 ==變數汙染==。

但要特別注意以下情況：

```javascript
var a = "global"; // 全域變數

function change() {
  var a = "change";
  function inner() {
    var a = "inner";
    test();
  }
  inner();
}

function test() {
  // a 在 test scope 找不到於是往外找 -> global scope
  console.log(a);
}

change(); // global
```

以上很容易會以為印出來的值是 `'inner'`，不過其實跟函式在哪裡呼叫沒有關係，在設計該 function 時就已經決定好了該變數的 scope chain。不過如果改成以下，把最終的 function 放入：

```javascript
var a = "global"; // 全域變數

function change() {
  var a = "change";
  function inner() {
    var a = "inner";
    function test() {
      console.log(a);
    }
    test();
  }
  inner();
}

change(); // inner
// 因為 test scope 裡面沒有 a ，因此往外找到 inner scope，inner scope  a = 'inner'
```

在上面的例子當中，如果 inner scope 跟 change scope 都沒有宣告 a，那麼找尋的歷程就會是：

```
test scope -> inner scope -> change scope -> global scope
答案就會是 global
```

## let & const

`const` 代表的其實是 constant（常數），也就是不會變的數值。因此`let` 跟 `const` 最大的差別就在於 `let` 宣告的變數可以再重新賦值或更改，但 `const` 不行。

```javascript
// 數字重新賦值
const a = 1;
a = 2;
console.log(a); // 會噴以下錯誤訊息，const 宣告不能重新賦值
// Uncaught SyntaxError: Identifier 'a' has already been declared

// 先宣告後賦值
const a;
a = 2;
console.log(a); // 會噴以下錯誤訊息，const 宣告時就要給值
// Uncaught SyntaxError: Missing initializer in const declaration
```

以上用 `let` 都沒有這些問題，不過要特別注意：

```javascript
// const 仍可成功改值
const b = {
  number: 1,
};
b.number = 2; // b 貼著的箱子裡仍然是同一個記憶體位置
console.log(b); // {number: 2}

// const 無法成功改值
const b = {
  number: 1,
};
b = {
  // b 貼著的箱子要存放不同的記憶體
  number: 2,
};
console.log(b); // 因此就會噴以下錯誤訊息
// Uncaught TypeError: Assignment to constant variable.
```

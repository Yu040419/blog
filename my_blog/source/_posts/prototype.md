---
title: 什麼是 prototype ?
date: 2021-03-09 20:52:49
tags: JavaScript
---

## 前言

在 ES5 的 JS 物件導向中，function 被當成建構子以及 class 用，因此被稱為構造函數。構造函數可透過語法 `new` 建造一個 instance 實體，也可以透過 `prototype` 做出共用的 method 方法：

```javascript
// ES5
function Calculator(name) {
  // 用 function 來取代 class 名稱跟建構子
  this.name = name;
  this.text = "";
}
// 透過在剛的 Calculator function 名稱後加上 .prototype 加上.function 名稱
Calculator.prototype.input = function (str) {
  this.text += str;
};
Calculator.prototype.getResult = function () {
  return eval(this.text);
};

let calculateA = new Calculator("A");
let calculateB = new Calculator("B");
// 底下是重點
console.log(calculateA.input === calculateB.input); // true
```

不過 JS 是如何知道 `calculateA.input` 等同於 `calculateB.input` 的呢？或者可以換個問題，JS 是如何繼承 method 跟 property 的呢？

> 透過 `__proto__`

## `__proto__`

當我們建造 instance 時，JS 會自動幫我們在 instance 加上 `__proto__` 這個屬性，讓 instance 要使用該 method 找不到時，可以循著 `__proto__` 連結，往上找到有這個 method 的 `prototype`，以上面的例子來說，有點類似這個概念：

```javascript
// 實體 A
calculateA {
  name: A,
  text: '',
  __proto__: Calculator.prototype
}

// 實體 B
calculateB {
  name: B,
  text: '',
  __proto__: Calculator.prototype
}

// 構造函式的 prototype （放共用的 method & property）
Calculator.prototype {
  input: function(str) {
    this.text += str;
  },
  getResult: function() {
    return eval(this.text);
  },
  __proto__: Object.prototype // 構造函式的 prototype 也有 __proto__ 連結到 Object.prototype
}

// 最頂層是物件的 prototype
Object.prototype {
  __proto__: null
}
```

像這樣不停往上找，從

```
            透過 calculateA.__proto__ 找到
calculateA --------------------------------> Calculator.prototype


                      透過 Calculator.prototype.__proto__ 找到
Calculator.prototype ------------------------------------------> Object.prototype
```



> 這個透過 `__proto__` 尋找的過程，被稱為 **prototype chain 原型鍊** 。

## 驗證

以下介紹三種方式驗證原型鍊：

### ===

**一、透過 `===`**

```javascript
console.log(calculateA.__proto__ === Calculator.prototype); // true
console.log(calculateA.__proto__.input === Calculator.prototype.input); // true

// 連接的 __proto__ 可以被省略
console.log(calculateA.input === Calculator.prototype.input); // true
```

值得注意的是，不只有建構式的 prototype 才有 `__proto__`，建構式本身也有 `__proto__`。

以上面的例子來說，Calculator 這個建構式的 `__proto__`會連接到 Function.prototype。 Function.prototype 的 `__proto__` 則連接到頂層的 Object.prototype。

仔細想想其實一切都很合理，Calculator 本來就是 Function，連接到 Function 很正常；而 Function 本來就是 Object，連接到 Object 也很正常。可以想像成以下這樣

```javascript
Calculator {
  __proto__ : Function.prototype
}

Function.prototype {
  __proto__: Object.prototype
}

// 一樣都連到頂層的 Object.prototype
Object.prototype {
  __proto__: null
}

console.log(Calculator.__proto__ === Function.prototype) // true (1)
console.log(Function.prototype.__proto__ === Object.prototype) // true (2)

// 因為 (1) 是 true 所以可以把 (2) 改寫成以下
console.log(Calculator.__proto__.__proto__ === Object.prototype)
```

### hasOwnProperty

**二、 透過 `hasOwnProperty`**

```javascript
console.log(calculateA.hasOwnProperty("input")); // false
console.log(calculateA.__proto__.hasOwnProperty("input")); // true
console.log(Calculator.prototype.hasOwnProperty("input")); // true
```

### instanceOf

**三、 透過 `instanceOf`**
如果 `A instanceOf B` A 是 B 的實體的話，會回傳 `true`。

```javascript
console.log(calculateA instanceof Calculator); // true
console.log(calculateA instanceof Object); // true

// 注意下面
// constructor 是 Function 的實體
console.log(Calculator instanceof Function); // true
// constructor 也是是 Object 的實體
console.log(Calculator instanceof Object); // true

// 底下是特別的例子
console.log(Function instanceof Object); // true
console.log(Object instanceof Function); // true
```

#### constructor

其實每個的 prototype 底下，除了共用的 method、property 以及 `__proto__` 外，還有一個叫做 `constructor` 的屬性。

在前面我們透過 `instanceOf` 驗證原形鍊時，得知不只是 `new` 創建出來的 instance 才是 instance，像是 constructor 也是 Object 的 instance。

所以相反的，`Object.prototype` 底下的 constructor 屬性也是 Object，因為他們自己就是建構式，可以 new 出 constructor。當然 `Calculator.prototype.constructor = Calculator` ，因為 Calculator 本身就是 constructor 拉，他的 constructor 也自然指向自己。

因此任何 `X.prototype.constructor = X` 。

## new

了解原型鍊後就容易理解 `new` 是如何運作的，其實他幫我們做的步驟如下：

1. 創建一個空物件（以下簡稱 O），將 O 當成是 `new` 出來的實體
2. 透過 `__proto__` 讓 O 跟建構函式產生連結
3. 將 O 當成建構函式裡的 `this` 並透過 `call` 或 `apply` 執行建構函式
4. 回傳 O

```javascript
// constructor 建構函式
function Calculator(num) {
  this.num = num;
  this.result = "";
}

// 建構函式的 prototype
Calculator.prototype.calculate = function (operator, num2) {
  return (this.result = eval(this.num + operator + num2));
};

// 自己試著用函式做出類似 new 的功能 -> let calA = new Calculator(5)
function newCal(constructor, arguments) {
  let O = {}; // 創建類似實體的空物件
  O.__proto__ = constructor.prototype; // 透過 __proto__ 連接實體跟建構函式
  constructor.apply(O, arguments); // 執行建構函式
  return O; // 回傳類似實體的空物件
}

let calA = newCal(Calculator, [5]);
console.log(calA.calculate("*", 2));
// 上面跟底下透過 new 的實作出來結果相同
// let calA = new Calculator(5);
// console.log(calA.calculate('*', 2));
```

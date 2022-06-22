---
title: GraphQL 系列文（二）：透過 Query 存取資料
date: 2022-03-09 21:05:02
tags: GraphQL
---

在上一篇文章 [GraphQL 系列文（一）：透過 The Schema Definition Language (SDL) 撰寫 GraphQL Schema](https://yu040419.github.io/blog/article/SDL/) 中有提到，GraphQL 有三種存取資料的方式 (operation type)，主要常被使用的是 Query 查詢資料，以及 Mutation 新增、編輯、刪除資料。

因為底下會直接使用 GraphQL 的相關名詞，不會對名詞進行解釋，因此建議先閱讀過上一篇文章，對 GraphQL 有基礎理解後，再繼續往下看。

## Query
```graphql
query operationName($id: ID! = "93he", $type: string!) {
  user(id: $id) {
    name
    email(type: $type)
    location
    friends {
      name
      email
    }
  }
}

------------------------------
# variable
{
  "id": "39d0"
}
```

在執行 query 時，一定會加上 keyword `query` 這個 operation type，後面再加上這個 query 的名稱，官方說法為 operation name，operation name 不是必要，但對開發及後續維護上非常方便，建議一定要加上，好處如下：

- 增加易讀性
- DEBUG 時有 operation name 就不會找不到 query
- 搜尋 request 時快速有效率，很方便

Query 有幾個特色如下：

- 可巢狀存取
- 可自由選擇存取哪些資料
- 可一次存取多個 field 或多個 object type
- 存取到 object type 時，需展開繼續存取內部的 field（ 才知道說到底要什麼資料 ）

### argument 參數
若是 schema 中要求帶上參數，則必須在 query 名稱後面以括號帶上參數。
- 一般參數會透過變數處理。
- 變數會以 `$` 進行宣告。
- 每個 field 及 object type 都可以放上參數。
- 聲明與 schema 相符的 scalar type 及透過 `!` 表明是否為必要帶上的參數。
- 可以透過 `=` 給定參數預設值。

### alias 別名
今天若要一次取兩個相同名稱的 field 時，GraphQL 會報錯，因為一筆 Query 中，不可以有相同的 field。這時候就可以透過 alias 別名的方式，來重新對這個 object type 進行命名。

```graphql
query getUser1AndUser2($id1: ID!, $id2: ID!, $age: Int!) {
  user1: user(id: $id1) {
    name
    email
    friends(age: $age) {
      name
    }
  }
  user2: user(id: $id2) {
    name
    email
    friends(age: $age) {
      name
    }
  }
}

-------------------------------------------
# variable
{
  "id1": "39j3",
  "id2": "3je2",
  "age": 40,
}
```
以上的結果，得到的 response 就會是：

```json
{
  "data": {
    "user1": {
      "name": "Yu",
      "email": "a@gmail.com",
      "friends": [{
        "name": "Jacky",
      }],
    },
    "user2": {
      "name": "Kuan",
      "email": "b@gmail.com",
      "friends": [{
        "name": "Tommy",
        "name": "Cindy",
      }],
    }
  }
}
```
在剛上面的範例中，取的資料都相同，都是 user 資料中的 name 跟 email。在 GraphQL 有沒有其他方式方式，可以像 function 一樣，將相同的地方抽出來，重複使用呢？

還真的有！那就是 `fragment`！

### fragment 片段
透過 `fragment` 這個關鍵字宣告。使用時，再以類似展開元素，透過 `...` 使用即可。
在 `fragment` 中，一樣可以透過變數取值。而上面一樣的 code 透過 `fragment` 重寫後，變成以下：

```graphql
query getUser1AndUser2($id1: ID!, $id2: ID!, $age: Int!) {
  user1: user(id: $id1) {
    ...userData 
  }
  user2: user(id: $id2) {
    ...userData 
  }
}

fragment userData on User {
  name
  email
  friends(age: $age) {
    name
  }
}
```

### inline fragment
當今天若是想要 query 的資料型別，不是 object type / field，而是 Interface 跟 Union type 時，這時候就會需要透過 inline fragment 進行存取。

拿上一篇提到的範例來說：
```graphql
# Schema
union Device = Mobile | PC

type Mobile {
  id: String!
  price: Float!
}

type PC {
  id: String!
  price: Float!
}

type Query {
  getDevice: [Device]
}
```
在 query 的時候，就會以 inline fragment 呈現：
```graphql
query ExampleQuery {
  getDevice {
    Device {
      ... on Mobile {
        id
        price
      }
      ... on PC {
        id
        price
      }
    }
  }
}

```

### directive 指令
在上一篇文章中有提到官方原生有三個 directive 指令，schema 中會使用一個。而在 query 中會使用到剩餘的兩個。

在了解之前，可以先猜猜看，以下的 query，最後會拿到什麼資料：
```graphql
query ($hasFriends: Boolean!, $isSensitive: Boolean!) {
  me {
    name
    ...sensitiveData @skip(if: $isSensitive)
    friends @include(if: $hasFriends) {
      name
    }
  }
}

fragment sensitiveData on User {
  age
  weight
  height
}

----------------------------
# varaibles
{
  "hasFriends": false,
  "isSensitive": true
}
```

#### @include
```graphql
@include(if: boolean)
```
如果 boolean 是 true 的話，就存取這個欄位，一般會透過變數管理 boolean。

#### @skip
```graphql
@skip(if: boolean)
```
如果 boolean 是 true 的話，就略過這個欄位，也就是不存取，一般會透過變數管理 boolean。

所以猜對了嗎？在上面的 query 中，最後拿到的資料是：
```json
{
  "data": {
    "me": {
      "name": "Yu"
    }
  }
}
```
噹啷！就是只有 name！

## 結語
Query 的部分大概就到這邊，Mutation 可參考下一篇。

## Reference
1. [官網](https://graphql.org/)
2. [Think in GraphQL 系列文](https://ithelp.ithome.com.tw/users/20111997/ironman/1878?page=1)

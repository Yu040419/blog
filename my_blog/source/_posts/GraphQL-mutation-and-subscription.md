---
title: GraphQL 系列文（三）：Mutation & Subscription 
date: 2022-06-22 20:28:01
tags: GraphQL
---

假設今天有個 GraphQL Schema 如下，今天需要發送一個請求，請求內容中需要拿到使用者的 ID、名稱、年紀、身高 (公分)、所有貼文的標題及內容等。

那該如何寫請求呢？答案我放在這篇文章最下方，可以先自己練習看看。

如果看不懂 Schema、或是不知道如何寫 Query，的，可以先參考系列文章的前兩篇，[GraphQL 系列文（一）：透過 The Schema Definition Language (SDL) 撰寫 GraphQL Schema](https://yu040419.github.io/blog/article/SDL/) 及 [GraphQL 系列文（二）：透過 Query 存取資料](https://yu040419.github.io/blog/article/GraphQL-query/)，複習一下再繼續往下看。

```graphql
schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}

type Query {
  hello: String
  me: User
  user(name: String!): User!
  users: [User!]!
}

interface Character {
  id: ID!
  name: String
}

type User implements Character {
  id: ID!
  name: String
  email: String
  age: Int @deprecated(reason: "Use birthDay")
  height(unit: HeightUnit = CENTIMETRE): Float
  weight(unit: WeightUnit = KILOGRAM): Float
  friends: [User]
  posts: [Post]
  birthDay: String
}

type Post {
  id: ID!
  title: String
  content: String
  createdAt: Int
}

type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload
  deletePost(id: ID!): DeletePostPayload
}

type Subscription {
  nextTrainArrivalTime(trainId: String!): String!
}

input CreatePostInput {
  title: String!
  content: String!
}

type CreatePostPayload {
  createPost: Post
  error: Error
}

type DeletePostPayload {
  deletePost: Post
  error: Error
}

type Error {
  code: Int
  message: String
}

enum WeightUnit {
  KILOGRAM
  GRAM
  POUND
}

enum HeightUnit {
  METRE
  CENTIMETRE
  FOOT
}
```

## Mutation

```graphql
mutation createPost($input: CreatePostInput!) {
  post1: createPost(input: $input) {
    createPost {
      id
      createdAt
    }
  }
}

-------------------
# Variables
{
  "input": {
    "title": "GraphQL",
    "content": "Mutation and Subscription"
  },
}
```

跟執行 Query 一樣，在執行 mutation 時，一定會加上 keyword `mutation` 這個 operation type，後面再加上這個 mutation 的名稱，官方說法為 operation name。至於 operation name 的好處，在這邊就不再說一次，可以參考上一篇文章。

從以上的 mutation 可以發現：

- 有使用 Input Object Type 取代 Arguments
- 透過 `$input` 這個變數去管理 mutation 的 Input
- 將 createPost 取了暱稱 `post1`
- 除了新增貼文外，createPost 這個 mutation 還存取了 `CreatePostPayload` 內容中的，`createPost`。
  - `CreatePostPayload` 跟 Query 的概念相同，都可以自由進行存取

上方的 Mutation 當中，拿到的資料顯示狀況會呈現如下：

```json
{
  "data": {
    "post1": { // 使用 Alias
      "createPost": {
        "id": "793ff34d-5a45-49b1-a6ca-b500efed625e",
        "createdAt": 1128746332
      }
    }
  }
}
```

### Input Object Type

在討論 Schema 的時候有提到，Mutation 有兩個以上的 Argument 參數時，就會建議透過 input Type 來作為 Object Type 的 Argument。

```graphql
input CreatePostInput {
  title: String!
  content: String!
}
```

以上方這個 input 為例，這個 input 需要 title 及 content 這兩個 Field Names，兩個的 Field Types 都是 String。而且兩個都有 `!` 這個 Type Modifier，顯示這個兩個 field 都是必填。

### Mutation 設計建議

#### 1. 動詞開頭

比方說 createPost / updatePost 等。

#### 2. 建議每個 mutation 搭配自己的 input 及 payload，方便未來的擴展

比方說 mutation 名稱為 updatePost 時，input 為 UpdatePostInput、payload 為 UpdatePostPayload。

#### 3. 依照商業邏輯細分 Mutation

剛開始大家會將 RESTful 的 CUD 來對應 Mutation，但 Mutation 可以完全因應前端需求調整，因此如果需要細分的就細分，需要一次更改多個欄位的就包在一隻 Mutation 中。

#### 4. 參數盡量使用 input object type 管理參數

除了易讀性較高外，input object type 經常也能被重複使用。

#### 5. 設計每個 mutation 的專屬回傳資料

概念上跟之前 RESTful 都會有個 response 內容有點類似，建議為每個 mutation 加上個別獨立的 Object Type。

### 練習

在上面的 Schema 當中，只有 CreatePost 及 DeletePost 的功能，所以可以試著建立看看 UpdatePost 來編輯貼文。並試著發送 mutation 的 request，然後獲取該文章更新後的內容及錯誤訊息，答案一樣會放在下方。

## Subscription

在前面提到的 Query 跟 Mutation，基本上都是常見的由 Client 端發送 request 給 Server，當 Server 接收到請求後，再回傳 Response 給 Client。

一般來說若希望可以由 Server 主動推送資料給 Client 端的話，會透過 WebSocket 來實作。在 GraphQL 當中，Subscription 就是扮演這樣的角色，開發者不需自行串接，只需要透過與 Query 相似的語法，就可以輕鬆實現（GraphQL Subscription 底層也是透過 WebSocket 實作）。

### Subscription Library

> TL;DR: 下載 [graphql-ws](https://github.com/enisdenjo/graphql-ws) 作為 Subscription Libray

在實作 Subscription 之前，需要先選擇前後端介接的 Protocol，專案的前後端都必須使用同一個 Protocol，才可以成功實作 Subscription。

而在 GraphQL 當中有兩個熱門的 Library 可供選擇，分別是 [subscriptions-transport-ws](https://github.com/apollographql/subscriptions-transport-ws) 及 [graphql-ws](https://github.com/enisdenjo/graphql-ws)。

神奇的地方來了！
Library 名稱是 subscriptions-transport-ws 的 `Sec-WebSocket-Protocol` 名稱是 graphql-ws。而 Library 名稱是 graphql-ws 的 `Sec-WebSocket-Protocol` 名稱是 subscriptions-transport-ws。

是的，你沒看錯，他們的 Library 名稱跟 Protocol 名稱剛好顛倒，非常容易造成混淆 QQ

而 Library 名稱是 subscriptions-transport-ws 目前已經不再維護了。不管是 Apollo 官方及 subscriptions-transport-ws Libray 本身，都推薦使用 graphql-ws 這個 Libray。所以毫無懸念，就下載 graphql-ws 吧！

graphql-ws 相關的設定，可以參考 [Apollo 官方文件](https://www.apollographql.com/docs/react/data/subscriptions/#setting-up-the-transport)。

### Client-Side

```graphql
subscription GetNextTrainArrivalTime($id: ID!) {
  nextTrainArrivalTime(trainId: $id)
}

-------------------
# Variables
{
  "id": "Knight-Bus",
}
```

以上方為例，可以看到架構上基本上都跟 Query 及 Mutation 無異，都是由 operation Type + Operation Name。如果需要 Argument 則都透過變數代為管理。

## 結語

GraphQL 系列文章就差不多這樣告一個段落，一開始介紹了最重要的 Schema，可以理解為前後端都需要遵守的 Spec。Schema 訂好之後，GraphQL 會協助把關，如果前端傳送非 Schema 訂定的資料類別或架構，就不會發送 Request 給後端；反之亦然，後端若回覆非 Schema 訂定的資料類別或架構，就無法成功發送 Response 給前端。

接下來了解了 Schema 三本柱，Query、Mutation 及 Subscription。Query 跟 Mutation 都是基於 HTTP 傳輸協定。Query 用來取得資料，而 Mutation 用來新增、修改、刪除資料。Subscription 則是基於 WebSocket 傳輸協定，讓 Server 可以主動推送訊息給 Client，常用於顯示一些即時資料。

上方的所有程式碼，都有建立 [codesandbox](https://codesandbox.io/s/priceless-driscoll-9jzqb6?file=/typeDefs.js) 作為 Live Code Demo，想玩玩的都可以點 Codesandbox 連結去玩玩看～

## Answer

### 取得使用者相關資料

```graphql
query getUserInfo($name: String!) {
  user(name: $name) {
    id
    name
    height
    birthDay
    posts {
      title
      content
    }
  }
}


=====================
# variables
{
  "name": "Minerva-McGonagall",
}
```

### 編輯貼文

#### Schema

```graphql
schema {
  mutation: Mutation
}

type Post {
  id: ID!
  title: String
  content: String
  createdAt: String
}

type Mutation {
  updatePost(input: UpdatePostInput!): UpdatePostPayload
}

input UpdatePostInput {
  id: ID!
  title: String
  content: String
}

type UpdatePostPayload {
  updatePost: Post
  error: Error
}

type Error {
  code: Int
  message: String
}
```

#### Mutation

```graphql=
mutation updatePost($input: UpdatePostInput!) {
  updatePost(input: $input) {
    updatePost {
      content
    }
    error {
      message
    }
  }
}

-----------------
# varialbes
{
  "input": {
    "id": "0731",
    "title": "The Philosopher's Stone",
    "content": "Harry Potter and The Philosopher's Stone",
  }
}
```

## Reference

- [Think in GraphQL 系列](https://ithelp.ithome.com.tw/users/20111997/ironman/1878)
- [Apollo Docs](https://www.apollographql.com/docs/)

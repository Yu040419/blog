---
title: The Schema Definition Language (SDL) - 撰寫 GraphQL Schema
date: 2022-01-13 21:48:44
tags: GraphQL
---

SDL 是建立 GraphQL Schema 的語言，而 Schema 則是定義 GraphQL API 的重要骨幹，包含資料架構格式和型別。

在還不太清楚前，可以先簡略的將 GraphQL Schema 想像成 DB Schema。一個 DB 可以有多個 Table，而一個 Schema 可以有多個 type；DB 中的 table 中有多個欄位，可分別設置資料型別，而 GraphQL type 也可以有多個 field (欄位)，每個欄位的資料也可設定型別。

## 存取方式

在 GraphQL 中，資料存取方式分成三種，分別是 query, mutation 跟 subscription。

- query：查詢資料
- mutation：新增、編輯、刪除資料
- subscription：擺脫以往發送 request 後得到 response 的方式，而是以透過 websocket 的方式，client 端訂閱某個資料後，Server 端一有新的資料後就會主動發送給 client。

## 資料型別 Scalar Types

1. 可以讓 Client 與 Server 的開發者對於資料格式有共同的認知
2. 強迫 Client 送出正確格式的 query
3. 強迫 Server 回覆正確格式的 response

### 預設 Scalar Types

資料型別預設分成五種，分別是：

- Int：32-bit 整數
- Float：double precision 雙精度的浮點數
- String
- Boolean
- ID：ID 一般來說會以 string 方式呈現，但當前端輸入 id 為 "483109245" (string) 或是 483109245 (int) 都會被 GraphQL 所接受。

### 自訂 Scalar Types

除了以上的型別外，也可以自訂型別，常見的有 Date, URL, Email, JSON 等等。可以動手 [實作](https://ithelp.ithome.com.tw/articles/10206366)，也可以透過 [套件](https://www.npmjs.com/package/graphql-custom-types)。

## directive 指令

- directive 指令以 `@` 宣告
- 是一種語法糖
- 可以 custom directive
- [原生](https://www.apollographql.com/docs/apollo-server/schema/directives/#default-directives)有三個指令，一個用在 schema 就是 `@deprecated(reason: String!)`，另外兩個指令可參照 [如何透過 GraphQL 存取資料 － Query](https://yu040419.github.io/blog/article/GraphQL-query/)。
  - `@deprecated(reason: String!)`：schema 使用，是用來呈現在文件上，告訴 client 端盡量不要存取該欄位的用法，因此一定需要帶上 reason 的值。

---

## Syntax 語法

在了解以上基礎後，接下來就可以直接來看看，如何透過 SDL 訂定 GraphQL 的 Schema。

### Comment

SDL 的註解方式分成三種，分別是 `#`、`"` 及 `"""`。

- `#`：SDL 單行註解方式，不會呈現在自動生成的 GraphQL 文件中。

```SDL
# 這個單行註解不會出現在文件中
```

- `"`：SDL 單行註解方式，會呈現在自動生成的 GraphQL 文件中，經常用來備註 field definition 。

```SDL
" 這個單行註解會出現在文件中 "
```

- `"""`：SDL 多行註解方式，會呈現在自動生成的 GraphQL 文件中。

```SDL
"""
這個多行註解會出現在文件中
"""
```

### Schema

GraphQL 中的語法關鍵字 `Schema` 被官方稱為 Root Types，可以理解成所有存取資料的 entry point 進入點。在上面我們有提到存取方式有三種，因此 `Schema` 最多也只會包含三種類型：

```graphql
schema {
  "查詢"
  query: Query
  "新增、修改、刪除"
  mutation: Mutation
  "訂閱"
  subscription: Subscription
}
```

### Type / Field

> `type` 是宣告 `Object Type` 的關鍵字。
> `Object Type` 中若有可選的欄位，則是 `Field`。

底下範例中：
* Query 是 Object Type 的名字，供查詢。
* 設定可以查詢的 Field 欄位被稱為 Field Names，這邊有有四個，分別是 hello, me, users, user。
* `string` `User` `User!` `[User!]!` 被稱作 Field Types。
* `users` 這個 Field Name 需要帶上 argument 參數 name，詳見以下 argument。

```graphql
type Query {
  hello: String                    # 資料格式是字串
  me: User                         # 資料指向另一個 Object Type，此 Object Type 命名為 User
  user(name: String!): User!       # 資料指向另一個 Object Type User，而且不會是空值。 () 內的是參數 argument，表示一定要帶上參數 name，且參數值一定要是不得為空的字串
  users: [User!]!                  # 資料不會是空值，且內容是一個陣列，此陣列內也不會是空值，陣列內容指向 User
}
```

### interface

> 透過 `interface` 關鍵字宣告介面。

在以下範例中 Character 是這個介面名稱，實作此介面的時候都必須包含以下三個欄位。（個人認為有點類似 OOP 的多型）。
```graphql
interface Character {
  id: ID!
  name: String
  avatarUrl: String
}
```

#### implements
> 當要透過 `type` 實作出 `interface` 時，都需要透過 `implements` 這個關鍵字。

```graphql
"""
使用者相關資料
"""
type User implements Character {
  id: ID!      
  name: String
  avatarUrl: String
  email: String
  age: Int @deprecated(reason: "Use birthday")   # 已被 deprecated
  height(unit: HeightUnit = CENTIMETER): Float   # 帶有 arguments 參數 unit，參數型別為 HeightUnit，詳見以下 enum，此參數預設值是 CENTIMETER
  weight(unit: WeightUnit = KILOGRAM): Float     # 帶有 arguments 參數 unit，參數型別為 WeightUnit，詳見以下 enum，此參數預設值是 KILOGRAM
  friends: [User]
  posts: [Post]                                    
  birthday: String
}

"""
粉專相關資料
"""
type FanPage implements Character {
  id: ID!
  name: String
  avatarUrl: String
  likeGivers: [User]
}
```

### enum
> 受限的選項，常用在參數、Field Types 等。

在以下的範例中，型別為 `WeightUnit` 的，value 只有可能有三種可能，分別是 `KILOGRAM`、`GRAM`、`POUND`等。

```graphql
"""
體重單位
"""
# 代表值只有三種可能，分別是
enum WeightUnit {
  KILOGRAM
  GRAM
  POUND
}

"""
身高單位
"""
enum HeightUnit {
  METRE
  CENTIMETRE
  FOOT
}
```

### union
可以理解成 type 的集合，當今天回傳資料類型，可能包含一個以上的 type 時，`union` 就非常適合。

以下範例來說，會依照參數需求，來回傳需要的設備是使用手機還是電腦，這時候就很適合使用 union 回傳。

```graphql
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
  getDevice(need: String!): [Device]
}
```

### argument / input
> 參數，經常使用在 query 及 mutation 中。
> input 是宣告 input 這個 Object Type 的關鍵字。

參數數量若大於 3 個的話，通常會包成一個 input。

```graphql
type Mutation {
  "新增使用者資訊，參數使用 input Object Type，一定要放參數"
  addUser(input: addUserInput!): addUserPayload         # 自定義回傳 payload
  "編輯使用者資訊"
  updateUser(input: updateUserInput!): User
  "新增貼文資訊，參數小於三個，直接放裡面，title 選填，author 必填"
  addPost(title: string, author: string!): Post
}

"""
新增使用者參數
"""
input addUserInput {
  name: String!
  email: String!
  age: Int
}

"""
更改使用者資料參數
"""
input updateUserInput {
  id: ID!
  name: String
  email: String
  age: Int
}

"""
新增使用者後的回傳資料
"""
type addUserPayload {
  addUser: User   
  "商業邏輯層的錯誤"
  error: [UserError!]!
  "執行時間"
  timestamp: String
}

"""
自定義 Error 格式
"""
type UserError {
  code: Int
  message: String!
}
```

## Reference
1. [官網](https://graphql.org/)
2. [Think in GraphQL 系列文](https://ithelp.ithome.com.tw/users/20111997/ironman/1878?page=1)

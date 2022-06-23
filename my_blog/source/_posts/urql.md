---
title: URQL 的基本介紹與常見應用
date: 2022-06-23 23:47:49
tags: GraphQL
---

[URQL](https://formidable.com/open-source/urql/)，全名為 Universal React Query Libray。從名稱就可以看得出來，最初是基於 React 而生的 Libray。

為什麼我說是「最初」呢？就是因為其強大的團隊，也開發出了支援 Vue (3) 及 Svelte 等其他前端框架。在整合度非常高的狀況下，卻又以輕便、高度客製化、多功能、簡單好上手等多樣特色聞名。

1. **整合性高**：除了支援前端的 React, Vue, Svelte 框架外，URQL 本身還提供 devtool 可供使用者開發 debug 使用。
2. **輕便**：bundle size 僅 7.1 kb，相較其他 GraphQL Client Library，[Apollo Client](https://www.apollographql.com/docs/react/) 最迷你的版本也需要 32 kb，[React Relay](https://relay.dev/) 也要 34 kb。
3. **多功能**：URQL 本身提供非常多樣化的功能，官方有提供一張 [比較](https://formidable.com/open-source/urql/docs/comparison/) URQL 與其他 GraphQL Client 的表格，就可以看到支援相當多的功能，包含 Stale while Revalidate / Focus Refetching / Dependent Query 等等。

接下來就來看看，該如何實際應用 URQL 吧！

## Install

```bash
yarn add @urql/vue graphql
```

## Setting

```typescript
// graphql/index.ts
import { getAuthToken } from "@/hooks/useAuthentication";

interface Headers {
  authorization: string;
}

interface ReturnObject {
  headers: Headers;
}

const urqlSetting = {
  // GraphQL API Endpoint
  url: "http://your-api/graphql",

  fetchOptions: (): ReturnObject => {
    // 將 token 帶入 header
    const token = getAuthToken();

    return {
      headers: {
        authorization: token ? `Bearer ${token}` : "",
      },
    };
  },

  // 快取設定
  requestPolicy: "cache-and-network",
};

export default urqlSetting;
```

```javascript
// main.ts

import { createApp } from "vue";
import App from "./App.vue";
import { router } from "./router";
import urql from "@urql/vue";
import urqlSetting from "@/graphql";

const app = createApp(App);
app.use(urql, urqlSetting);
app.use(router);
app.mount("#app");
```

## Request Policy

URQL 的請求政策共分成四種，分別是：

- **cache-first**：預設值，有快取的時候從快取拿值，沒有快取的時候，就發送請求取得資料。
- **cache-and-network**：會先回傳快取資料，並同時在背後發送請求。一般推薦以這個方式，可以達到使用者體驗與取得正確資料的平衡。
  - 當有快取資料，發送請求時，`isFetching` 值會是 false。所以使用者會優先看到快取資料，不會顯示 loading 內容。
- **network-only**：忽略快取，每次都會發送請求。
- **cache-only**：只回傳快取資料，若是沒有快取資料，會回傳 `null`。

## useQuery

> GET

query 內容或是變數有變動時，會自動發出 request。

```javascript
const { data, fetching, error, isPaused, pause, resume } = useQuery({ query: `...`, varaiables: { ... }, pause: boolean})
```

範例：

```html
<template>
  <div v-if="fetching">Loading...</div>
  <div v-else-if="error">Oh no... {{error}}</div>
  <div v-else>
    <ul v-if="data">
      <li v-for="todo in data.todos" :key="todo.id">{{ todo.title }}</li>
    </ul>
  </div>
  <button @click="isPaused ? resume() : pause()">Toggle Query</button>
  <button @click="refresh">Query Again</button>
</template>
```

```javascript
<script>
import { useQuery, gql } from '@urql/vue';

export default {
    setup(props) {
      const from = ref(0);
      const limit = ref(10);

      // pause, resume, executeQuery 都是 function
      const { data, fetching, error, isPaused, pause, resume, executeQuery } = useQuery({
        query: gql`
          query ($from: Int!, $limit: Int!, $isFinished: Boolean!){
            todos (from: $from, limit: $limit, isFinished: $isFinished) {
              id
              title
            }
          }
        `,
        variables: {
          from,
          limit,
          isFinished: props.isFinished
        },
        // pause 值是 true 的話停止執行該 query
        pause: computed(() => !from.value || !limit.value)
      });


      // 直接這樣取值的時候，因為 call API 後，response 還沒回來，所以會是 undefined
      // 所以 return 出去到 template 的時候都要篩選
      console.log(data.value.todos.title);

      watch(data, (newVal) => {
        // 這邊就可以拿到值
        console.log(newVal.todos.title);
      })

      // 手動立即執行 query
      const refresh = () => {
        executeQuery({
          requestPolicy: 'network-only',
        });
      };

      return {
        fetching,
        data,
        error,
        refresh,
        isPaused,
        pause,
        resume,
      }
    }
</script>
```

### data

> ref 值，最重要的資料

存取時可透過 `.value` 存取 data 值。要記得再 . 存取 query 內容的值，詳細可看上方範例。

### fetching

> ref 值，Boolean，`true` 表示正在抓取資料

轉變過程：undefined -> true -> false

### error

> ref 值，沒有錯誤的時候拿到的值會是 undefined

[文件](https://formidable.com/open-source/urql/docs/basics/errors/) 中把錯誤稱為 combinedError，因為 error 包含兩種狀況，分別是 networkError 以及 graphQLErrors。

可以透過 `error.value.message ` 拿到錯誤訊息，這個錯誤訊息會合併兩種錯誤情況，

`error.value.name` 會拿到錯誤名稱，名稱會是 `'networkError'` 或 `'graphQLErrors'`。

error 可以跟 data 共存，也就是說有可能出現，同時有 data 也有 error 的狀況，因為 GraphQL query 可以是部分失敗的。error 這時候就會拿到 graphQLErrors。

#### networkError

任何跟網路連線相關的錯誤，就會被歸類到 networkError。存取時是透過 `error.value.networkError` 存取。

networkError 中又包含兩個 property，分別是 `message` 及 `networkError`。

#### graphQLErrors

跟 GraphQL 相關的錯誤就會被歸類到這裡，舉凡 scalar type 不對、缺少某些 field 或是 variables，GraphQL 格式不對等等。

graphQLErrors 中也包含兩個 property，分別是 `message` 及 `graphQLErrors`。

跟 `networkError` 不一樣的是，`graphQLErrors` value 的 type 是一個陣列，因為可能併發多種 GraphQL 的錯誤。

### isPaused

> ref 值，Boolean，true 代表目前 query 有被暫停

### pause

> function，如果 query 可執行，可用來暫停執行 query。

pause 是用來提供在 setup 外、也就是 template，也可以暫停執行 query。

### resume

> function，如果 query 被暫停執行，可用來啟動執行 query。

resume 是用來提供在 setup 外、也就是 template，也可以啟動執行 query ( 但不是立即執行，只是可以執行 )。

### executeQuery

> function，立即執行 query，

執行此 function 的情況，通常都是用來希望可以重新取得最新資料，所以一般都會在參數內放入 `requestPolicy: 'network-only'`，詳細可參考上方範例。

## useMutation

> CREATE / UPDATE / DELETE

```javascript
const { executeMutation: rename, data, error } = useMutation(`...`);

rename({ input }).then(...)
```

範例：

```javascript
// 變數
const newTodoInput = reactive({
  todo: "sleep",
});

// useMutation 只有單一參數 string
// 這邊跟 query 一樣，都可以拿到 request 完之後的 data、fetching 以及 error
// executeMutation 基本上都會改名
const {
  executeMutation: addTodo,
  data,
  fetching,
} = useMutation(gql`
  mutation addNewTodo($newTodoInput: NewTodoInput!) {
    addTodo(input: $newTodoInput) {
      addTodo {
        id
      }
    }
  }
`);

// 變數包在 {} 中當成參數傳給 addField
// data 就是 request 成功後的回傳值，編輯、新增時都可以拿得到
addTodo({ newTodoInput }).then(({ data, error }) => {
  // 取值需要很落落長...
  // 第一個 addTodo 是 Field Name
  // 第二個 addTodo 是 addTodo 成功後回傳的內容
  console.log(data.addTodo.addTodo.id);
  // 這邊的 error 格式跟 useQuery 回傳的 error 格式相同
  console.log(error?.message);
});

// 也可以透過 async / await 取值
const addNewTodo = async () => {
  const { data, error } = await addTodo({ newTodoInput });
};
```

useMutation 跟 useQuery 大同小異，比較特別的是，有兩種方式可以取得 response：

1. 透過 `useMutation` 回傳的 data
2. 執行 function 後，透過 `.then(({ data }) => {})` 或是 `async / await` 取得

一般來說會比較推薦透過第二種方式取值。

## useSubscription

> 基於 websocket 所實作

在使用 `useSubscription` 之前，需要先做好設定，並下載相關 package。

### Setting

```javascript
import { Client, defaultExchanges, subscriptionExchange } from "urql";
import { SubscriptionClient } from "subscriptions-transport-ws";

const WEBSOCKET_URL = "wss://websocket.example/";

const subscriptionClient = new SubscriptionClient(WEBSOCKET_URL, {
  reconnect: true,
});

const client = new Client({
  url: "http://api.example/graphql",
  exchanges: [
    ...defaultExchanges,
    subscriptionExchange({
      forwardSubscription: (operation) => subscriptionClient.request(operation),
    }),
  ],
});
```

### Usage

```html
<template>
  <div v-if="error">Oh no... {{error}}</div>
  <div v-else>
    <ul v-if="data">
      <li v-for="msg in data">{{ msg.from }}: "{{ msg.text }}"</li>
    </ul>
  </div>
</template>
```

```javascript
<script>
import { useSubscription } from '@urql/vue';

export default {
  setup() {
    const handleSubscription = (messages = [], response) => {
      return [response.newMessages, ...messages];
    };

    const { data, error } = useSubscription({
      query: `
        subscription MessageSub {
          newMessages {
            id
            from
            text
          }
        }
      `
    }, handleSubscription)

    return {
      data,
      error,
    };
  }
};
</script>
```

## Conclusion

URQL 簡單介紹就到這邊告一個段落，這邊介紹到的都是一些常用到、基礎的功能，這邊大概可以涵括到實際應用場景的七八成。沒有提到的部分，包含像是 URQL 的快取機制，有分成 Document Cache 及 Normalized Caching，之後有機會再介紹給大家～

## Reference

- [URQL 官網](https://formidable.com/open-source/urql/docs/)

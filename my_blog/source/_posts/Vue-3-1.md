---
title: Vue 3 學習筆記 (一)
date: 2021-12-19 12:00:06
tags: Vue
---
此篇筆記是在 2021 年六月公司開新專案，導入 Vue 3 時所記錄下來的，主要參考 [Kuro 大的 Vue 書籍](https://book.vue.tw/)，以及各式網路資源後所彙整。適合給過去曾寫過 Vue 2 或對 Vue 有基礎了解的人閱讀，且不會探討 Vue 底層核心運作模式。

筆記主要分成兩個章節，分別為與 Vue 2 差異，以及 Vue 3 新增的 Composition API。

## 與 Vue 2 差異

### 引入方式

使用 `createApp`：

```javascript
// main.js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

// 第一種
createApp(App).use(router).mount("#app");

// 第二種
const app = createApp(App);
app.use(router);
app.mount("#app");
```
### Vue Router ( v4.x )

使用 `creatRouter`：

```javascript
// src/router/index.js
import { createRouter, createWebHashHistory } from "vue-router";
import Home from "../views/Home.vue";
import About from "../views/About.vue";

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
];

const router = createRouter({
  history: createWebHashHistory(),
  routes,
})
```

### VueX ( v4.x )
使用 `creatStore`：
```javascript
// src/store/index.js
import { createStore } from 'vuex'

const store = createStore({
  state () {
    return {
      count: 0
    }
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

```

### 不提供全域性註冊

```javascript
// vue 2.x
Vue.use(...);
Vue.directive(...);
Vue.component(...);
Vue.mixins(...);

const vm = new Vue({ ... }).$mount('#app');
```

```javascript
// vue 3.x
const vm = Vue.createApp({ ... });

vm.use(...); vm.directive(...);
vm.component(...);
vm.mixins(...);
vm.mount('#app');
```

### 新增語法

#### 1. 子 component 中新增 emit
```javascript
app.component('the-button', {
  // 增加之後提升易讀性
  emits: ['update'],
  template: '<button @click="updateMessage">Click me</button>',
  methods: {
    updateMessage() {
      this.$emit('update');
    },
  },
});
```
除此之外，也可以在 `emits` 中增加 function，處理當 emit 傳回去父層的內容，常用於驗證或是非同步操作。詳細可參考 [Vue 3 官網](https://v3.cn.vuejs.org/api/options-data.html#emits)。

```javascript
// 父層
<template>
  <Custom :obj="obj" @changeName="handleName" />
</template>

script>
import Custom from "@/components/Custom.vue";
import { reactive, ref } from "vue";

export default {
  name: "Home",
  components: {
    Custom,
  },
  setup() {
    const obj = reactive({
      name: "tom",
      age: 10,
      nickname: "apple",
    });

    const handleName = (name) => {
      obj.nickname = name;
    };

    return {
      title,
      obj,
      changeName,
    };
  },
};
</script>
```

```javascript
// 子層

<template>
  <div>
    <input v-model="nickname" />
    <button @click="updateName(nickname)">update nickname</button>
  </div>
</template>

<script>
export default {
  name: "custom",
  // 這邊可進行驗證或是非同步操作
  emits: {
    changeName: (payload) => {
      if (!payload) {
        console.log("需填寫欄位內容");
        return;
      }
      // 驗證後一定要回傳 true
      return true;
    },
  },
  setup(_, { emit }) {
    const updateName = (nickname) => {
      emit("changeName", nickname);
    };

    let nickname = ref("");

    return {
      nickname,
      props,
      updateName,
    };
  },
};
</script>
```

#### 2. teleport

> 透過 `teleport` 將類似 modal 視窗的 component 移動到外層的 DOM 節點上

視窗在有序 DOM Tree 中，應該是跟常用的 APP 平行，而不是被巢狀包裹在各個標籤中。
但如果透過 `teleport` 可以將該 component 移動到指定的父層節點下。詳情可參考 [官網](https://v3.cn.vuejs.org/guide/teleport.html#%E4%B8%8E-vue-components-%E4%B8%80%E8%B5%B7%E4%BD%BF%E7%94%A8)。

```html
<template>
  <!-- 透過 teleport 包起來，並且加上該移動去哪個父層元素下 -->
  <teleport to="body">
    <div v-if="modalOpen" class="modal">
      <div>
        I'm a teleported modal! 
        (My parent is "body")
        <button @click="modalOpen = false">
          Close
        </button>
      </div>
    </div>
  </teleport>
</template>
```
#### 3. fragment

> template 再也不需要根節點

在 Vue2 中，每個 template 都必需要有個根節點，但因為 Vue3 內部會將節點放進 fragment 虛擬元素中，因此不需要根節點也可以成功建立 template。
```html
<template>
  <h2>aaaa</h2>
  <h2>aaaa</h2>
</template>
```
當子 component 沒有根節點包住的時候，在父層元素引入 component 時，就要特別避免以下使用：
* 綁定 class
* 綁定 `v-if`
* 綁定事件

### 更改語法

#### 1. v-enter / v-leave
vue 中有一些封裝好的 component 可使用，其中 [transition](https://v3.cn.vuejs.org/guide/migration/transition.html#frontmatter-title) 的 `v-enter` 改成 `v-enter-from`，`v-leave` 改成 `v-leave-from`。
```css
.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.v-leave-from,
.v-enter-to {
  opacity: 1;
}
```

#### 2. data 必須使用函式並回傳
```javascript
data() {
  return { 
    message: 'This works in Vue 2!' 
  };
},
```
#### 3. lifecycle hooks
推薦直接參考 Kuro 大的 [文章](https://book.vue.tw/CH1/1-7-lifecycle.html#%E7%94%9F%E5%91%BD%E9%80%B1%E6%9C%9F%E8%88%87-hooks-function)

#### 4. 選取 DOM 元素
[Vue 2](https://cn.vuejs.org/v2/api/#ref) 會透過 `ref`，以及 `this.$refs.x` 存取 DOM 元素。
在 Vue 3 中因為有 composition API 的 `ref`，因此有一點不一樣：

```javascript
<template>
  // 把回傳出來的值進行綁定
  <h5 ref="dom">被選取的 DOM 元素</h5>
</template>

<script>
import { ref } from "@vue/reactivity";

export default {
  name: "custom",
  setup() {
    // 初始值是 null
    const dom = ref(null);
    // 回傳
    return {
      dom,
    };
  },
};
</script>
```

### 移除語法

#### 1. filters
原先的 option API 中有 filters 的語法，但因為與 methods 的重複性很高，在 Vue3 就被移除了

#### 2. $on、$off、$once
因為 Vue3 不推薦使用 eventbus，因此移除 eventbus 中的 $on、$off、$once。

### 其他特性
1. 支援 JSX
2. 支援 TypeScript

### 注意事項
* 應避免 $ 或是 _ 開頭命名的變數名稱

## composition API
引入方式：
```javascript
import { ref, reactive, watch, computed } from 'vue';
```
### setup
特性：
* `setup` 相當於 `beforecreate` 跟 `created` 這兩個 lifecycle hooks。
* `setup` 接受兩個參數，第一個是 props，第二個是 context，context 是一個 object，包含 attrs, slots, emit。因此也可以對 context 進行解構。
* 當今天不需要 props 只需要 emit 時，可以將第一個參數變成 `_`，vue 會自動忽略第一個參數。
* 如果今天有傳 props 時，推薦在 `setup()` 中把 props 當參數引入再 return 出去，這樣在 template 中引用時就可以透過 `props.x` 操作。未來在看 template 時可以清楚知道哪些是 `props` 哪些是 `state`。

```javascript
setup(props, context) {
  // 存取 attrs 時需要透過 . 存取
  console.log(context.attrs);
  console.log(context.slots);

  // 把 props 傳出去
  return {
    props,
  }
}

// context 可解構
setup(props, { emit }) {}

// props 用不到可以用 _
setup(_, { emit }) {}
```

以下講解的 `ref`、`reactive`、`watch`、`watchEffect`、`computed` 都會放在 `setup` 裡面。

### ref
資料綁定。`ref` 習慣會綁定的資料是原始型別，綁定後要存取資料時會透過 `.value` 進行存取。

### reactive
資料綁定。`reactive` 習慣會綁定的資料是物件型別，綁定後 **不須** 透過 `.value` 進行存取。

### watch
```
watch(監聽資料, callback fn(newValue, oldValue));
```
資料監聽。第一個參數是要監聽的資料，第二個參數是 callback function。
特別注意的是：
* 如果要監聽的是 `ref` 的值，可直接將值放入。如果是要監聽 `reactive` 中某個 value，需要透過 function return reactive 的值回去。
* 第二個參數的 callback function 中，第一個參數是改動後的新值，第二個參數是舊值。
* 一次監聽多個值時，第一個參數改成 array，並放入要監控的值。

範例：
```javascript
setup() {
  const text = ref('');
  const goals = reactive({
    a: 1,
    b: 2,
  });

  watch(text, (newValue, oldValue) => {
    console.log(newValue, oldValue)
  })
  
  // 監聽 goals 所有的值
  watch(goals, (newValue, oldValue) => {
    console.log(newValue, oldValue)
  })

  // 只監聽 goals 的 value
  watch(() => goals.a, (newValue, oldValue) => {
    console.log(newValue, oldValue)
  })

  // 監聽多個值
  watch([text, () => goals.a], (newValue, oldValue) => {
    console.log(newValue, oldValue)
  })

  return {
    text,
    goals,
  }
}
```
### watchEffect
實際上很少用到，跟前面提到的 `watch()` 有點像，不同的是不需要加入參數，而是直接監聽 `watchEffect()` 中有存取到的值，有更新時會執行當中的扣。

一般值在初始化的時候會執行一次，更新時會再執行一次。

```javascript=
// 只有當 text 改變時才會印出
watchEffect(() => console.log(text))
```

### computed
跟原本的 vue 2 概念相同。
* 傳入一個 function 的時候表示會回傳一個 readonly 的 `ref` 值。

```javascript
const computedValue = computed(fn);
```
* 傳入一個 object 的時候，可設定 getter 跟 setter。

```javascript=
const computedValue = computed({
  get() {
    ...
  },
  set() {
    ...
  },
})
```
範例：

```javascript
setup() {
  const goals = reactive([]);

  const computedValu = computed(() => {
    return goals.filter(
        (goal) => !goal.text.includes("Angular") && !goal.text.includes("React")
      );
  });
  
  // 記得 computed 後的值要用 .value 存取
  onMounted(() => console.log(computedValue.value));

  return {
    goals,
    computedValue,
  }
}
```

### custom hooks

```javascript
// src/hooks/useMouse.js

import { ref, onMounted, onUnmounted } from "vue";

export default function () {
  let x = ref(0);
  let y = ref(0);

  const handleMouse = (e) => {
    // 記得使用 .value 賦值
    x.value = e.pageX;
    y.value = e.pageY;
  };

  onMounted(() => {
    window.addEventListener("mousemove", handleMouse);
  });

  onUnmounted(() => {
    window.removeEventListener("mousemove", handleMouse);
  });

  return {
    x,
    y,
  };
}
```
```javascript
<template>
  <div>
    <div>x: {{ x }}</div>
    <div>y: {{ y }}</div>
  </div>
</template>

<script>
import useMouse from "@/hooks/useMouse";

export default {
  name: "custom",
  setup() {
    const { x, y } = useMouse();
    // 這邊記得引入後再引出
    return {
      x,
      y,
    };
  },
};
</script>
```
#### toRefs
這邊要特別注意，以上的範例是使用 `ref`，但是當如果使用 `reactive` 的話，狀況會有點不太一樣，會需要多使用 `toRefs`：
```javascript
// src/hooks/useMouse.js

import { reactive, toRefs, onMounted, onUnmounted } from "vue";

export default function () {
  // 使用 reactive
  let location = reactive({
    x: 0,
    y: 0,
  });

  const handleMouse = (e) => {
    location.x = e.pageX;
    location.y = e.pageY;
  };

  onMounted(() => {
    window.addEventListener("mousemove", handleMouse);
  });

  onUnmounted(() => {
    window.removeEventListener("mousemove", handleMouse);
  });

  // 要使用 toRefs 包起來後回傳
  // 如果只有一個回傳值，直接 `return toRefs(location);` 也可以
  return {
    ...toRefs(location),
  };
}
```
接收 `toRefs(xxx)` 的值時，記得也要透過 `.value` 存取。

### 改寫 VueX
使用 Composition API 後，因為資料有共通性，因此有些人推薦以 Composition API 來取代繁雜的 VueX。

[Kuro 大](https://github.com/kurotanshi/mask-map-demo/blob/vite-demo/src/composition/store.js) 就曾經示範。在 Vite-demo 分支中，是使用 Composition API 共享資料，在 master 分支則是採用原 VueX 寫法。

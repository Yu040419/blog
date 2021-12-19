---
title: Vue 3 學習筆記 (二)
date: 2021-12-19 21:45:06
tags: Vue
---
此篇筆記為 Vue 相關筆記第二篇，建議可先閱讀 [第一篇](https://yu040419.github.io/blog/article/Vue-3-1/)，對 Vue 3 基礎較為熟悉後，再閱讀會更好理解。
筆記部分參考 VueConf China 2021 中，VueUse 作者分享。

## script setup
Vue 3.2 出的新功能中，包含了 [script setup](https://v3.cn.vuejs.org/api/sfc-script-setup.html#%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95) 這個語法糖，包含以下幾個特點：
* 更簡潔的 Code
* 優化執行效能
* 對 TypeScript 的支援度更好

### 再也不需要 return
在過去 Composition API 中，`template` 若要使用任何 data，都會需要在 `setup` 中將資料 `return` 出來。

但使用 script setup 的語法後，引入的 state / function 可以立即使用，不需要 `return`，十分方便。

### 提升 TypeScript 支援
script setup 同時新增幾個語法，來提昇 TypeScript 支援。分別是：

*  `defineProps`：定義 `props` 的型別
*  `defineEmits`：定義 `emits` 的型別
*  `withDefaults`：搭配 `defineProps` 賦予 `props` 預設值。

### defineExpose
因為 `script setup` 默認不將 component 內的 data 及 method 暴露出去。所以當如果想要在父層使用子層 dom 元素時，就可以在子 component 將該 ref 值透過 `defineExpose()` 暴露出去。

### 範例
可搭配以上特色服用：

```javascript=
// 原先 Vue 3 版本
<script lang="ts">
import { defineComponent, computed } from "vue";
import { useI18n } from "vue-i18n";

export default defineComponent({
  name: "BaseInput",
  props: {
    isRequired: {
      type: Boolean,
      default: false,
    },
    error: {
      type: String,
    },
    title: {
      type: String,
    },
    label: {
      type: String,
      required: true,
    },
    val: {
      required: true,
    },
    type: {
      type: String,
      required: true,
    },
  },
  setup(props, { emit }) {
    const { t } = useI18n();
    const inputVal = computed({
      get: () => props.val,
      set: (val) => emit("update:val", val),
    });
    
    return {
      props,
      inputVal,
      t,
    };
  },
```
使用 `script setup` 後：

```javascript=
<script lang="ts" setup>
import { computed, defineProps, defineEmits, withDefaults } from "vue";
import { useI18n } from "vue-i18n";

interface Props {
  isRequired?: boolean;
  error?: string;
  title?: string;
  label: string;
  val: string | number;
  type: string;
}
  
const props = withDefaults(defineProps<Props>(), {
  isRequired: false,
});
const emit = defineEmits<{ (e: "update:val", val: string | number): void }>();

const { t } = useI18n();
const inputVal = computed({
  get: () => props.val,
  set: (val) => emit("update:val", val),
});
</script>
```

## ref
`ref` 一般常用於 primitive type 的狀態使用，會在 `ref()` 中，放入 primitive type，像是 boolean, string, number 等等。

但其實，`ref` 本身也可以接受 `ref` 值，兩者返回的值，也完全相同。
```javascript=
const a = ref(1);
const b = ref(a);

console.log(a === b); // true
```
這常用於 TypeScript 中，當今天無法確定參數的型別時，可以很方便地處理：
```javascript=
function useTitle(title: Ref<string> | string | null) {
  const title = isRef(title) ? title : title ? : ref(title) : 'title';

  // 不需要做以上一連串判斷，可以直接將 ref 值放進來
  const title = ref(title || 'title');
  
  document.title = title;
  return title;
}
```
也因為完全相同的關係，使用上也需要特別注意，在改動其中一個 ref 值時，另一個 ref 值也會被一起更動。

## unref

[unref](https://v3.vuejs.org/api/refs-api.html#unref) 是官方原生的 API，是一種語法糖，將以下用法包裹起來：
```javascript=
function unref(val) {
  return isRef(val) ? val.value : val;
}
```
`unref` 會先用一樣是官方原生的 `isRef` API 判定是否是 Ref，如果是的話，就將 ref 值回傳回去，如果不是的話，就直接回傳這個值。

簡單用法就如同以下，透過 unref 可以確保一定拿得到該值：
```javascript=
const data = unref(value);
```
如果搭配 TypeScript，實用性也會更高：
```typescript=
// 接受 ref 值或是純數字
function add(
  a: Ref<number> | number,
  b: Ref<number> | number
) {
  // 這邊就不需要再做多一層型別判斷
  return computed(() => unref(a) + unref(b));
}

```
補充，也可以搭配自定義的 `MaybeRef` 型別，使用上可以更簡潔：
```typescript=
type MaybeRef<T> = Ref<T> | T

function add(
  a: MaybeRef<number>,
  b: MaybeRef<number>
) {
  return computed(() => unref(a) + unref(b));
}
```
## computed
一般我們會透過非同步的方式取得資料後，再將取得的資料賦值給 state 或 store。但也可以先將資料透過 computed 的方式串連起來，概念跟 React 的 SWR 相似。

```javascript=
// call API 取得資料後，再透過 computed 賦值
const { data } = useFetch('https://api.github.com/').json()

const user_url = computed(() => data.value?.user_url)
```

## InjectionKey<T>
透過 `provide` 跟 `inject` 可以避免 props drilling 的問題，但對型別的支援卻不友善。這時可以透過 `InjectionKey` 來解決型別問題。

```typescript=
// context.ts 共用 state 檔案
import { InjectionKey } from 'vue'

export interface UserInfo {
  id: number
  name: string
}

export const injectKeyUser: InjectionKey<UserInfo> = Symbol()
export const injectKeyIsBoolean: InjectionKey<boolean> = Symbol();
```
    
```typescript=
// parent.vue
// 在父元件 provide 值
import { provide } from 'vue' 
import { injectKeyUser, injectKeyIsBoolean } from './context'

export default {
  setup() {
    provide(injectKeyUser, {
      id: '7', // error 型別錯誤
      name: 'Yu'
    });
    
    provide(injectKeyIsBoolean, true); // 檢查通過
  }
}
```

```typescript=
// child.vue
// 在子元件 inject 值
import { inject } from 'vue' 
import { injectKeyUser } from './context'

export default {
  setup() {
    const user = inject(injectKeyUser) 
    // UserInfo | undefined
    // 因為有可能不被 provide，所以有可能是 undefined

    if (user)
      console.log(user.name) // Yu
  }
}
```
## v-model
當今天子組件接收父組件的 props 綁定 input 時，可以透過 `computed` 搭配 `v-model` 達成簡易快速的 `emit`。

```typescript=
// parent.vue

<template>
    // 這邊記得要加上 v-model
    <ChildComponent v-model:val="value" />
</template>

<script setup>
const value = ref('test');
</script>
```
    

```typescript=
// ChildComponent.vue

<template>
    // 綁定 v-model
    <input v-model.lazy="inputVal" />
</template>

<script setup>
// 宣告 props
const props = defineProps<{ val: string; }>();

// 透過 computed 達成快速 emit
const inputVal = computed({
  get: () => props.val,
  set: (val) => emit("update:val", val),
});
</script>
```

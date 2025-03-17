---
layout: post
title:  "Vue3学习之如何引入Pinia"
categories: vue
tags: Javascript Vue
---

* content
{:toc}

Vue3学习之如何引入Pinia







## 一、引入Pinia

首先，需要安装 Pinia：

```bash
npm install pinia
```

## 二、创建并配置Store

在 `src` 文件夹下创建 `store` 文件夹和 `index.ts` 文件。然后，在 `main.ts` 中引用 Pinia：

```typescript
import { createPinia } from 'pinia';
const store = createPinia();
createApp(App).use(router).use(store).mount('#app');
```

在 `index.ts` 中使用 Pinia，通过 `defineStore API` 创建 Store，传入的第一个参数是唯一的，作为该 Store 的唯一标识符：

```typescript
import { defineStore } from 'pinia';

export const studyStore = defineStore("getNum", {
  state: () => {
    return { num: 1 as number };
  },
  getters: {},
  actions: {}
});
```

如果项目中有多个 Store，可以在 store 文件夹下创建 type 文件夹，用于存放 state 中的各种类型。

## 三、调用和修改 State 中的值

调用Store中的值         

```vue
<script setup lang="ts">
import { useUserStore } from '@/stores/user';
import { computed } from 'vue';
import { storeToRefs } from 'pinia';

// 方式1, 计算属性方式
const userid = computed(() => useUserStore().userid);

// 方式2, 直接使用
const user = useUserStore();

// 方式3, 使用 toRef 获取 userid
const userid = toRef(useUserStore(), 'userid');

// 方式4, 使用 storeToRefs 实现
const { userid } = storeToRefs(user);
</script>
```

修改State中的值

* 方法1: 直接修改
  
```vue
<template>
  <div>
    <span>data:{{ test.num }}</span>
    <button @click="add">数字++</button>
  </div>
</template>

<script setup lang="ts">
import { studyStore } from "../store";
const test = studyStore();
let add = () => {
  test.num++;
}
</script>
```

* 方法2: 使用 $patch 函数修改

```typescript
test.$patch({ num: 211 });
```

或者

```typescript
test.$patch((state) => {
  state.num++;
});
```

* 方法3：使用 action 修改

```typescript
actions: {
  addNum() {
    // 注意不要使用箭头函数，否则会影响 this 指向
    this.num++;
  }
}

let add = () => {
  test.addNum();
}
```

* 方法4: 使用 getter

```typescript
getters: {
  getNewNum():number{
    return this.num;
  }
}
```

或者

```typescript
getters: {
  getNewNum:(state) => {
    return state.num;
  }
}
```

在模板中使用：

```vue
<template>
  <div>
    <span>data:{{ test.num }}</span>
    <span>newData:{{ test.getNewNum }}</span>
    <button @click="add">数字++</button>
  </div>
</template>
```


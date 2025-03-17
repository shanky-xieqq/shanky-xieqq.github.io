---
layout: post
title:  "Nuxt3 整合 WangEditor"
categories: vue
tags: Javascript Vue
---

* content
{:toc}

在开发过程中，我们经常需要使用富文本编辑器来提升用户体验。本文将介绍如何在 Nuxt3 项目中整合 WangEditor 富文本编辑器。







## 1. 创建插件文件

首先，在 `plugins` 目录下创建一个名为 `wang-editor.ts` 的文件，并添加以下内容：

```typescript
import { Editor, Toolbar } from '@wangeditor/editor-for-vue';

export default defineNuxtPlugin((nuxt) => {
    nuxt.vueApp.component('WeEditor', Editor);
    nuxt.vueApp.component('WeToolbar', Toolbar);
});
```

## 2. 关闭 SSR

由于 WangEditor 是一个客户端组件，我们需要在 `nuxt.config.ts` 中关闭 SSR（服务端渲染）。请注意，旧版本中使用 `ssr: false` 已被弃用，应参考官方文档进行配置。                

```typescript
export default defineNuxtConfig({
    plugins: [
        { src: '~/plugins/wang-editor', mode: 'client' },
    ],
});
```

### 3. 创建 WangEditor 组件

在 `components` 目录下创建一个新的组件文件，例如 `WangEditorComponent.vue`，并添加以下内容：            

```vue
<template>
    <div style="border: 1px solid #ccc; margin-top: 10px">
        <WeToolbar :editor="editorRef" :defaultConfig="toolbarConfig" :mode="mode" />
        <WeEditor 
            v-model="valueHtml"
            :defaultConfig="editorConfig" 
            :mode="mode"
            @onCreated="handleCreated"
            @onChange="handleChange"
            @onDestroyed="handleDestroyed"
            @onFocus="handleFocus"
            @onBlur="handleBlur"
            @customAlert="customAlert"
            @customPaste="customPaste" />
    </div>
</template>

<script setup lang="ts">
import '@wangeditor/editor/dist/css/style.css'; // 引入 css
import { onBeforeUnmount, ref, shallowRef, onMounted } from 'vue';
import { Editor, Toolbar } from '@wangeditor/editor-for-vue';

const mode = 'default';
const isClient = ref(false);

if (process.client) {
    isClient.value = true;
}

// 编辑器实例，必须用 shallowRef
const editorRef = shallowRef();

// 内容 HTML
const valueHtml = ref('<p></p>');

// 模拟 ajax 异步获取内容
onMounted(() => {
    setTimeout(() => {
        valueHtml.value = '<p>模拟异步获取的内容</p>';
    }, 1500);
});

const toolbarConfig = {};
const editorConfig = {
    placeholder: '请输入文章内容...'
};

// 组件销毁时，也及时销毁编辑器
onBeforeUnmount(() => {
    const editor = editorRef.value;
    if (editor == null) return;
    editor.destroy();
});

const handleCreated = (editor: any) => {
    editorRef.value = editor; // 记录 editor 实例，重要！
};

const handleChange = (editor: any) => {
    console.log('change:', editor.getHtml());
};

const handleDestroyed = (editor: any) => {
    console.log('destroyed', editor);
};

const handleFocus = (editor: any) => {
    console.log('focus', editor);
};

const handleBlur = (editor: any) => {
    console.log('blur', editor);
};
</script>
```

### 4. 使用 WangEditor 组件

在您的页面或组件中引入并使用刚刚创建的 `WangEditorComponent.vue` 组件：         

```vue
<template>
    <div>
        <WangEditorComponent />
    </div>
</template>

<script setup lang="ts">
import WangEditorComponent from '@/components/WangEditorComponent.vue';
</script>
```

### 总结

通过上述步骤，您可以成功地在 Nuxt3 项目中整合 WangEditor 富文本编辑器。以下是关键步骤总结：

1. 创建插件文件：在 `plugins` 目录下创建 `wang-editor.ts` 文件。            
2. 关闭 SSR：在 `nuxt.config.ts` 中设置插件为客户端模式。           
3. 创建 `WangEditor` 组件：在 components 目录下创建新的组件文件，并添加必要的逻辑和样式。           
4. 使用 `WangEditor` 组件：在页面或组件中引入并使用新创建的组件。           

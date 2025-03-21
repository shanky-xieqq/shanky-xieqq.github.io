---
layout: post
title:  "Vue3.0路由与导航攻略"
categories: Vue
tags: Vue
---

* content
{:toc}

在现代前端开发中，单页应用（SPA）已经成为主流趋势。而作为 Vue.js 的核心功能之一，Vue Router 提供了强大的路由管理能力，帮助开发者轻松构建流畅、高效的单页应用。本文将带你深入探讨 Vue3 中的路由配置与导航操作，从安装到实战，手把手教你掌握 Vue Router 的使用技巧。








一、为什么需要Vue Router？
------------------

在单页应用（SPA）中，前端路由负责动态管理视图切换，避免页面刷新带来的性能损耗。Vue3官方推荐使用**Vue Router 4.x**实现这一能力，它具备以下优势：

*   **无缝集成**：与Vue3响应式系统深度绑定
*   **灵活配置**：支持动态路由、嵌套路由、导航守卫等高级特性
*   **多模式支持**：HTML5 History/Hash路由模式自由选择

二、Vue Router 的安装与初始化
--------------------

### 2.1 安装 Vue Router

在开始之前，确保你的项目已经初始化为 Vue3 项目。如果尚未安装 `vue-router`，可以通过以下命令安装最新版本：

```bash
npm install vue-router@next
```

安装完成后，在项目的 `src` 目录下创建一个 `router` 文件夹，并在其中新建 `index.js` 文件用于配置路由。

### 2.2 配置路由实例

#### 1\. 推荐项目结构：

```
src/
├── router/
│   └── index.js  # 路由主文件
└── views/        # 路由组件目录
```

接下来，我们需要在 `index.js` 文件中创建路由实例并定义路由规则。以下是完整的代码示例：

```javascript
import { createRouter, createWebHistory } from 'vue-router'

// 1. 导入路由组件（推荐懒加载）
const Home = () => import('@/views/Home.vue')
const About = () => import('@/views/About.vue')

// 2. 定义路由规则
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: { title: '首页' }  // 路由元信息
  },
  {
    path: '/about',
    name: 'About',
    component: About,
    meta: { title: '关于我们' }
  }
]

// 3. 创建路由实例
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL), // 使用History模式
  routes,
  scrollBehavior(to, from, savedPosition) {  // 滚动行为控制
    return savedPosition || { top: 0 }
  }
})

// 4. 全局路由守卫示例
router.beforeEach((to, from) => {
  document.title = to.meta.title || '默认标题'
})

export default router
```

#### 2\. 关键配置解析：

*   **`routes` 数组** ：定义了路由映射关系，每个对象包含 `path（路径）`、`name（路由名称）` 和 `component（对应的组件`）
    
*   **`createWebHistory`**：使用HTML5 History模式（需要服务器支持）
    
*   **`createWebHashHistory`**：使用Hash模式（URL带#号，兼容性好）
    
*   **路由懒加载**：通过`() => import()`提升首屏加载速度
    
*   **`scrollBehavior`**：控制页面切换时的滚动位置
    

三、在 Vue 应用中挂载路由
---------------

### 3.1 全局挂载路由

在完成路由配置后，需要将其挂载到 Vue 应用中。打开 `main.js` 文件，添加以下代码：

```javascript
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);
app.use(router);
app.mount('#app');
```

#### 关键点：

*   `app.use(router)` ：将路由实例挂载到 Vue 应用中，使得整个应用可以使用路由功能
    
*   `router-view` ：在模板中使用 `<router-view>` 标签来渲染匹配的组件
    

### 3.2 路由出口组件

在根组件`App.vue`中：

```html
<template>
  <div class="app-container">
    <nav class="global-nav">
      <router-link 
        to="/" 
        active-class="active-link"
        exact
      >首页</router-link>
      
      <router-link 
        :to="{ name: 'About' }"
        custom
        v-slot="{ navigate }"
      >
        <button @click="navigate">关于我们</button>
      </router-link>
    </nav>
    
    <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" />
      </transition>
    </router-view>
  </div>
</template>

<style>
.active-link {
  color: #42b983;
  font-weight: bold;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

#### 技术亮点：

*   **`active-class`**：自定义激活状态的CSS类名
    
*   **`exact`**：精确匹配路由
    
*   **`v-slot`**：自定义导航渲染方式
    
*   **过渡动画**：通过`<transition>`实现页面切换动画
    

四、路由导航方式详解
----------

### 4.1 声明式导航：使用 `<router-link>`

在 Vue 中，最常用的导航方式是通过 `<router-link>` 组件。它会生成一个超链接，点击后触发路由跳转。

```html
<template>
  <div>
    <router-link to="/">Home</router-link>
    <router-link to="/about">About</router-link>
    <router-view></router-view>
  </div>
</template>
```

#### 特性：

*   `to` 属性 ：指定目标路径
    
*   默认样式 ：`<router-link>` 会自动为当前激活的链接添加 `active` 类名，方便开发者自定义样式
    

### 4.2 编程式导航全解析

#### 1\. 基础导航方法

```javascript
// 在组合式API中使用
import { useRouter } from 'vue-router'

const router = useRouter()

// 字符串路径
router.push('/about')

// 带参数的对象形式
router.push({ path: '/user/123' })

// 命名路由+参数
router.push({ 
  name: 'User',
  params: { id: 123 }
})

// 替换当前路由（无历史记录）
router.replace('/login')

// 前进/后退
router.go(1)  // 前进1步
router.back() // 等同于 router.go(-1)
```

#### 2\. 动态路由实战

定义带参数的路由：

```javascript
{
  path: '/user/:id',
  name: 'User',
  component: () => import('@/views/User.vue')
}
```

在组件中获取参数：

```javascript
import { useRoute } from 'vue-router'

const route = useRoute()
console.log(route.params.id)  // 输出URL中的id值
```

#### 3\. 嵌套路由

嵌套路由适用于多层结构的页面布局。例如，一个用户中心页面可能包含多个子页面：

```javascript
const routes = [
  {
    path: '/user',
    component: UserLayout,
    children: [
      { path: 'profile', component: UserProfile },
      { path: 'settings', component: UserSettings }
    ]
  }
];
```

在父组件中，使用 `<router-view> `渲染子路由。

#### 4\. 导航守卫进阶

**常见守卫**：

*   全局守卫 ：`beforeEach`、`afterEach`
    
*   组件内守卫 ：`beforeRouteEnter`、`beforeRouteLeave`
    

```javascript
// 全局前置守卫
router.beforeEach((to, from) => {
  if (to.meta.requiresAuth && !isAuthenticated) {
    return { path: '/login' }
  }
})

// 路由独享守卫
{
  path: '/admin',
  component: Admin,
  beforeEnter: (to, from) => {
    // 权限校验逻辑
  }
}

// 组件内守卫
onBeforeRouteLeave((to, from) => {
  const answer = window.confirm('确定要离开吗？')
  if (!answer) return false
})
```

五、性能优化技巧
--------

### 5.1 路由懒加载：

```javascript
const User = () => import(/* webpackChunkName: "user" */ '@/views/User.vue')
```

### 5.2 路由组件预加载：

```javascript
// 在用户hover导航链接时预加载
<router-link 
  to="/about" 
  @mouseenter="preloadAbout"
></router-link>

<script setup>
const preloadAbout = () => {
  import('@/views/About.vue')
}
</script>
```

### 5.3 路由分割策略：

```j
dist/
├── js/
│   ├── main.js
│   ├── home.js     # 首页路由代码
│   └── about.js    # About页路由代码
```

六、常见问题排查
--------

### 问题1：页面刷新后404

#### 解决方案：

*   History模式需要服务器配置Fallback
    
*   Nginx示例配置：
    

```bash
location / {
  try_files $uri $uri/ /index.html;
}
```

### 问题2：路由参数不更新

#### 解决方法：

在组件中监听路由变化：

```javascript
watch(
  () => route.params.id,
  (newId) => {
    // 重新获取数据
  }
)
```

七、最佳实践总结
--------

1.  路由分层管理：大型项目采用**模块化路由**
    
2.  路由元信息：通过`meta`字段存储权限标识
    
3.  异常处理：配置全局错误路由
    

```javascript
{
  path: '/:pathMatch(.*)*',
  redirect: '/404'
}
```

4.  类型安全：配合TypeScript使用路由类型提示

```javascript
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [...]
```
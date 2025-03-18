---
layout: post
title:  "Echarts与Vue3中获取DOM节点可能出现的异常错误"
categories: Vue
tags: Vue Echart
---

* content
{:toc}

在开发的过程中遇到此问题，比较有意思，记录一下。






#### useTemplateRef 的简单介绍

官方:返回一个浅层 ref，其值将与模板中的具有匹配 ref attribute 的元素或组件同步。  
参数匹配机制‌：useTemplateRe的参数需与模板中 ref 属性值必须完全一致  
‌响应式变量类型明确‌：返回值是一个 浅层 ref对象，其 .value 直接指向绑定的 DOM 元素或组件实例。

#### useTemplateRef源码浅析

packages/runtime-core/src/helpers/useTemplateRef.ts 文件中

```typescript
import { type ShallowRef, readonly, shallowRef } from '@vue/reactivity'
import { getCurrentInstance } from '../component'
import { warn } from '../warning'
import { EMPTY_OBJ } from '@vue/shared'
export function useTemplateRef(key) {
  // 获取当前 Vue 实例对象。
  const i = getCurrentInstance()
  // 创建一个浅层的ref对象r，初始值为null。 
  const r = shallowRef(null)
  //如果存在 Vue 实例
  if (i) {
    // i.refs 默认初始化为 EMPTY_OBJ（空对象）首次使用时动态创建新对象‌
    const refs = i.refs === EMPTY_OBJ ? (i.refs = {}) : i.refs
    Object.defineProperty(refs, key, {
        enumerable: true,
        get: () => r.value,
        set: val => (r.value = val),
      })
  }
  return r
}
```

1.获取当前 Vue 实例对象。  
2.创建一个浅层的ref对象r，初始值为null。  
3.如果存在 Vue 实例，则获取实例上的refs属性（用于存储模板引用）。  
4.使用Object.defineProperty对refs对象的key属性进行拦截：  
get拦截：返回r.value，即useTemplateRef返回的ref变量的值。  
set拦截：将val赋值给r.value，从而将 DOM 元素或组件实例绑定到ref变量上。  
5.返回ref对象r。

#### 下面这段代码看看有啥问题

```xml
<template>
  <div>
    <div class="pic" ref="chartNode"></div>
  </div>
</template>
<script setup>
import * as echarts from 'echarts'
import { onMounted, useTemplateRef } from 'vue'
// useTemplateRef接受的是一个字符串，表示你要获取哪一个节点
const divNode = useTemplateRef("chartNode");
console.log(1111, divNode)
// 初始化图表
let chartDom = echarts.init(divNode);
onMounted(() => {
  drawCharts()
})
const drawCharts=()=>{
  // 指定图表的配置项和数据
  let option = {
    xAxis: {
      type: 'category',
      data: ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
    },
    yAxis: {
      type: 'value'
    },
    tooltip: {
      trigger:'axis',
    },
    series:[
      {
        data: [1150, 200, 300, 356, 105, 200, 345],
        type: 'line'
      }
    ]
  };
  // 使用刚指定的配置项和数据显示图表。
  chartDom.setOption(option);
}
</script>
<style scoped>
.pic{
  width: 600px;
  height: 380px;
}
</style>
```

![Image 12](https://img2024.cnblogs.com/blog/1425695/202503/1425695-20250303091511106-1317487382.jpg)

#### 实际情况:this.dom.getContext is not a function

报错信息: Uncaught (in promise) TypeError: this.dom.getContext is not a function  
01-echarts引入报错.jpg  
为啥会报这个错误呢?  
原因:因为在初始化echarts的时候，echarts规定只能传入实际的DOM元素。  
此时我们传递的是Ref对象,而不是实际的DOM元素  
需要传递 divNode.value。

#### 传递一个实际的DOM元素(divNode.value)

```javascript
const divNode = useTemplateRef("chartNode");
console.log(1111, divNode)
// 初始化图表，传递实际DOM
let chartDom = echarts.init(divNode.value);
onMounted(() => {
  drawCharts()
})
const drawCharts=()=>{
  ...代码爆出不变
}
```

![Image 13](https://img2024.cnblogs.com/blog/1425695/202503/1425695-20250303091522065-1341766319.jpg)

#### 为啥还会报错:Error: Initialize failed: invalid dom.

原因在于：let chartDom = echarts.init(divNode.value);这一行代码。  
此时divNode.value为null。  
为啥是null呢？  
这个跟获取时机有关:此时还没有完成绑定哈，所以得到的是null。  
什么时候可以正常绑定呢？  
第1种：在onMounted中肯定是绑定了，此时divNode.value是一个实际的DOM元素了。  
第2种：与它同一级的DOM元素已经渲染完成(其实也是在onMounted)

#### 使用useTemplateRef正常渲染图表

```xml
<template>
  <div>
    <div class="pic" ref="chartNode"></div>
  </div>
</template>

<script setup>
import * as echarts from 'echarts'
import { onMounted, useTemplateRef } from 'vue'
let chartDom = null //存储的是 ECharts 实例
// useTemplateRef接受的是一个字符串，表示你要获取哪一个节点
const divNode = useTemplateRef("chartNode");
console.log(222, divNode)
onMounted(() => {
  // 在onMounted中初始化图表，可以得到原生的DOM节点
  chartDom = echarts.init(divNode.value);
  // 调用图表
  drawCharts()
})
const drawCharts=()=>{
  // 指定图表的配置项和数据
  let option = {
    xAxis: {
      type: 'category',
      data: ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
    },
    yAxis: {
      type: 'value'
    },
    tooltip: {
      trigger:'axis',
    },
    series:[
      {
        data: [1150, 200, 300, 356, 105, 200, 345],
        type: 'line'
      }
    ]
  };
  // 使用刚指定的配置项和数据显示图表。
  chartDom.setOption(option);
}
</script>
<style scoped>
.pic{
  width: 600px;
  height: 380px;
}
</style>
```

![Image 14](https://img2024.cnblogs.com/blog/1425695/202503/1425695-20250303091529272-1886996341.jpg)

#### DOM元素已经渲染完成,useTemplateRef拿到DOM元素

```xml
<template>
  <div class="art-page">
    <div ref="divNode">我是div元素</div>
    <button @click="getNodeHandler">我是按钮，获取div节点</button>
  </div>
</template>
<script setup>
import { useTemplateRef } from 'vue'
const divNode = useTemplateRef("divNode");
const getNodeHandler= () =>{
  /**
   * 为啥这里的 divNode.value不是null
   * 因为：与它同一级的DOM元素已经渲染完成。
   * 也就是说:点击的时候已经渲染完成了。也就完成了DOM的绑定，因此可以拿到DOM元素
  */
  if(divNode.value){
    divNode.value.innerText = '通过dom来赋值';
  }
}
</script>
```

![Image 15](https://img2024.cnblogs.com/blog/1425695/202503/1425695-20250303091537654-1725939369.png)

当然我们除了使用 useTemplateRef 来获取DOM元素  
还可以使用 ref 来获取DOM元素

#### 使用 ref 来获取DOM元素

使用ref 来获取DOM元素需要注意的点。  
通过ref函数创建，并赋值给与模板中同名的变量。

```html
<div class="pic" ref="chartNode"></div> 
<script setup>
// 通过ref函数创建，并赋值给与模板中同名的变量。
const chartNode = ref(null)
</script>
```

#### 错误的获取DOM节点的方式

```html
<div class="pic" ref="chartNode"></div> 
<script setup>
// 这一种是创建了一个响应式的对象。并不是获取DOM节点。
const node = ref('chartNode')
</script>
```

#### 使用 ref 来获取DOM元素，并渲染echarts

```xml
<template>
  <div>
    <div class="pic" ref="chartNode"></div>
  </div>
</template>

<script setup>
import * as echarts from 'echarts'
import { onMounted,ref } from 'vue'
let chartDom = null //存储的是 ECharts 实例
let chartNode  = ref() // 通过ref函数创建，并赋值给与模板中同名的变量。
onMounted(() => {
  // 在onMounted中初始化图表，可以得到原生的DOM节点
  chartDom = echarts.init(chartNode.value);
  // 调用图表
  drawCharts()
})
const drawCharts=()=>{
  // 指定图表的配置项和数据
  let option = {
    xAxis: {
      type: 'category',
      data: ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
    },
    yAxis: {
      type: 'value'
    },
    tooltip: {
      trigger:'axis',
    },
    series:[
      {
        data: [1150, 200, 300, 356, 105, 200, 345],
        type: 'line'
      }
    ]
  };
  // 使用刚指定的配置项和数据显示图表。
  chartDom.setOption(option);
}
</script>
<style scoped>
.pic{
  width: 600px;
  height: 380px;
}
</style>
```

#### 存储的是ECharts实例变成响应式数据出现的问题

```xml
<template>
  <div class="chart-page">
    <div class="pic" ref="chartNode"></div>
  </div>
</template>
<script setup>
import * as echarts from 'echarts'
import { onMounted,ref, onBeforeUnmount  } from 'vue'
// 存储 echarts 实例的变量
et chartDom = ref(null)
let chartNode  = ref()
onMounted(() => {
  // 存储的是 ECharts 实例此时是一个响应式的
  chartDom.value = echarts.init(chartNode.value);
  // 调用图表
  drawCharts()
  // 监听窗口大小变化，重绘图表
  window.addEventListener('resize', chartRedraw)
})
onBeforeUnmount(()=>{
  window.removeEventListener('resize', chartRedraw)
})
const drawCharts = () => {
  let option = {
    ...配置不变
  }
  chartDom.value.setOption(option);
}
const chartRedraw = ()=> {
  chartDom.value && chartDom.value.resize()
}
</script>
```

![Image 16](https://img2024.cnblogs.com/blog/1425695/202503/1425695-20250303091549070-541678117.jpg)

#### 缩放echarts出现:Cannot read properties of undefined (reading 'type')

缩放窗口大小，echarts图表出现报错信息如下：  
Cannot read properties of undefined (reading 'type')  
原因是：存储的是 ECharts 实例变成了响应式,从而在resize 的时候获取不到。  
其实存储 ECharts 实例不应该是一个响应式的数据。  
就是一个普通类型的数据就行

#### 解决办法

现在我们知道出现问题的原因。  
解决办法就是不让它变成响应式的数据就行。  
1:存储的是 ECharts实例不要变成响应式,让其成为普通对象。  
2.使用markRaw将它标记为一个对象,使其不会被转换为响应式对象  
第1种我们已经使用过了，下面我们演示第2种

#### Vue3中的markRaw

我的理解是: \*\*\*\*  
用于‌标记一个对象，使其永远不会被转换为响应式对象‌。返回该对象本身。  
该对象即使被包裹在 reactive()、ref()、shallowReactive() 等响应式 API 中  
也会保持原始状态，不会触发依赖追踪和视图更新‌。  
官网的解释是：  
将一个对象标记为不可以被转化为代理对象。返回该对象本身。

#### 使用markRaw来解决

```xml
<script setup>
import { onMounted,ref, onBeforeUnmount, markRaw } from 'vue'
// ...省略其他代码,其他代码不变，使用markRaw包裹
chartDom.value = markRaw(echarts.init(chartNode.value));
// ...省略其他代码,其他代码不变
</script>
```

#### 切换页面的时候echarts实例会自动销毁嘛？

不会的。需要手动销毁。  
‌在 Vue 中切换路由(页面)时或刷新页面时，ECharts 实例不会自动销毁。  
需手动调用 dispose() 方法销毁实例‌,否则会导致内存泄漏或二次渲染异常‌。

#### 销毁ECharts实例

```csharp
let chartDom = echarts.init('echarts容器');
// 销毁实例，避免内存泄漏
onBeforeUnmount(()=>{
  chartDom && chartDom.dispose()
})
<script>
```

#### 解决echarts第二次无法渲染的问题

```javascript
// 获取存储echarts容器的节点
let chartNode = document.getElementById('chart')
//移除容器上的 _echarts_instance_ 属性
chartNode.removeAttribute('_echarts_instance_')
```

#### 避免多次重复初始 echarts

通过 echarts.getInstanceByDom() 检查是否已存在实例，避免重复初始化‌

```xml
<template>
  <div class="chart-page">
    <div class="pic" ref="chartNode"></div>
  </div>
</template>
<script setup>
import { ref } from 'vue'
let chartNode  = ref()
// echarts.getInstanceByDom的参数是页面中渲染echarts的DOM节点
chartInstance = echarts.getInstanceByDom(chartNode.value);
// 检查是否已存在实例
if (!chartInstance) {
  console.log('实例不存在')
}else{
  console.log('实例已存在')
}
<script>
```

遇见问题，这是你成长的机会，如果你能够解决，这就是收获。
============================
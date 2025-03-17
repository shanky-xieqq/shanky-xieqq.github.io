---
layout: post
title:  "Element 如何避免 Message 组件重复弹出问题"
categories: Element
tags: Javascript Vue
---

* content
{:toc}

在项目开发中，我们经常使用到 Element 中的 Message 消息提示组件。但是你是否遇到过一个问题：当需要连续弹出多个消息提示时，Element 会弹出多个弹框，使界面看起来十分混乱。本文将带你解决这一问题，让你的界面更加美观整洁！







## 原因分析

可以看到，消息提示是动态向页面根节点插入 `message` 元素来实现弹框效果的，也就是说它们都拥有 `message` 的属性，通过这个信息我们就可以判断页面中 `message` 的个数来决定要不要弹出第二个提示框从而解决这个问题。

## 解决方案

通过上述我们已经知道了问题所在和解决问题的思路，下面我们需要的就是将思路转为实操，如下：

### 实现代码

```javascript
// 重置 message，防止重复点击重复弹出 message 弹框
import { Message } from 'element-ui';

let messageInstance = null;

const mainMessage = options => {
    // 如果弹窗已存在先关闭
    if (messageInstance) messageInstance.close();
    messageInstance = Message(options);
};

const arr = ['success', 'warning', 'info', 'error'];

arr.forEach(type => {
    mainMessage[type] = options => {
        if (typeof options === 'string') {
            options = {
                message: options
            };
        }
        options.type = type;
        return mainMessage(options);
    };
});

const message = mainMessage;

export default message;

// 在 Vue 实例中引入并重写 message 提示框
import message from '@/utils/optimizePop.js'; // 引入
Vue.prototype.$message = message; // 重写 message 提示框，注意: 此行代码一定要放在 vue.use(ElementUI) 后面，否则不生效
```

通过上述代码实现后，可以有效避免 Message 组件重复弹出的问题，使得界面更加整洁美观。

### 总结

通过重写 Message 组件的方法，我们可以控制其弹出行为，避免多次点击导致的重复弹出问题。这样不仅提升了用户体验，也保持了界面的整洁性。
---
title: Vue进阶系列第6篇:Vue实例生命周期详解--组件从创建到销毁的全过程
date: 2025-12-20
tags: Vue, JavaScript进阶
category: Vue
---
<!--  # Vue2生命周期详解：组件从创建到销毁的全过程 -->

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第6篇，今天我们来谈谈剖析一下Vue实例的生命周期。这里我们讲Vue2版本的，Vue3版本生命周期的解析我回放在Vue3对应的专题下面。

Vue2里的每个组件，都有一段从创建到销毁的完整“生命周期”。就像一个东西从生产出来到最终报废，不同阶段有不同的状态，Vue2在这些关键阶段都设置了“钩子函数”，让我们能在合适的时机做该做的事。

## 一、创建阶段：组件的“初始化”
这个阶段是组件从无到有的过程，核心是把数据、方法这些基础东西准备好，还没和页面上的DOM产生关联。

### **1. beforeCreate**
组件实例刚被创建出来，就像一个空架子——`data`里的数据、`methods`里的方法都还没初始化，这时候用`this`也拿不到这些东西。这个钩子基本用不上，只能做些和组件实例没关系的简单操作。

### **2. created**
这时候组件的“核心功能”已经就绪了：响应式数据、方法、侦听器都初始化好了，用`this`能直接访问`this.msg`、`this.getData()`这些。但DOM还没生成，`this.$el`是`undefined`。这个阶段最适合做异步数据请求、处理路由参数、初始化全局变量这些不用操作DOM的事。

## 二、挂载阶段：组件的“落地”
创建阶段准备好了数据和方法，接下来就要把组件渲染成页面上能看到的真实DOM，这个过程就是挂载。

### **1. beforeMount**
Vue已经把模板编译成了渲染函数，也生成了虚拟DOM，但还没把虚拟DOM转换成真实DOM挂载到页面上。这时候`this.$el`只是个占位符，不是真正的DOM元素，还不能操作DOM。

### **2. mounted**
终于，真实DOM已经挂载到页面上了，`this.$el`能直接拿到组件对应的DOM元素，子组件也都挂载完成了（父组件的`mounted`会等所有子组件挂载好才执行）。这是操作DOM的最佳时机，比如初始化ECharts图表、获取元素的尺寸、绑定第三方DOM插件这些需要真实DOM的操作，都适合在这里做。

## 三、更新阶段：组件的“动态刷新”
当组件里的响应式数据发生变化时，比如把`this.msg`从“hello”改成“hi”，就会触发更新阶段，让页面跟着数据变化重新渲染。

### **1. beforeUpdate**
数据已经更新了，但页面上的DOM还没跟着变。这时候能拿到更新前的DOM状态，比如输入框原来的值、元素原来的位置，适合做一些更新前的准备工作，比如保存旧数据用于对比。

### **2. updated**
DOM已经根据新数据重新渲染好了，能看到更新后的页面效果。但要注意，这里不能再修改响应式数据了，不然会一直触发更新，陷入无限循环；也别做太复杂的DOM操作，会影响性能。

## 四、销毁阶段：组件的“清理”
当组件被移除时，比如用`v-if="false"`隐藏、路由跳转离开，就会进入销毁阶段，核心是清理资源，避免浪费内存。

### **1. beforeDestroy**
组件马上要被销毁了，但此时实例还能用，`this`依然能访问数据和方法，DOM也还在页面上。这是清理资源的关键时机，一定要在这里做这些事：清除定时器、解绑自定义事件、取消没完成的网络请求、销毁第三方插件实例（比如ECharts实例），不然这些资源会一直占用内存。

### **2. destroyed**
组件已经完全被销毁了，实例上的所有东西都失效了，DOM元素也被移除了，响应式系统也停止工作。这个钩子基本不用写复杂逻辑，最多做个日志记录。

## 五、特殊的生命周期钩子
除了上面四个核心阶段，Vue2还有几个专门应对特殊场景的钩子：

###  **activated 和 deactivated**
这两个钩子要配合`<keep-alive>`组件使用。`<keep-alive>`会缓存组件，不让它销毁，这时候组件显示（激活）时触发`activated`，隐藏（失活）时触发`deactivated`。

比如列表页跳转到详情页再返回，列表组件会触发`activated`，可以在这里恢复滚动位置；跳转时触发`deactivated`，可以暂停定时器。

###  **errorCaptured**
用来捕获子组件的错误。如果子组件报错，这个钩子会触发，还能返回`false`阻止错误扩散，避免整个应用崩溃，适合做全局的错误提示。


## 写在最后

钩子的执行顺序是固定的，比如先执行`created`再执行`beforeMount`，父组件的`created`比子组件的`created`先执行，但父组件的`mounted`比子组件的`mounted`后执行。

总的来说，Vue2的生命周期就是组件的“成长轨迹”，理解它的核心就是知道“在哪个阶段该做什么事”。

数据请求放在`created`，DOM操作放在`mounted`，资源清理放在`beforeDestroy`，跟着这个逻辑写代码，就能避免很多问题，让组件运行得更稳定、高效。

**【往期精彩】**
- [Vue进阶系列第5篇--Vue实例挂载的过程中发生了什么?](https://mp.weixin.qq.com/s/WzozYCBd3AeSkPonLlsslQ)
- [Vue进阶系列第4篇-- v-show 和 v-if 的实用解析：搞懂用法，选对场景](https://mp.weixin.qq.com/s/bQmFjzSrE_1Z6TzLri-1uQ)

- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)

- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文全说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
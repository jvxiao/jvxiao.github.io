---
title: Vue进阶系列第9篇--你真的懂nextTick吗？我表示很怀疑
date: 2025-12-28 20:58:02
tags: Vue, JavaScript进阶
category: Vue
banner_img: /imgs/baners/vue.jfif
index_img: /imgs/baners/vue.jfif
---

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第9篇，今天我们来讲讲Vue中的`nextTick`的定义和几种使用场景，下一篇单独开一篇讲解它的实现原理。

为什么我会对nextTick“情有独钟“呢，这里还有一个小故事。记得最早接触前端开发解除Vue的时候，由于自己属于看了2周Vue官方文档就开始写业务的半路出家和尚，编程能力属于菜狗都算不上那种。

有一次有个业务逻辑是需要再页面更新之后才进行数据变更的，当时哪知道有nextTick这么牛而x之的函数，直接一个`setTimeout`, 不出意外项目上线后bug时隐时现，最后排查到了是这里有错，当时带我的导师给我指出问题时，咱脸上火辣辣的，哈哈。

言归正传，怎么来讲讲 `nextTick`吧，以后故事慢慢讲。

## nextTick是什么？

### 官方定义(不是我瞎写的哈)

官方给出的说法是：**nextTick 会在下次 DOM 更新循环结束之后，执行我们传入的延迟回调**。简单说就是，当我们修改了 Vue 数据之后，要是想立刻拿到更新后的 DOM 元素，就可以用这个方法。

###  通俗理解

很多刚学 Vue 的朋友可能会有疑问：我明明改了数据，怎么打印 DOM 内容还是旧的？其实核心原因是——Vue 更新 DOM 不是同步的，而是异步执行的。

咱们可以这么想，Vue 内部有个“异步更新队列”，当数据发生变化时，Vue 不会马上就去更新 DOM，而是先把这次数据变化的操作放进这个队列里。等同一轮事件循环里所有的数据变化都处理完了，Vue 才会统一拿出队列里的操作，一次性更新 DOM。这样做的好处是能避免频繁操作 DOM 导致页面性能变差。

### 实际例子（一看就懂）

先写一段简单的 HTML 结构，就一个挂载点，显示 message 的值：

```html

<div id="app"> {{ message }} </div>
```

然后创建 Vue 实例，data 里定义 message 初始值为“原始值”：

```javascript

const vm = new Vue({
  el: '#app',
  data: {
    message: '原始值'
  }
})
```

接下来我们连续修改三次 message 的值，然后打印 DOM 里的内容：

```javascript

this.message = '修改后的值1'
this.message = '修改后的值2'
this.message = '修改后的值3'
// 打印DOM中的内容
console.log(vm.$el.textContent) // 输出：原始值
```

是不是很奇怪？明明改了三次数据，打印出来还是旧值。这就是因为 Vue 没有在每次改数据后都立刻更 DOM，而是把这三次修改都放进了异步队列里，而且队列会自动去重——毕竟三次都是改同一个 message，最后只需要保留最后一次“修改后的值3”就行，没必要执行三次更新。

等这一轮事件循环结束，队列里的操作会被执行，DOM 才会真正更新，这时候再拿 DOM 内容才是最新的。

### 为什么要有 nextTick？

nextTick 本质上是 Vue 的一种性能优化策略，咱们举个极端点的例子就明白了。

比如页面上要显示一个 num 值，然后我们写一个循环，连续修改 10 万次 num：

```javascript

{{num}}
for(let i=0; i<100000; i++){
    num = i
}
```

要是没有 nextTick 这种异步更新机制，num 每变一次，Vue 就会去更新一次视图，10 万次修改就会触发 10 万次 DOM 更新——大家都知道 DOM 操作是很耗性能的，这样页面肯定会卡死。

而有了 nextTick 对应的异步队列机制，不管循环里改了多少次 num，都只会被放进队列一次，最后只更新一次视图，大大提升了页面性能。

## nextTick 的使用场景

核心场景就一个：修改数据后，想立刻获取更新后的 DOM 结构，这时候就用 nextTick 把获取 DOM 的操作包起来。

它有三种常用用法，咱们一个个说，都很简单。

### 1. 全局 Vue.nextTick() 方法

第一个参数是回调函数，在这个函数里能拿到最新的 DOM；第二个参数是执行上下文（可选，一般用不到）。

```javascript

// 修改数据
vm.message = '修改后的值'
// 这时候DOM还没更新，拿到的是旧值
console.log(vm.$el.textContent) // 输出：原始的值
// 用Vue.nextTick包裹获取DOM的操作
Vue.nextTick(function () {
  // 这里DOM已经更新完成，能拿到最新值
  console.log(vm.$el.textContent) // 输出：修改后的值
})
```

### 2. 组件内 this.$nextTick() 方法

在组件内部用的时候，更推荐用 this.$nextTick()，它不用手动绑定 this，回调函数里的 this 会自动指向当前组件实例，更方便。

```javascript

// 组件内修改数据
this.message = '修改后的值'
// 此时DOM未更新
console.log(this.$el.textContent) // 输出：原始的值
// 组件内调用$nextTick
this.$nextTick(function () {
    // this自动指向当前组件
    console.log(this.$el.textContent) // 输出：修改后的值
})
```

### 3. 结合 async/await 使用（最简洁）

$nextTick() 会返回一个 Promise 对象，所以我们可以用 async/await 语法，写法更简洁，看起来也更清爽，现在很多项目里都这么用。

```javascript

// 注意：外层函数要加async
async function changeMsg() {
  this.message = '修改后的值'
  // 此时DOM未更新
  console.log(this.$el.textContent) // 输出：原始的值
  // 等待$nextTick执行完成（也就是DOM更新完成）
  await this.$nextTick()
  // 现在能拿到最新DOM了
  console.log(this.$el.textContent) // 输出：修改后的值
}
```

## 写在最后

脑子机灵的你想必已经掌握了`nextTick`的使用了，但是保不准你同事没有看过，你可以分享给他让他看看这篇文章。

当然了，如果某一天你在代码中将`setTimeout`和`nextTick` 嵌套起来使用，作者也不会怪你，为啥嘞？

因为代码和人，有一个能跑就行了！

**【往期精彩】**
- [Vue进阶系列第8篇--Vue组件间通信方式都有哪些？](https://mp.weixin.qq.com/s/KN8RfpWcAxUNJxMXdnT9dg)

- [Vue进阶系列第7篇--对象添加属性之后页面没有刷新，怎么回事？](https://mp.weixin.qq.com/s/ZpwTlAjMMtzmfFZPh52jTw)

- [深入理解 JavaScript 中的原型与原型链](https://mp.weixin.qq.com/s/2lcovAIzSptuidQNtlVPwg)

- [聊聊ES6里的Promise：简单理解和实际用法
](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
- [说说JavaScript中ES6 module(模块化)的诞生背景、核心语法及应用场景](https://mp.weixin.qq.com/s/pe4fywIh7WdigHuwRiqDlQ)
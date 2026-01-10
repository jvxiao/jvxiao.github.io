---
title: Vue进阶系列第10篇--nextTick 工作原理揭秘：附核心代码实现与解析
date: 2025-12-28 22:30:28
tags: Vue, JavaScript进阶
category: Vue
banner_img: /imgs/baners/vue.jfif
index_img: /imgs/baners/vue.jfif
---

> 不出意外这应该是2025年该公众号更新的最后一篇文章，在这里感谢大家的关注与陪伴。愿大家新的一年代码零 bug，接口全 200，升职加薪，一路开挂。


这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第10篇，在[上一篇文章](https://mp.weixin.qq.com/s/XATTpbsFwd6vuw5EqTUCfQ)中讲了Vue中的`nextTick`的定义和几种使用场景，今天我们结合源码来说说`nextTick`的原理和实现。


`nextTick` 的源码在 Vue 项目的 `/src/core/util/next-tick.js` 里，核心逻辑不复杂，咱们拆解开来讲，不用怕看不懂。

## 核心变量和函数

首先有几个关键东西：

- **callbacks**：就是咱们说的“异步操作队列”，所有通过 nextTick 传入的回调函数，都会被放进这个数组里。

- **pending**：一个标识位，用来保证同一时间只执行一次异步任务，避免重复执行。

- **timerFunc**：核心函数，用来决定用什么方式执行异步任务（会做降级处理）。

- **flushCallbacks**：用来执行 callbacks 队列里所有回调函数的函数。

## nextTick 核心函数逻辑

咱们先看 nextTick 函数的核心代码（保留关键逻辑，去掉冗余注释）：

```javascript

export function nextTick(cb?: Function, ctx?: Object) {
  let _resolve;

  // 把传入的回调函数处理后，放进callbacks队列
  callbacks.push(() => {
    if (cb) {
      // 执行回调，加上try-catch防止报错影响其他逻辑
      try {
        cb.call(ctx);
      } catch (e) {
        handleError(e, ctx, 'nextTick');
      }
    } else if (_resolve) {
      // 没有回调函数的话，就resolve（支持Promise用法）
      _resolve(ctx);
    }
  });

  // 如果pending是false，说明当前没有异步任务在执行，就执行timerFunc
  if (!pending) {
    pending = true;
    timerFunc();
  }

  // 没有传cb的话，返回一个Promise，支持async/await
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve;
    });
  }
}
```

逻辑很简单：

1. 把我们传入的回调函数，包装一下放进 callbacks 队列；

2. 如果当前没有正在执行的异步任务（pending 为 false），就调用 timerFunc 开启异步任务；

3. 如果没传回调函数，就返回一个 Promise，这样就能用 async/await 了。

## timerFunc 降级策略

**timerFunc** 是用来执行异步任务的，它会根据当前浏览器环境，选择最优先、性能最好的方式来执行，做了四层降级处理，顺序是：`Promise.then > MutationObserver > setImmediate > setTimeout`。

为什么要降级？因为不同浏览器对这些 API 的支持不一样，而且优先级也不同——微任务（Promise、MutationObserver）的优先级比宏任务（setImmediate、setTimeout）高，能更快执行，所以优先用微任务。

```javascript

export let isUsingMicroTask = false
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  // 1. 优先用Promise（微任务），兼容性好且性能优
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // 兼容iOS某些版本的bug，加个空的setTimeout
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true // 标记当前用的是微任务
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // 2. 不支持Promise的话，用MutationObserver（微任务）
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true // 监听文本节点内容变化
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter) // 修改文本内容，触发observer
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // 3. 前面都不支持，用setImmediate（宏任务，IE和Node环境支持）
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // 4. 最后降级用setTimeout（宏任务，所有浏览器都支持）
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

## flushCallbacks 执行回调队列

不管是用微任务还是宏任务，最终都会调用 `flushCallbacks` 函数，来执行 callbacks 队列里的所有回调。

```javascript

function flushCallbacks () {
  pending = false // 重置pending，允许下一次异步任务执行
  const copies = callbacks.slice(0) // 复制一份callbacks队列
  callbacks.length = 0 // 清空原队列
  // 循环执行所有回调函数
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
```

这里有个小细节：为什么要复制一份 callbacks 再执行？因为在执行回调的过程中，可能会有新的 `nextTick` 被调用，要是直接操作原队列，会导致回调函数重复执行或者顺序错乱，复制一份就能避免这个问题。

## 简单总结

把 `nextTick `传入的回调函数放进 `callbacks` 队列，然后根据浏览器环境选择最优的异步方式（微任务优先），等异步任务触发后，执行 `flushCallbacks` 函数，依次执行队列里的所有回调，这样就能保证回调函数里拿到的是更新后的 DOM 了。

**【往期精彩】**
- [Vue进阶系列第9篇--你真的懂nextTick吗？我表示很怀疑](https://mp.weixin.qq.com/s/XATTpbsFwd6vuw5EqTUCfQ)
- [Vue进阶系列第8篇--Vue组件间通信方式都有哪些？](https://mp.weixin.qq.com/s/KN8RfpWcAxUNJxMXdnT9dg)

- [深入理解 JavaScript 中的原型与原型链](https://mp.weixin.qq.com/s/2lcovAIzSptuidQNtlVPwg)

- [聊聊ES6里的Promise：简单理解和实际用法
](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
- [说说JavaScript中ES6 module(模块化)的诞生背景、核心语法及应用场景](https://mp.weixin.qq.com/s/pe4fywIh7WdigHuwRiqDlQ)
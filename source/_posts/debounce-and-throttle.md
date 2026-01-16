---
title: 一文读懂：什么时候该用防抖，什么时候该用节流
date: 2026-01-15 
tags: JavaScript进阶
keywords: [JavaScript, Web, 防抖与节流的区别]
---

在 JavaScript 开发中，面对像“滚动页面”“窗口缩放”或者“搜索框输入”这种高频触发的事件，如果代码每触发一次就执行一次任务，电脑很容易“累死”（浏览器卡顿、服务器压力大）。

为了解决这个问题，我们通常会用到防抖（Debounce）和节流（Throttle）。它们就像是给高频事件安上了“减速带”或“过滤器”。

## 防抖（Debouncing）
防抖的核心逻辑是：当事件触发时，不立刻执行，而是等一段时间。如果这段时间内事件又触发了，就重新计时。

就像电脑休眠。你设置了 10 分钟不动就关屏，如果你在第 9 分钟动了下鼠标，电脑会重新开始数 10 分钟。只有当你整整 10 分钟没碰它，它才真的关屏。

**核心理念**： “等最后一个人上车再发车”。


```javascript
function debounce(fn, delay) {
  let timer = null; // 准备一个闹钟变量
  return function(...args) {
    // 如果闹钟还没响又触发了，赶紧把旧闹钟掐了，重新定一个
    if (timer) clearTimeout(timer); 
    timer = setTimeout(() => {
      fn.apply(this, args); 
    }, delay);
  };
}
```


### 应用场景
- **搜索框输入**： 用户不停打字时不需要每敲一个字母就发请求，等他停下（比如 500ms）再发请求最省资源。
- **登录/注册按钮**： 防止用户因为网络卡顿连点好几次，导致后台生成好几个重复账号。

## 节流（Throttling）

节流的核心逻辑是：不管你触发得有多快，我都按照固定的频率执行任务。

就像打游戏放技能（CD）。你疯狂按技能键没用，技能必须等 3 秒冷却时间（CD）好了才能放第二次。在冷却时间内，你按多少次都被忽略。

**核心理念**： “按固定频率放行”。


```javascript
function throttle(fn, delay) {
  let lastTime = 0; // 记录上次干活的时间
  return function(...args) {
    let now = Date.now();
   if (now - lastTime > delay) {
      fn.apply(this, args);
      lastTime = now; // 记得更新最后干活的时间
    }
  };
}
```
> 请谨慎使用此类代码。

### 应用场景
- **滚动监听（scroll）**： 比如判断用户滚到了页面哪个位置。不需要每滚 1 像素就判一次，每 200ms 检查一次就行。
- **游戏射击**： 无论你鼠标点多快，子弹发射的速度是恒定的。

 防抖和节流核心区别
假设你疯狂连续点击按钮 5 秒钟，我们设置的延迟/冷却时间是 1 秒：

| 特性       | 防抖 (Debounce)                | 节流 (Throttle)                |
|------------|--------------------------------|--------------------------------|
| 执行次数   | 只执行 1 次（点完停下后才执行） | 执行 5 次（每秒匀速执行 1 次）  |
| 重置机制   | 每次点击都会重新开始计时       | 不会重置，按固定节奏运行       |
| 关注点     | 关注结果：等最后那一下         | 关注过程：控制执行的频率       |

## 总结：该选哪个？
在 2026 年的开发中，如果你拿不准用哪个，就问自己一个问题：这个活儿是需要“停下来才干”，还是需要“匀速地干”？

- 如果你希望在动作停止后再做响应，选**防抖**。
- 如果你希望在动作进行中持续但有节奏地响应，选**节流**。

**【往期精彩】**
- [原生 DOM 真的慢吗？为什么现代框架都迷恋虚拟 DOM？](https://mp.weixin.qq.com/s/hY97PUXy_WQeagpaYtzk9g)
- [Vue进阶系列第9篇--你真的懂nextTick吗？我表示很怀疑](https://mp.weixin.qq.com/s/XATTpbsFwd6vuw5EqTUCfQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)

- [把大象装进冰箱分几步？手把手教你实现大文件切片上传
](https://mp.weixin.qq.com/s/vDvJG-jOCp9LfuWWgvnqFw)
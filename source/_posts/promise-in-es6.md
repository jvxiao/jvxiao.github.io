---
title: 聊聊ES6里的Promise：简单理解和实际用法
date: 2025-12-06 12:33:46
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---

在JavaScript里，经常会遇到“异步操作”——就是那些不会马上完成的事情，比如从服务器拿数据、读取文件、设置一个定时器等。

以前处理这些事，全靠“回调函数”，但写多了就容易乱。Promise就是ES6里出来的一个工具，专门帮我们把这些异步操作理清楚。


## 为啥需要Promise？

假设我们要做三件事，而且得按顺序来：先烧水，水开了再泡茶叶，茶泡好了再倒出来喝。用以前的回调函数写法，大概是这样：

```javascript
烧开水(function(水) {
  泡茶叶(水, function(茶) {
    倒出来喝(茶, function(结果) {
      console.log(结果);
    });
  });
});
```

这看起来还好，但如果步骤再多一点呢？比如烧水前要先洗水壶，泡茶叶前要选茶叶，代码就会一层套一层，像叠罗汉一样，越叠越高。这就是常说的“回调地狱”——看着头晕，改起来也麻烦，哪一步出错了都得一层层找。

Promise的作用，就是把这种“叠罗汉”的写法，变成“排队走”的样子，步骤清楚，出错了也能一起处理。


##  Promise到底是个啥？

Promise可以理解成一个“容器”，里面装着一个正在进行的异步操作。这个容器有三种“状态”，而且状态一旦确定，就再也改不了：

- **pending（进行中）**：刚开始的状态，比如水刚放到炉子上，还没开。
- **fulfilled（成功了）**：异步操作完成了，比如水烧开了，有结果了（水）。
- **rejected（失败了）**：异步操作出问题了，比如水烧干了，出错了（原因）。

状态只能从“进行中”变成“成功”，或者从“进行中”变成“失败”，变了就定死了。就像水烧开了就不能再变回没开的状态，烧干了也不能再变回去。


## 怎么用Promise？简单三步

### 1. 创建Promise
用`new Promise()`就能创建一个，里面要传一个函数，这个函数有两个参数：`resolve`（成功时调用）和`reject`（失败时调用）。

比如模拟一个“烧水”的异步操作：

```javascript
// 创建一个“烧水”的Promise
const 烧水 = new Promise((resolve, reject) => {
  setTimeout(() => { // 用定时器模拟烧水需要时间
    const 水开了 = Math.random() > 0.3; // 70%概率水烧开
    if (水开了) {
      resolve('水已经烧开了'); // 成功了，把结果传出去
    } else {
      reject(new Error('水没烧开，凉了')); // 失败了，把错误原因传出去
    }
  }, 2000); // 假设烧水需要2秒
});
```


### 2. 处理结果：用then和catch
创建好之后，用`then`处理成功的情况，用`catch`处理失败的情况：

```javascript
烧水
  .then(结果 => { // 水烧开了会走这里
    console.log(结果); // 输出：水已经烧开了
  })
  .catch(错误 => { // 水没烧开会走这里
    console.log(错误.message); // 输出：水没烧开，凉了
  });
```

这样写，成功和失败的处理是分开的，看着清楚。


### 3. 链式调用：解决步骤多的问题
最有用的是，`then`方法会返回一个新的Promise，所以可以“链式”调用，把多个步骤串起来。

比如“烧水→泡茶叶→倒出来喝”这三步，用链式调用写：

```javascript
烧水
  .then(水 => {
    console.log(水);
    return 泡茶叶(水); // 泡茶叶也是一个Promise，返回它
  })
  .then(茶 => {
    console.log('茶叶泡好了');
    return 倒出来喝(茶); // 倒出来喝也是一个Promise
  })
  .then(结果 => {
    console.log(结果); // 输出：可以喝了
  })
  .catch(错误 => { // 任何一步出错，都会走到这里
    console.log('出错了：', 错误.message);
  });
```

这样一来，步骤是按顺序排的，不像以前那样嵌套，一目了然。


## 几个常用的“批量操作”方法

有时候需要同时处理多个异步操作，Promise提供了几个现成的方法：

### Promise.all：等所有操作都完成
比如同时煮米饭和炒菜，等两个都做好了再开饭：

```javascript
// 煮米饭和炒菜都是Promise
const 煮米饭 = new Promise(...);
const 炒菜 = new Promise(...);

// 等两个都完成
Promise.all([煮米饭, 炒菜])
  .then(结果 => {
    const [米饭, 菜] = 结果;
    console.log('米饭和菜都好了，可以开饭了');
  })
  .catch(错误 => {
    console.log('有一个没做好：', 错误.message); // 只要一个失败，就会走这里
  });
```


###  Promise.race：比谁快
比如同时调用两个接口获取数据，谁先返回就用谁的结果：

```javascript
const 接口A = 调用接口('A');
const 接口B = 调用接口('B');

Promise.race([接口A, 接口B])
  .then(数据 => {
    console.log('先返回的数据：', 数据);
  })
  .catch(错误 => {
    console.log('先出错的：', 错误.message);
  });
```

这个也常用在“超时控制”上，比如设置一个3秒的定时器，如果接口3秒内没返回，就提示超时。


## 实际开发中，Promise都用在哪些地方？

**网络请求**：现在前端调接口（比如用fetch、axios），返回的都是Promise。比如获取用户信息：
   ```javascript
   fetch('https://xxx.com/user')
     .then(响应 => 响应.json()) // 解析数据
     .then(用户信息 => {
       console.log('用户信息：', 用户信息);
     })
     .catch(错误 => {
       console.log('请求失败：', 错误);
     });
   ```

**有顺序的异步操作**：比如先登录拿到token，再用token获取用户详情，链式调用很方便。

**批量加载资源**：比如页面加载时，同时加载图片、样式、数据，用`Promise.all`等全部加载完再渲染页面。

**处理超时**：比如接口请求怕太久没反应，用`Promise.race`和定时器结合，超过5秒就提示“加载超时”。


## 总结一下

Promise其实就是个帮我们管理异步操作的工具。它通过“状态”记录操作结果，用“链式调用”代替嵌套回调，让代码更清楚；还能批量处理多个异步操作，解决了以前写异步代码的很多麻烦。

虽然它也有一些小缺点（比如一旦开始就不能停，没处理错误会默默失败），但现在几乎所有前端项目都会用到它，甚至后来的`async/await`也是基于它实现的。搞懂Promise，写异步代码会轻松很多。

**【往期精彩内容】**
- [为什么 ES6 要新增 Set 和 Map？看完这篇就懂了](https://mp.weixin.qq.com/s/UuJeSErftQY7ee1a54Pv4A)
- [JavaScript进化论：ES6如何让函数编写更加简洁、高效？](https://mp.weixin.qq.com/s/haq8TFc_YJlPjeWdyXokQw)
- [JavaScript ES6 中对象的拓展，你了解几个？](https://mp.weixin.qq.com/s/PTI3HGsfpDz4-GvRx0i9ew)
- [Javascript高频面试点--ES6 数组新特性](https://mp.weixin.qq.com/s/qR65z8RbPFxW326UnogLqA)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [程序员如何打破职业瓶颈？先搬开这3块绊脚石。](https://mp.weixin.qq.com/s/DRpZXprnTndjKb1YMqsfjQ)

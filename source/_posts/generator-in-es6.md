---
title: JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文全说透
date: 2025-12-06 12:34:14
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---
今天是[JavaScript进阶系列文章](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4120256678140837890#wechat_redirect)的第8篇文章了，今天我们来讲讲ES6中的生成器(Generator)。

Generator 是 ES6 新加的一种特殊函数，简单说就是**能暂停、能继续、能分批次给结果的函数**。普通函数一旦调用就从头跑到尾，而它像带了暂停键，想停就停，想继续就继续。核心特点就三个：

##  怎么写？So easy!
- 函数名前加 `function*`（星号跟着 function 写最清楚）
- 用 `yield` 当“暂停标记”，碰到它就停，还能把后面的值抛出来
- 调用后不会立马执行，会返回一个“操作手柄”（生成器对象），按手柄上的 `next()` 按钮才会启动/继续

```javascript
// 举个例子：分三次给数字
function* numberGenerator() {
  yield 1; // 第一次暂停，返回1
  yield 2; // 第二次暂停，返回2
  return 3; // 结束了，返回3
}

// 拿个操作手柄
const gen = numberGenerator();

console.log(gen.next()); // { value: 1, done: false } → 按第一次，跑到第一个暂停点
console.log(gen.next()); // { value: 2, done: false } → 按第二次，跑到第二个暂停点
console.log(gen.next()); // { value: 3, done: true } → 按第三次，跑完了
```

 打个比方，普通函数像自助售货机一次性把所有商品全吐出来（全程不停）

而生成器(Generator) 函数就是投一次币出一件货（出完就等下一次投币，也就是按 next()）

---

## 工作原理：记着暂停位置的“执行机器”
Generator 能暂停续跑，全靠一套“记位置的规则”（迭代器协议），本质就是个**带记忆的执行机器**，步骤特简单：

 **拿手柄不启动**：调用  `numberGenerator()` 时，函数里的代码一动不动，只给你个“手柄”（生成器对象）。
 
 **按 next() 才走**：每按一次手柄上的 `next()`，就从上次暂停的地方接着跑，直到碰到下一个 `yield` 或 `return`。
 
**跑一步给个反馈**：`next()` 会返回一个对象，里面有俩信息：
 - `value`：这次跑出来的结果（yield 后面的值或 return 的值）
  - `done`：布尔值，`false` 表示没跑完，`true` 表示彻底结束了
    
 **结束就停摆**：一旦 `done` 变成 `true`，再按 `next()` 也没用，只会返回 `{ value: undefined, done: true }`。

---

## 好用的特性：不止能暂停

### 能跟函数“对话”
按 `next()` 时还能传参数，这个参数会变成**上一个 yield 的返回值**，相当于函数内外能互相传消息：

```javascript
// 模拟聊天机器人
function* chatGenerator() {
  // 暂停并问问题，等外部传名字进来
  const name = yield "请问你叫什么名字？";
  // 收到名字后，返回问候语
  yield `你好，${name}！`;
}

const chat = chatGenerator();
console.log(chat.next().value); // 请问你叫什么名字？（第一次按，只提问）
console.log(chat.next("张三").value); // 你好，张三！（传名字过去，得到回应）
```

###  能抓错不崩溃
可以主动给函数里抛错误，配合 `try/catch` 就能接住，不会让整个程序挂掉：

```javascript
function* safeGenerator() {
  try {
    yield "正在正常跑";
    yield "没出问题";
  } catch (err) {
    // 接住错误并返回提示
    yield `出错了：${err.message}`;
  }
}

const gen = safeGenerator();
console.log(gen.next().value); // 正在正常跑
// 主动抛个“网络断了”的错
console.log(gen.throw(new Error("网络中断")).value); // 出错了：网络中断
```

### 能叫“帮手”干活（yield*）
用 `yield*` 能让别的“可迭代对象”（比如另一个 Generator、数组、字符串）帮着出结果，不用自己写重复代码：

```javascript
// 帮手1：负责出1和2
function* gen1() { yield 1; yield 2; }
// 主函数：让帮手先干活，再自己出3
function* gen2() {
  yield* gen1(); // 叫帮手gen1先上
  yield 3;
}

[...gen2()] // 结果是 [1,2,3]，帮手的结果直接接过来

// 也能让数组当帮手
function* stringGenerator() {
  yield* "ABC"; // 相当于依次出'A'、'B'、'C'
}
```

###  能强制“叫停”
用 `return()` 能直接终止函数，不管后面还有没有 `yield`，之后再按 `next()` 也没用：

```javascript
const gen = numberGenerator();
console.log(gen.next().value); // 1（跑第一步）
console.log(gen.return("不跑了").value); // 不跑了（强制叫停）
console.log(gen.next().done); // true（彻底结束）
```

---

## 什么场景用它？

### 异步代码变“直”（替代回调地狱）
以前写异步请求（比如调接口）要嵌套一堆回调，Generator 能让代码像同步一样写，不过得配合 Promise 和“自动按按钮的工具”（比如 co 库）：

```javascript
// 模拟调接口拿数据（1秒后返回结果）
const fetchData = (url) => new Promise(resolve => {
  setTimeout(() => resolve(`从${url}拿到的数据`), 1000);
});

// 用Generator写异步流程，跟写同步代码一样
function* asyncTask() {
  try {
    const user = yield fetchData("/api/user"); // 等用户数据
    const posts = yield fetchData(`/api/posts?uid=${user.id}`); // 再拿文章
    return posts;
  } catch (err) {
    console.error("请求失败：", err);
  }
}

// 用co库自动按next（不用自己一次次点）
const co = require("co");
co(asyncTask).then(result => console.log("最终结果：", result));
```
> 原理特简单：co库会盯着每次 yield 出来的 Promise，等它有结果了，自动按 next 把结果传进去，一直到函数跑完。

###  自定义遍历逻辑（想要啥数据自己造）
Generator 天生就能当“遍历器”，比自己写复杂的遍历代码简单多了，比如：
- 生成无限序列（比如斐波那契数列）
- 遍历树、图这种复杂结构
- 分批加载数据（用多少取多少）

```javascript
// 生成斐波那契数列（要多少取多少，不占内存）
function* fibonacci() {
  let a = 0, b = 1;
  while (true) { // 无限循环也不怕，因为会暂停
    [a, b] = [b, a + b];
    yield b;
  }
}

const fib = fibonacci();
// 只取前10个，不会一下生成无数个占内存
for (let i = 0; i < 10; i++) {
  console.log(fib.next().value); // 1,2,3,5,8...
}
```

### 分批处理大数据（不卡崩页面）
处理海量数据或分页加载时，用 Generator 一次拿一批，避免一次性加载太多数据把页面卡崩：

```javascript
// 分页加载数据的工具
function* paginatedLoader(pageSize = 10) {
  let page = 1;
  while (true) {
    // 每次暂停，等着拿这一页的数据
    const data = yield fetch(`/api/data?page=${page}&size=${pageSize}`);
    if (data.length === 0) break; // 没数据了就停
    page++;
  }
}

// 用的时候按需加载
const loader = paginatedLoader();
loader.next().then(data => 处理第1页数据(data));
loader.next().then(data => 处理第2页数据(data));
```

###  管理状态流转（比如订单状态）
Generator 的“暂停-续跑”特别适合记录状态变化，每个 `yield` 就是一个状态，比如订单从“待支付”到“已完成”：

```javascript
// 订单状态机
function* orderStateMachine() {
  yield "待支付"; // 下单后状态
  yield "已支付"; // 付款后状态
  yield "已发货"; // 商家发货后
  yield "已完成"; // 收货后
}

const order = orderStateMachine();
console.log(order.next().value); // 待支付（刚下单）
console.log(order.next().value); // 已支付（付完钱）
console.log(order.next().value); // 已发货（商家发了货）
```

---

## Generator 和 async/await 啥关系？为啥还要学？
async/await 其实是 Generator 的“简化版”（底层就是 Generator + Promise），但它俩不是谁替代谁的关系，各有各的用处：

| 特点         | Generator                | async/await          |
|--------------|--------------------------|----------------------|
| 写法         | function* + yield        | async + await        |
| 要不要帮手   | 要（比如co库或自己按next）| 不用，浏览器直接支持  |
| 主要能干啥   | 啥都能搞（暂停、遍历、状态）| 专门优化异步代码      |
| 出错怎么抓   | 要调throw()方法          | 直接用try/catch就行  |

为啥现在还要学 Generator？因为它有不可替代的地方：
- **懂底层才会用上层**：明白 Generator 怎么跑，才真的懂 async/await 不是“黑魔法”；

- **复杂遍历更简单**：生成无限序列、遍历树结构，比 async/await 代码短多了；
 - **能手动控制节奏**：比如做游戏帧同步、逐帧渲染动画，需要精确控制执行时机，Generator 更灵活。

---

## 总结：Generator 核心就是“能控制”
Generator 最牛的地方，就是给 JavaScript 加了“可控制的执行流程”——以前函数要么不跑，要么一口气跑完，现在能想停就停、想续就续、想传消息就传消息。

虽然 async/await 抢了它“异步编程”的风头，但在造遍历器、管状态、处理大数据这些场景里，它还是无可替代。学懂它不光能解决实际问题，更能明白 JavaScript 函数是怎么“干活”的，算得上是从“会写代码”到“懂代码”的关键一步。

**【往期精彩】**

- [聊聊ES6里的Promise：简单理解和实际用法](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
- [为什么 ES6 要新增 Set 和 Map？看完这篇就懂了](https://mp.weixin.qq.com/s/UuJeSErftQY7ee1a54Pv4A)
- [JavaScript进化论：ES6如何让函数编写更加简洁、高效？](https://mp.weixin.qq.com/s/haq8TFc_YJlPjeWdyXokQw)
- [JavaScript ES6中Object的拓展，你了解几个？](https://mp.weixin.qq.com/s/PTI3HGsfpDz4-GvRx0i9ew)
- [Javascript高频面试点--ES6 数组新特性](https://mp.weixin.qq.com/s/qR65z8RbPFxW326UnogLqA)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)

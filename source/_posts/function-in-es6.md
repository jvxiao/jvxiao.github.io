---
title: JavaScript进化论：ES6如何让函数编写更加简洁、高效？
date: 2025-12-06 12:32:43
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---

以前写 JavaScript 函数，真的有不少麻烦事：参数默认值得手动判断、处理多个参数要靠 `arguments` 瞎折腾、`this` 指向还老让人晕头转向。ES6 算是把这些痛点都补上了，给函数加了好多实用扩展，写代码又快又不容易踩坑。下面用大白话+简单例子，把这些功能讲明白，新手也能一看就会～

## 参数默认值：不用再写 “a || 1” 啦
以前想给函数参数设默认值，得在函数里写一堆判断，又麻烦又容易出问题。ES6 直接允许在参数列表里写默认值，省事多了。
- 核心特点：要是没传参数，或者传了个 `undefined`，就自动用默认值。
- 对比示例：
```javascript
// ES5 写法：得手动判断参数在不在
function fn1(a, b) {
  a = a || 1; // 这里有个坑啊——要是你传个 a=0，它也会被改成 1
  b = b === undefined ? 2 : b;
  return a + b;
}

// ES6 写法：直接在参数里设默认值，多直观
function fn2(a = 1, b = 2) {
  return a + b;
}

console.log(fn2()); // 输出 3（没传参数，直接用默认值 1 和 2）
console.log(fn2(3)); // 输出 5（只传了 a=3，b 还是用默认值 2）
console.log(fn2(0, 0)); // 输出 0（传 0 也不会被替换，完美避开 ES5 的坑）
```

---

## 剩余参数：用 ... 收齐所有剩余参数
以前处理不确定个数的参数，得用 `arguments` 这个类数组，还得手动转成真正的数组才能用 `forEach`、`map` 这些方法，太折腾了。ES6 的剩余参数直接用 ... 就能收齐所有参数，天生就是数组，不用额外处理。
- 核心特点：只能放在参数列表最后，不管剩多少个参数，都能一次性接住。
- 对比示例：
```javascript
// ES5 写法：靠 arguments 处理多个参数，麻烦到家
function sum1() {
  // 先把类数组 arguments 转成真正的数组，才能用 reduce
  const nums = Array.prototype.slice.call(arguments);
  return nums.reduce((total, num) => total + num, 0);
}

// ES6 写法：用 ...nums 直接收齐所有参数，省事
function sum2(...nums) {
  return nums.reduce((total, num) => total + num, 0);
}

console.log(sum1(1, 2, 3)); // 输出 6
console.log(sum2(1, 2, 3, 4)); // 输出 10（不管传多少个参数，都能搞定）
```

---

## 扩展运算符：把数组 “拆” 成参数用
扩展运算符和剩余参数是反过来的：剩余参数是“收”参数，扩展运算符是“拆”数组，把数组里的元素一个个拆出来当函数参数，再也不用写 `apply `了。
- 核心特点：在数组前面加个 ...，就能直接展开成逗号分隔的参数。
- 实用示例：
```javascript
// 场景 1：求数组最大值（ES5 得用 apply 绕一圈）
const arr = [10, 20, 5, 15];
console.log(Math.max.apply(null, arr)); // ES5 写法：输出 20，看着就麻烦
console.log(Math.max(...arr)); // ES6 写法：输出 20，直接拆数组，多简洁

// 场景 2：合并数组
const arr1 = [1, 2];
const arr2 = [3, 4];
console.log([...arr1, ...arr2]); // 输出 [1,2,3,4]，直接拼就行

// 场景 3：给函数传数组参数
function fn(a, b, c) {
  return a + b + c;
}
console.log(fn(...[1, 2, 3])); // 输出 6（数组直接拆成 1、2、3 三个参数）
```

---

## 箭头函数：写代码快到飞起，还解决 this 坑
箭头函数绝对是 ES6 里最常用的扩展，不仅写法极简，还把普通函数最让人头疼的 this 指向问题给解决了，再也不用记各种 this 指向规则了。
- 核心特点：
  - 语法超简洁：`(参数) => 返回值`，不用写 function、return，甚至连大括号都能省（单条语句时）。
  - 没有自己的 this：this 直接继承外层作用域的，不管怎么调用都不变。
  - 不能当构造函数：没法用 `new` 关键字创建实例。
  - 没有 arguments：想处理多个参数，直接用剩余参数就行。
- 对比示例：
```javascript
// 场景 1：简单函数简写，一行搞定
const add1 = function(a, b) { // ES5 普通函数，写一堆字
  return a + b;
};
const add2 = (a, b) => a + b; // ES6 箭头函数，精简到极致

// 场景 2：解决 this 指向的坑（以前写代码常踩的雷）
const obj = {
  name: "张三",
  age: 20,
  // ES5 普通函数：this 会跟着调用方式变，太坑了
  sayHi1: function() {
    setTimeout(function() {
      console.log(`我是 ${this.name}`); // 输出 "我是 undefined"（this 指向 window 了）
    }, 100);
  },
  // ES6 箭头函数：this 直接继承外层的 obj，稳得很
  sayHi2: function() {
    setTimeout(() => {
      console.log(`我是 ${this.name}`); // 输出 "我是 张三"（this 一直指向 obj）
    }, 100);
  }
};
obj.sayHi1();
obj.sayHi2();
```

---

## 参数解构：直接“拆”对象/数组当参数
有时候函数参数是个对象或数组，以前得先在函数里手动提取属性，写起来很啰嗦。ES6 能直接在参数列表里解构，把需要的属性或元素直接拿出来用。
- 实用示例：
```javascript
// 场景 1：解构对象参数，还能设默认值
function printUser({ name, age = 18 }) { // 直接提取 name 和 age，age 没传就用 18
  console.log(`姓名：${name}，年龄：${age}`);
}
printUser({ name: "李四" }); // 输出 "姓名：李四，年龄：18"（age 用默认值）
printUser({ name: "王五", age: 25 }); // 输出 "姓名：王五，年龄：25"

// 场景 2：解构数组参数
function printPoint([x, y]) { // 直接提取数组的前两个元素当 x 和 y
  console.log(`坐标：(${x}, ${y})`);
}
printPoint([10, 20]); // 输出 "坐标：(10, 20)"
```

---

## name 属性优化：函数名字更靠谱了
以前函数的 `name` 属性不太准，调试的时候容易 confusion。ES6 把这个问题修好了，函数名字能准确显示，调试起来更方便。
- 对比示例：
```javascript
// ES5：匿名函数赋值给变量，name 是空字符串，调试找不到
const fn1 = function() {};
console.log(fn1.name); // 输出 ""

// ES6：匿名函数赋值给变量，name 就是变量名，一目了然
const fn2 = () => {};
console.log(fn2.name); // 输出 "fn2"

// 具名函数赋值，name 还是函数本身的名字
const fn3 = function myFn() {};
console.log(fn3.name); // 输出 "myFn"
```

---

## 总结：这些扩展该怎么用？
- **箭头函数优先用**：写简单逻辑、怕 this 出错的时候，直接用箭头函数，省心又快捷。
- **参数默认值+解构**：传对象或数组当参数时，用解构+默认值，不用手动提属性，代码更干净。
- **剩余参数替代 arguments**：处理多个参数时，剩余参数天生是数组，比 arguments 好用多了。
- **扩展运算符多活用**：合并数组、给函数传数组参数时，用 ... 直接拆，比 apply 简洁太多。

这些扩展本质上都是为了让代码更短、更好懂、少踩坑，现在前端开发里，这些写法早就成了标配，学会了能省不少事～

**【往期精彩内容】**
- [JavaScript ES6 中对象的拓展，你了解几个？](https://mp.weixin.qq.com/s/PTI3HGsfpDz4-GvRx0i9ew)
- [Javascript高频面试点--ES6 数组新特性](https://mp.weixin.qq.com/s/qR65z8RbPFxW326UnogLqA)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [程序员如何打破职业瓶颈？先搬开这3块绊脚石。](https://mp.weixin.qq.com/s/DRpZXprnTndjKb1YMqsfjQ)
- [深入理解 JavaScript 中的原型与原型链](https://mp.weixin.qq.com/s/2lcovAIzSptuidQNtlVPwg)

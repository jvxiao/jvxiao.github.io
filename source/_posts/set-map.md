---
title: 为什么 ES6 要新增 Set 和 Map？看完这篇就懂了
date: 2025-12-06 12:33:14
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---

在ES6（ECMAScript 2015）的众多特性中，Set和Map两种新数据结构的引入，为JavaScript开发者提供了更灵活、高效的数据存储与处理方式。它们弥补了传统数组与对象的局限性，在处理唯一值、复杂键值对场景时展现出独特优势。

今天我将从概念、特性、用法到实际应用，全面解析一下`Set`与`Map`。


## Set：无重复值的集合

Set是一种**无序的集合**，其核心特性是**成员唯一**——不会存储重复的值。这一特性使其在需要去重、判断存在性等场景中大放异彩。


###  基本特性与创建方式

- **创建Set**：通过`new Set()`构造函数创建，可接收一个**可迭代对象**（如数组、字符串）作为初始数据。
- **核心属性**：`size`用于获取集合中成员的数量（类似数组的`length`）。

```javascript
// 空Set
const emptySet = new Set();

// 从数组创建（自动去重）
const numSet = new Set([1, 2, 2, 3]); 
console.log(numSet.size); // 输出：3（重复的2被移除）

// 从字符串创建（字符串是可迭代对象）
const strSet = new Set('hello'); 
console.log(strSet); // 输出：Set(4) { 'h', 'e', 'l', 'o' }（重复的'l'被移除）
```


###  核心方法

Set提供了一套简洁的方法用于操作成员：

- `add(value)`：添加成员，返回Set本身（可链式调用）。
- `delete(value)`：删除指定成员，返回布尔值表示是否删除成功。
- `has(value)`：判断是否包含指定成员，返回布尔值。
- `clear()`：清空所有成员，无返回值。

```javascript
const fruits = new Set();

// 添加成员
fruits.add('apple').add('banana').add('apple'); 
console.log(fruits); // 输出：Set(2) { 'apple', 'banana' }

// 判断存在性
console.log(fruits.has('banana')); // 输出：true

// 删除成员
console.log(fruits.delete('apple')); // 输出：true（删除成功）
console.log(fruits); // 输出：Set(1) { 'banana' }

// 清空
fruits.clear();
console.log(fruits.size); // 输出：0
```


### 遍历与值的比较规则

Set是**可迭代对象**，支持多种遍历方式：

- `forEach(callback)`：按插入顺序遍历，回调参数为`(value, key, set)`（注意：Set中`value`与`key`相同）。
- `keys()`/`values()`：返回键/值的迭代器（因键值相同，两者行为一致）。
- `entries()`：返回`[value, value]`形式的迭代器。

```javascript
const colorSet = new Set(['red', 'green', 'blue']);

// forEach遍历
colorSet.forEach((value, key) => {
  console.log(`${key}: ${value}`); // 输出：red: red、green: green、blue: blue
});

// for...of遍历
for (const color of colorSet) {
  console.log(color); // 输出：red、green、blue
}
```

需要注意Set的**值比较规则**：与`===`类似，但有一个例外——`NaN`被视为与自身相等（而`===`中`NaN !== NaN`）。

```javascript
const specialSet = new Set([NaN, NaN, 1, '1']);
console.log(specialSet.size); // 输出：3（两个NaN被视为重复，1与'1'是不同值）
```


### 典型应用场景

- **数组去重**：利用Set的唯一性，配合扩展运算符快速去重。

```javascript
  const arr = [1, 2, 2, 3, 3, 3];
  const uniqueArr = [...new Set(arr)]; // 输出：[1, 2, 3]
 ```

- **集合运算**：实现交集、并集、差集等数学集合操作。
 ```javascript
  const setA = new Set([1, 2, 3]);
  const setB = new Set([2, 3, 4]);

  // 并集：A∪B
  const union = new Set([...setA, ...setB]); // {1,2,3,4}

  // 交集：A∩B
  const intersection = new Set([...setA].filter(x => setB.has(x))); // {2,3}

  // 差集：A-B
  const difference = new Set([...setA].filter(x => !setB.has(x))); // {1}
  ```

- **存储唯一标识**：如用户ID、DOM节点等需要唯一存储的数据。


## Map：键值对的高级实现

Map是一种**有序的键值对集合**，与对象（Object）类似，但解决了对象作为键值对容器的诸多限制，尤其在键的类型灵活性上表现突出。


###  基本特性与创建方式

- **创建Map**：通过`new Map()`构造函数创建，可接收一个**二维数组**（每个子数组为`[key, value]`）作为初始数据。
- **核心属性**：`size`用于获取键值对的数量。

```javascript
// 空Map
const emptyMap = new Map();

// 从二维数组创建
const userMap = new Map([
  ['name', 'Alice'],
  [18, 'age'], // 键可以是数字
  [() => {}, 'method'] // 键可以是函数
]);
console.log(userMap.size); // 输出：3
```


### 核心方法

Map的方法围绕键值对的操作设计：

- `set(key, value)`：添加键值对，返回Map本身（可链式调用）。
- `get(key)`：获取指定键对应的值，若不存在返回`undefined`。
- `delete(key)`：删除指定键值对，返回布尔值表示是否删除成功。
- `has(key)`：判断是否包含指定键，返回布尔值。
- `clear()`：清空所有键值对，无返回值。

```javascript
const bookMap = new Map();

// 添加键值对（键可以是任意类型）
bookMap
  .set('title', 'ES6 Guide')
  .set(2023, 'publishYear')
  .set({ author: 'Bob' }, 'info');

// 获取值
console.log(bookMap.get('title')); // 输出：'ES6 Guide'
console.log(bookMap.get({ author: 'Bob' })); // 输出：undefined（对象引用不同）

// 判断键是否存在
console.log(bookMap.has(2023)); // 输出：true

// 删除键值对
bookMap.delete('title');
console.log(bookMap.size); // 输出：2
```


### 遍历与键的比较规则

Map同样是**可迭代对象**，且**按插入顺序遍历**，遍历方式包括：

- `forEach(callback)`：回调参数为`(value, key, map)`。
- `keys()`：返回键的迭代器。
- `values()`：返回值的迭代器。
- `entries()`：返回`[key, value]`形式的迭代器（默认迭代器）。

```javascript
const langMap = new Map([['js', 'JavaScript'], ['py', 'Python']]);

// forEach遍历
langMap.forEach((value, key) => {
  console.log(`${key}: ${value}`); // 输出：js: JavaScript、py: Python
});

// 默认迭代器（entries()）
for (const [key, value] of langMap) {
  console.log(`${key} → ${value}`); // 输出：js → JavaScript、py → Python
}
```

Map的**键比较规则**与Set一致：`NaN`视为相等，对象键通过引用比较（不同引用的对象视为不同键）。


### 典型应用场景

- **复杂键值存储**：当键需要是对象、函数等非字符串类型时，Map是唯一选择。

 ```javascript
  // 用DOM元素作为键存储数据（避免污染DOM属性）
  const domMap = new Map();
  const button = document.querySelector('button');
  domMap.set(button, { clickCount: 0 });

  // 点击时更新数据
  button.addEventListener('click', () => {
    const data = domMap.get(button);
    data.clickCount++;
    console.log(`点击次数：${data.clickCount}`);
  });
  ```

- **需要保持键顺序的场景**：如配置项、有序缓存等（依赖插入顺序时更可靠）。

- **频繁增删键值对**：相比对象，Map在动态修改时性能更稳定。


## 总结：何时选择Set与Map？

Set和Map作为ES6新增的数据结构，为JavaScript开发提供了更精准的工具：

- **选择Set**：当需要存储**唯一值**，且重点关注“值是否存在”而非“键值关联”时（如去重、集合运算）。
- **选择Map**：当需要**键值对映射**，且键的类型复杂（非字符串）、需要保持顺序或频繁修改时（如复杂状态管理、缓存）。

**【往期精彩内容】**
- [JavaScript进化论：ES6如何让函数编写更加简洁、高效？](https://mp.weixin.qq.com/s/haq8TFc_YJlPjeWdyXokQw)
- [JavaScript ES6 中对象的拓展，你了解几个？](https://mp.weixin.qq.com/s/PTI3HGsfpDz4-GvRx0i9ew)
- [Javascript高频面试点--ES6 数组新特性](https://mp.weixin.qq.com/s/qR65z8RbPFxW326UnogLqA)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [程序员如何打破职业瓶颈？先搬开这3块绊脚石。](https://mp.weixin.qq.com/s/DRpZXprnTndjKb1YMqsfjQ)



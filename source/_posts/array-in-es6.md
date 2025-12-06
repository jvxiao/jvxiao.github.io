---
title: Javascript高频面试点--ES6 数组(Array)新特性
date: 2025-12-06 12:29:11
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---

数组是 JavaScript 处理数据的核心结构，ES6（ECMAScript 2015）对数组的扩展，解决了传统操作的繁琐问题，让数据处理更高效。下面我们来结合实例，梳理 ES6 数组的核心扩展特性与用法。

## 一、语法层面的突破：扩展运算符


扩展运算符（`...`）是 ES6 中最具代表性的语法糖之一，它将数组或类数组对象展开为独立的元素，彻底改变了数组拷贝、合并、参数传递等常见操作的实现方式。

### 1. 数组拷贝

传统拷贝数组需借助`slice()`或`concat()`，ES6 中使用扩展运算符更为简洁直观，且支持深拷贝一维数组：



```javascript
// 浅拷贝一维数组

const arr1 = [1, 2, 3];

const arr2 = [...arr1]; // [1, 2, 3]

arr2[0] = 100;

console.log(arr1); // [1, 2, 3]（原数组不受影响）

// 对比ES5写法

const arr3 = arr1.slice();

const arr4 = [].concat(arr1);
```

### 2. 数组合并

无需再依赖`concat()`的链式调用，扩展运算符可直接合并多个数组，且支持在任意位置插入元素：



```javascript
const arrA = [1, 2];

const arrB = [3, 4];

const arrC = [0, ...arrA, 2.5, ...arrB, 5]; // [0, 1, 2, 2.5, 3, 4, 5]
```

### 3. 函数参数传递

解决了`apply()`传递数组参数的繁琐问题，直接将数组元素作为独立参数传入函数：



```javascript
const numbers = [10, 20, 30];

// ES6写法

Math.max(...numbers); // 30

// ES5写法

Math.max.apply(null, numbers); // 30
```

### 4. 解构赋值结合

扩展运算符可与解构赋值配合，便捷提取数组部分元素：



```javascript
const [first, second, ...rest] = [1, 2, 3, 4, 5];

console.log(first); // 1

console.log(second); // 2

console.log(rest); // [3, 4, 5]（剩余元素组成新数组）
```

## 二、数组构造函数的增强：Array.from () 与 Array.of ()

ES6 为`Array`构造函数新增了两个静态方法，分别解决了 “类数组对象转数组” 和 “创建固定长度数组” 的经典问题。

#### 1. Array.from ()：类数组对象与可迭代对象转数组

类数组对象（如 DOM 集合、`arguments`）和可迭代对象（如 Set、Map），在 ES6 前需通过`Array.prototype.slice.call()`转换为数组，`Array.from()`提供了统一、直观的解决方案，还支持映射函数参数：



```javascript
// 转换类数组对象（DOM集合）

const lis = document.querySelectorAll('li');

const liArray = Array.from(lis); // 转为数组，可使用数组方法

// 转换arguments对象

function sum() {
	 return Array.from(arguments).reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6

// 带映射函数（类似map()）

const str = 'hello';

Array.from(str, char => char.toUpperCase()); // ['H', 'E', 'L', 'L', 'O']

// 转换Set对象

const set = new Set([1, 2, 2, 3]);

Array.from(set); // [1, 2, 3]（自动去重）
```

#### 2. Array.of ()：统一数组创建行为

传统`Array`构造函数存在行为不一致问题：当传入单个数字时，创建的是指定长度的空数组，而非包含该数字的数组。`Array.of()`修复了这一缺陷，无论传入多少参数，均创建包含这些参数的数组：



```javascript
// ES6 Array.of()

Array.of(5); // [5]

Array.of(1, 2, 3); // [1, 2, 3]

Array.of(); // []（无参数返回空数组）

// ES5 Array构造函数（对比）

Array(5); // [empty × 5]（长度为5的空数组）

Array(1, 2, 3); // [1, 2, 3]（多参数正常）
```

## 三、数组实例方法的升级：查找、遍历与修改

ES6 新增了多个数组实例方法，覆盖了查找、遍历、填充、扁平化等高频场景，让数据处理逻辑更简洁。

### 1. 查找类方法：find () 与 findIndex ()

传统查找数组元素依赖`indexOf()`，但它仅支持查找原始值，且无法处理 “满足特定条件的元素”。`find()`和`findIndex()`弥补了这一不足，支持传入回调函数，返回第一个满足条件的元素或其索引：



```javascript
const users = [
  { id: 1, name: '张三', age: 20 },
  { id: 2, name: '李四', age: 25 },
  { id: 3, name: '王五', age: 30 }
];

// find()：返回第一个满足条件的元素

const targetUser = users.find(user => user.age > 22);

console.log(targetUser); // { id: 2, name: '李四', age: 25 }

// findIndex()：返回第一个满足条件的元素索引

const targetIndex = users.findIndex(user => user.name === '王五');

console.log(targetIndex); // 2

// 支持查找NaN（indexOf()无法做到）

const numbers = [1, NaN, 3];

numbers.find(n => Object.is(n, NaN)); // NaN

numbers.findIndex(n => Object.is(n, NaN)); // 1
```

### 2. 包含判断：includes ()

`indexOf()`判断元素是否存在时，需通过返回值是否为`-1`判断，语义不够清晰。`includes()`直接返回布尔值，且支持判断`NaN`，使用更直观：



```javascript
const fruits = ['苹果', '香蕉', '橙子'];

fruits.includes('香蕉'); // true

fruits.includes('葡萄'); // false

// 支持NaN判断

[1, 2, NaN].includes(NaN); // true

[1, 2, NaN].indexOf(NaN); // -1（对比）
```

### 3. 迭代器方法：entries ()、keys ()、values ()

ES6 为数组新增了三个迭代器方法，支持通过`for...of`循环遍历数组的键、值或键值对，替代了传统的`for`循环或`forEach()`：



```javascript
const arr = ['a', 'b', 'c'];

// keys()：遍历索引

for (const key of arr.keys()) {

&#x20; console.log(key); // 0, 1, 2

}

// values()：遍历值（ES2017正式标准化）

for (const value of arr.values()) {

 console.log(value); // 'a', 'b', 'c'

}

// entries()：遍历键值对（返回数组\[key, value]）

for (const [key, value] of arr.entries()) {

console.log(`${key}: ${value}`); // 0: a, 1: b, 2: c

}
```

#### 4. 填充与复制：fill () 与 copyWithin ()

`fill()`用于将数组指定范围的元素替换为固定值，`copyWithin()`则将数组内部的元素复制到另一位置，均支持修改原数组（也可通过浅拷贝避免）：



```javascript

// fill(value, start?, end?)：填充数组

const arr1 = [1, 2, 3, 4, 5];

arr1.fill(0); // [0, 0, 0, 0, 0]（填充整个数组）

arr1.fill(9, 1, 3); // [0, 9, 9, 0, 0]（从索引1到3前填充9）

// copyWithin(target, start?, end?)：复制内部元素

const arr2 = [1, 2, 3, 4, 5];

arr2.copyWithin(0, 3); // [4, 5, 3, 4, 5]（将索引3及以后的元素复制到索引0开始的位置）

arr2.copyWithin(1, 0, 2); // [4, 4, 5, 4, 5]（将索引0-2前的元素复制到索引1开始的位置）
```

### 5. 扁平化处理：flat  与 flatMap 

多维数组扁平化是常见需求，ES6 前需递归实现，`flat()`和`flatMap()`提供了原生支持：



```javascript
// flat(depth?)：扁平化数组，depth默认1（可传Infinity扁平化所有层级）

const nestedArr = [1, [2, [3, [4]]]];

nestedArr.flat(); // [1, 2, [3, [4]]]（扁平化1层）

nestedArr.flat(2); // [1, 2, 3, [4]]（扁平化2层）

nestedArr.flat(Infinity); // [1, 2, 3, 4]（扁平化所有层级）

// flatMap()：先执行map()，再执行flat(1)（效率更高）

const arr = [1, 2, 3];

// 等价于 arr.map(x => [x * 2]).flat()

arr.flatMap(x => [x * 2]); // \[2, 4, 6]

// 处理嵌套数组场景

const lists = [
 ['a', 'b'],
 ['c', 'd'],
 ['e']
];

lists.flatMap(list => list.map(item => item.toUpperCase())); // \['A', 'B', 'C', 'D', 'E']
```

## 四、空位处理的规范化

ES5 中数组空位（如`[1, , 3]`）的行为不一致（`forEach()`、`map()`跳过空位，`join()`视为空字符串）。ES6 明确规定：数组空位应被视为`undefined`，新增方法（如`find()`、`includes()`）均按`undefined`处理，避免了歧义：



```javascript
const arr = [1, , 3];

// ES6方法：将空位视为undefined

arr.includes(undefined); // true

arr.find(x => x === undefined); // undefined（找到空位）

arr.flat(); // [1, undefined, 3]（扁平化不忽略空位）

// ES5方法（对比）

arr.map(x => x || 0); // [1, 0, 3]（map()跳过空位，返回原位置）

arr.join('-'); // "1--3"（join()将空位视为空字符串）
```

ES6中数组的拓展用简洁语法和实用方法，解决了传统操作的痛点，减少代码量、提升可读性。掌握这些特性，能帮助我们更高效地处理项目中的数据。

最关键的是，面试频率很高啊，哈哈!
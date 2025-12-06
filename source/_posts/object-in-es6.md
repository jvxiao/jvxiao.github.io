---
title: JavaScript ES6 中对象的拓展，你了解几个？
date: 2025-12-06 12:31:10
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---


## 引言
ES6（ECMAScript 2015）作为 JavaScript 语言发展的里程碑版本，对原生`Object`进行了全方位的升级优化。这些扩展不仅简化了对象的创建与操作语法，还补充了此前缺失的核心功能（如原型操作、属性遍历、深度复制支持等），彻底改变了 JavaScript 开发者的编码习惯。本文将系统拆解 ES6 对象的六大核心扩展特性，结合实战代码示例，让每个特性的应用场景一目了然。


## 一、对象字面量的语法简化
ES6 大幅简化了对象字面量的声明语法，减少冗余代码，提升可读性。

### 1.1 属性简写
当对象属性名与变量名一致时，可省略冒号（`:`）和属性值，直接简写为属性名：
```javascript
// ES5写法
const name = "张三";
const age = 28;
const user = {
  name: name,
  age: age
};

// ES6简写
const user = { name, age }; // 等价于ES5写法
```

### 1.2 方法简写
对象方法声明可省略`function`关键字和冒号，直接使用`方法名()`语法：
```javascript
// ES5写法
const obj = {
  sayHi: function() {
    console.log("Hello");
  }
};

// ES6简写
const obj = {
  sayHi() {
    console.log("Hello"); // 无需function关键字
  },
  async fetchData() { // 支持async/await语法
    const res = await fetch("/api");
  }
};
```

### 1.3 计算属性名
通过方括号（`[]`）包裹表达式，动态生成对象属性名，解决了 ES5 中无法直接声明动态属性的问题：
```javascript
// ES5动态属性名（需额外赋值）
const key = "gender";
const obj = {};
obj[key] = "male"; // 只能在声明后添加

// ES6计算属性名（声明时直接定义）
const obj = {
  [key]: "male",
  [`user_${10}`]: "admin" // 支持表达式拼接
};
```


## 二、对象属性的遍历机制
ES6 新增了 3 个遍历对象属性的方法，配合`for...of`循环，实现更灵活的属性遍历。

### 2.1 Object.keys ()：获取属性名数组
返回对象**自身可枚举属性**的名称数组（不含原型链属性）：
```javascript
const obj = { a: 1, b: 2, c: 3 };
console.log(Object.keys(obj)); // ["a", "b", "c"]
```

### 2.2 Object.values ()：获取属性值数组
返回对象**自身可枚举属性**的对应值数组：
```javascript
console.log(Object.values(obj)); // [1, 2, 3]
```

### 2.3 Object.entries ()：获取键值对数组
返回对象**自身可枚举属性**的`[key, value]`二维数组，完美适配`for...of`循环：
```javascript
for (const [key, value] of Object.entries(obj)) {
  console.log(`${key}: ${value}`); // 输出 "a: 1" "b: 2" "c: 3"
}

// 快速转换为Map
const map = new Map(Object.entries(obj));
```

> 注意：以上三个方法均遵循 “自身可枚举” 规则，即忽略`enumerable: false`的属性和原型链上的属性。


## 三、对象的合并与复制：Object.assign ()
ES6 新增`Object.assign()`方法，用于将多个**源对象**的属性复制到**目标对象**，实现对象合并或浅拷贝。

### 3.1 基本用法
```javascript
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { a: 3, c: 4 };

// 合并：source1、source2的属性依次复制到target
Object.assign(target, source1, source2);
console.log(target); // { a: 3, b: 2, c: 4 }（同名属性被后续源对象覆盖）
```

### 3.2 核心特性
1. **浅拷贝**：仅复制属性值，若属性值为引用类型（如对象、数组），则复制引用地址：
```javascript
const source = { info: { age: 20 } };
const target = Object.assign({}, source);
target.info.age = 30;
console.log(source.info.age); // 30（源对象被修改，因为复制的是引用）
```

2. **仅复制可枚举属性**：忽略`enumerable: false`的属性：
```javascript
const source = Object.defineProperty({}, "hidden", {
  value: 10,
  enumerable: false // 不可枚举属性
});
const target = Object.assign({}, source);
console.log(target.hidden); // undefined（未复制）
```

3. **支持多源对象**：可传入多个源对象，属性按顺序覆盖（后传入的源对象优先级更高）。


## 四、原型操作的规范化
ES6 提供了直接操作对象原型的 API，替代了 ES5 中不规范的`__proto__`属性。

### 4.1 Object.setPrototypeOf ()：设置原型
用于指定对象的原型（即`[[Prototype]]`内部属性）：
```javascript
const parent = { greet() { console.log("Hello"); } };
const child = {};

// 设置child的原型为parent
Object.setPrototypeOf(child, parent);
child.greet(); // "Hello"（继承自parent）
```

### 4.2 Object.getPrototypeOf ()：获取原型
返回对象的原型对象，替代`obj.__proto__`：
```javascript
console.log(Object.getPrototypeOf(child) === parent); // true
```

### 4.3 super 关键字：访问原型属性
在对象方法中，`super`指向当前对象的原型，可直接调用原型的属性或方法：
```javascript
const parent = { name: "父对象" };
const child = {
  getParentName() {
    return super.name; // 等价于 Object.getPrototypeOf(this).name
  }
};
Object.setPrototypeOf(child, parent);
console.log(child.getParentName()); // "父对象"
```


## 五、属性描述符的完整获取：Object.getOwnPropertyDescriptors ()
ES6 新增该方法，用于获取对象**所有自身属性**的完整描述符（包括`value`、`writable`、`enumerable`、`configurable`、`get`、`set`等），解决了`Object.getOwnPropertyDescriptor()`只能获取单个属性描述符的局限。

### 5.1 基本用法
```javascript
const obj = {
  name: "张三",
  get age() { return 28; }, // getter属性
  set age(val) { console.log(val); } // setter属性
};

// 获取所有自身属性的描述符
const descriptors = Object.getOwnPropertyDescriptors(obj);
console.log(descriptors.name.value); // "张三"
console.log(descriptors.age.get); // 对应的getter函数
```

### 5.2 核心应用：深拷贝含 getter/setter 的对象
`Object.assign()`无法复制`getter`/`setter`属性（仅复制其返回值），而结合`Object.getOwnPropertyDescriptors()`和`Object.create()`可实现完整复制：
```javascript
const clone = Object.create(
  Object.getPrototypeOf(obj), // 继承原型
  Object.getOwnPropertyDescriptors(obj) // 复制所有属性描述符
);
console.log(clone.age); // 28（getter正常工作）
```


## 六、Symbol 作为对象属性名
ES6 新增`Symbol`原始类型，其生成的实例具有唯一性，可作为对象属性名，避免属性名冲突。

### 6.1 基本用法
```javascript
// 创建Symbol实例（参数仅为描述，不影响唯一性）
const id = Symbol("id");
const obj = {
  [id]: 1001, // 作为属性名需用方括号
  name: "李四"
};

// 访问Symbol属性
console.log(obj[id]); // 1001（必须用Symbol实例访问）

// 特性：无法通过for...in/Object.keys()遍历
for (const key in obj) {
  console.log(key); // 仅输出 "name"，忽略Symbol属性
}

// 获取所有Symbol属性名
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]
```

### 6.2 应用场景：定义 “私有” 属性
虽然 ES6 没有真正的私有属性，但`Symbol`属性无法被常规遍历方法获取，可模拟私有属性：
```javascript
// 模块内部定义Symbol，外部无法访问
const _private = Symbol("private");
export const obj = {
  [_private]: "敏感数据",
  publicMethod() {
    return this[_private]; // 内部可访问
  }
};

// 外部无法通过常规方式获取_private属性
```


## 总结
ES6 对对象的扩展围绕 “简化语法、完善功能、提升安全性” 三大核心目标，其特性已成为现代 JavaScript 开发的基础：
- 语法简化（属性 / 方法简写、计算属性名）减少冗余代码；
- 遍历与合并（Object.keys/values/entries、Object.assign）提升操作效率；
- 原型与描述符操作（setPrototypeOf、getOwnPropertyDescriptors）规范底层逻辑；
- Symbol 属性名解决命名冲突问题。

这些特性不仅让对象操作更直观、灵活，也为后续框架（如 Vue、React）的底层实现提供了语法支撑。掌握这些扩展，能显著提升代码质量与开发效率，是前端开发者的必备技能。


**【往期精彩内容】**
- [JavaScript ES6 中对象的拓展，你了解几个？](https://mp.weixin.qq.com/s/PTI3HGsfpDz4-GvRx0i9ew)
- [Javascript高频面试点--ES6 数组新特性](https://mp.weixin.qq.com/s/qR65z8RbPFxW326UnogLqA)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [程序员如何打破职业瓶颈？先搬开这3块绊脚石。](https://mp.weixin.qq.com/s/DRpZXprnTndjKb1YMqsfjQ)
- [深入理解 JavaScript 中的原型与原型链](https://mp.weixin.qq.com/s/2lcovAIzSptuidQNtlVPwg)

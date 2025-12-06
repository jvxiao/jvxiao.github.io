---
title: JavaScript ES6中的装饰器(Decorator)是什么？有哪些应用场景？万字长文，谨慎阅读！
date: 2025-12-06 12:35:25
tags: JavaScript进阶, ES6进阶
category: ES6进阶
---
<!--# ES6 Module：重塑JavaScript模块化生态的基石 -->
这是[JavaScript进阶系列文章](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4120256678140837890#wechat_redirect)的第11篇文章, 今天我们来讲讲`装饰器(Decorator)`，内容有点长，代码有些多，坐稳发车了。

在前端开发中，我们常面临这样的困境：如何在不修改原有代码的前提下，为类、方法或属性添加日志、权限校验、缓存等通用功能？

如果直接在业务逻辑中嵌入这些辅助代码，会导致核心逻辑与通用逻辑耦合，代码可读性和可维护性大幅下降。而 ES6 提案中`Decorator(装饰器)`，正是为解决这一问题而生的优雅方案。

本文将从 Decorator 的**核心本质**出发，详解其**语法特性**、**使用场景**，并结合实战案例说明其在项目中的价值，帮助开发者真正掌握这一“非侵入式扩展”的利器。

## Decorator 是什么？ 
### 核心定义
Decorator 是基于 **“包装模式”（Wrapper Pattern）** 的语法糖，其核心目标是：**在不侵入原有代码结构的前提下，为类、类成员（方法/属性）动态扩展功能**。

它的本质是一个`特殊函数`：接收被装饰的“目标对象”作为参数，通过修改或包装目标对象，返回增强后的新对象。简单来说，Decorator 就像给礼物包装——礼物本身（核心业务逻辑）不变，但包装（通用辅助逻辑）能让它具备额外的“属性”（如美观、防护）。

### 核心价值
Decorator 最核心的价值是 **“分离关注点”**：将日志、权限、缓存等通用逻辑与核心业务逻辑解耦，实现通用逻辑的复用，让代码更简洁、可维护。

举个直观的对比：
- 无 Decorator：在每个需要日志的方法中重复写 `console.log`，业务代码与日志逻辑混杂；
- 有 Decorator：写一个通用日志装饰器，通过 `@log` 语法一键应用到任意方法，业务代码保持纯净。

## Decorator 的核心用法
Decorator 的语法简洁，核心分为“类装饰器”和“类成员装饰器”两类，各自承担不同的扩展职责。

### 1. 类装饰器：增强整个类
类装饰器直接作用于类本身，用于为类添加静态属性、静态方法，或修改类的原型（prototype）。

#### 语法与参数
- 仅接收 1 个参数 `target`：被装饰的类本身；
- 可选返回值：若返回新类，则新类会替代原类；若不返回，则默认使用修改后的原类。

#### 实战示例：给类添加通用能力
```javascript
// 类装饰器：为类添加静态配置和实例初始化方法
function withConfig(defaultConfig) {
  // 支持传入参数的装饰器（高阶函数形式）
  return function (target) {
    // 添加静态属性：默认配置
    target.defaultConfig = defaultConfig;
    
    // 添加静态方法：合并配置
    target.mergeConfig = function (userConfig) {
      return { ...defaultConfig, ...userConfig };
    };
    
    // 增强实例构造逻辑
    const originalConstructor = target;
    // 重写类构造函数
    const newConstructor = function (...args) {
      originalConstructor.apply(this, args);
      // 实例初始化：合并配置到实例
      this.config = target.mergeConfig(args[0] || {});
    };
    
    // 继承原类的原型（保证实例方法不丢失）
    newConstructor.prototype = Object.create(originalConstructor.prototype);
    newConstructor.prototype.constructor = newConstructor;
    
    return newConstructor;
  };
}

// 应用装饰器：传入默认配置
@withConfig({ theme: "light", size: "medium" })
class Button {
  constructor(userConfig) {
    // 原构造函数逻辑
    this.name = "按钮";
  }
  
  getStyle() {
    return `主题：${this.config.theme}，尺寸：${this.config.size}`;
  }
}

// 测试效果
console.log(Button.defaultConfig); // { theme: "light", size: "medium" }
const darkButton = new Button({ theme: "dark" });
console.log(darkButton.config); // { theme: "dark", size: "medium" }
console.log(darkButton.getStyle()); // 主题：dark，尺寸：medium
```

### 2. 类成员装饰器：增强方法/属性
类成员装饰器作用于类的方法、属性或访问器（getter/setter），是开发中最常用的场景。

#### 语法与参数
接收 3 个核心参数：
- `target`：类的原型（针对实例方法）或类本身（针对静态方法）；
- `name`：被装饰的成员名称（方法名/属性名）；
- `descriptor`：成员的属性描述符（`Object.getOwnPropertyDescriptor(target, name)` 的返回值），包含 `value`（方法本身）、`writable`（是否可修改）、`enumerable`（是否可枚举）等属性。

#### 实战示例：包装方法实现日志记录
```javascript
// 方法装饰器：记录方法调用参数、返回值和执行时间
function logPerformance(target, name, descriptor) {
  // 保存原方法（避免覆盖后丢失）
  const originalMethod = descriptor.value;
  
  // 重写方法逻辑
  descriptor.value = async function (...args) {
    const start = Date.now();
    console.log(`[${name}] 开始执行，参数：`, args);
    
    try {
      // 执行原方法（绑定 this 上下文，确保实例方法的 this 正确）
      const result = await originalMethod.apply(this, args);
      console.log(`[${name}] 执行成功，返回值：`, result);
      return result;
    } catch (error) {
      console.error(`[${name}] 执行失败，错误：`, error);
      throw error; // 抛出错误，不影响上层处理
    } finally {
      const duration = Date.now() - start;
      console.log(`[${name}] 执行耗时：${duration}ms`);
    }
  };
  
  // 返回修改后的描述符（替代原成员）
  return descriptor;
}

class DataService {
  // 应用方法装饰器
  @logPerformance
  async fetchData(url) {
    // 模拟接口请求
    const response = await fetch(url);
    return response.json();
  }
}

// 测试
const service = new DataService();
service.fetchData("https://api.example.com/data");
// 输出：
// [fetchData] 开始执行，参数： ["https://api.example.com/data"]
// [fetchData] 执行成功，返回值： { ...接口返回数据... }
// [fetchData] 执行耗时：320ms
```

## 三、Decorator 的典型使用场景：让通用逻辑复用更优雅
Decorator 的核心优势在于通用逻辑的复用，以下是前端开发中最实用的 6 个场景，覆盖日志、权限、缓存等高频需求。

### 1. 日志与性能监控
为核心业务方法（如接口请求、数据处理）添加日志，自动记录调用参数、返回值、执行时间，无需侵入业务代码。适合用于线上问题排查和性能优化。

```javascript
// 通用日志装饰器：支持开关和自定义格式
function log(options = { enable: true, format: "default" }) {
  return function (target, name, descriptor) {
    if (!options.enable) return descriptor; // 关闭日志时直接返回原描述符
    
    const original = descriptor.value;
    descriptor.value = function (...args) {
      const time = new Date().toLocaleString();
      // 自定义日志格式
      if (options.format === "detail") {
        console.log(`[${time}] [${target.constructor.name}.${name}] 调用参数：`, args);
      } else {
        console.log(`[${time}] [${name}] 调用参数：`, args);
      }
      return original.apply(this, args);
    };
    return descriptor;
  };
}

class OrderService {
  @log({ enable: true, format: "detail" })
  createOrder(params) {
    console.log("创建订单：", params);
    return { orderId: "123456" };
  }
}
```

### 2. 权限校验与访问控制
限制敏感方法的调用权限（如管理员操作、用户登录态校验），将权限逻辑与业务逻辑分离，避免在每个方法中重复写校验代码。

```javascript
// 权限装饰器：支持多角色校验
function requireRoles(...allowedRoles) {
  return function (target, name, descriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args) {
      // 假设从全局状态或实例中获取当前用户角色
      const userRole = this.getCurrentUserRole();
      
      if (!allowedRoles.includes(userRole)) {
        throw new Error(`权限不足！当前角色：${userRole}，允许角色：${allowedRoles.join(",")}`);
      }
      
      return original.apply(this, args);
    };
    return descriptor;
  };
}

class SystemManager {
  getCurrentUserRole() {
    return "admin"; // 实际场景中从登录态获取
  }
  
  @requireRoles("admin", "super_admin") // 仅管理员和超级管理员可调用
  deleteUser(userId) {
    console.log(`删除用户：${userId}`);
  }
  
  @requireRoles("user", "admin") // 普通用户和管理员均可调用
  queryUser(userId) {
    console.log(`查询用户：${userId}`);
  }
}
```

### 3. 缓存优化（记忆化）
对计算密集型方法（如递归、大数据处理）或高频调用的接口添加缓存，避免重复计算或请求，提升性能。

```javascript
// 缓存装饰器：支持过期时间
function cache(options = { ttl: Infinity }) {
  const cacheMap = new Map(); // 缓存容器
  
  return function (target, name, descriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args) {
      // 用参数生成唯一缓存键（复杂参数可使用 hash 函数优化）
      const cacheKey = `${name}_${JSON.stringify(args)}`;
      
      // 检查缓存是否存在且未过期
      if (cacheMap.has(cacheKey)) {
        const { data, timestamp } = cacheMap.get(cacheKey);
        if (Date.now() - timestamp < options.ttl) {
          console.log(`[${name}] 命中缓存`);
          return data;
        }
        console.log(`[${name}] 缓存过期，重新计算`);
        cacheMap.delete(cacheKey);
      }
      
      // 执行原方法并缓存结果
      const result = original.apply(this, args);
      cacheMap.set(cacheKey, {
        data: result,
        timestamp: Date.now()
      });
      
      return result;
    };
    return descriptor;
  };
}

// 斐波那契数列计算（计算密集型）
class FibCalculator {
  @cache({ ttl: 5000 }) // 缓存 5 秒
  fib(n) {
    console.log(`[fib] 计算 fib(${n})`);
    if (n <= 1) return n;
    return this.fib(n - 1) + this.fib(n - 2);
  }
}

const calculator = new FibCalculator();
calculator.fib(10); // 首次计算，打印多次 "计算 fib(n)"
calculator.fib(10); // 命中缓存，直接返回结果
setTimeout(() => calculator.fib(10), 6000); // 缓存过期，重新计算
```

### 4. 防抖与节流
装饰事件处理方法（如输入框搜索、按钮点击、窗口resize），避免频繁触发，提升用户体验。

```javascript
// 防抖装饰器：延迟执行，多次触发仅最后一次生效
function debounce(delay = 300, immediate = false) {
  let timer = null;
  
  return function (target, name, descriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args) {
      clearTimeout(timer);
      
      // 立即执行模式：首次触发时立即执行
      if (immediate && !timer) {
        original.apply(this, args);
      }
      
      timer = setTimeout(() => {
        original.apply(this, args);
        timer = null;
      }, delay);
    };
    return descriptor;
  };
}

// 节流装饰器：固定时间内仅执行一次
function throttle(interval = 500) {
  let lastTime = 0;
  
  return function (target, name, descriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args) {
      const now = Date.now();
      if (now - lastTime >= interval) {
        lastTime = now;
        original.apply(this, args);
      }
    };
    return descriptor;
  };
}

class SearchComponent {
  @debounce(500) // 输入停止 500ms 后执行搜索
  handleSearch(keyword) {
    console.log("搜索关键词：", keyword);
  }
  
  @throttle(1000) // 1 秒内仅执行一次
  handleScroll() {
    console.log("窗口滚动");
  }
}
```

### 5. 错误捕获与统一处理
捕获方法执行中的错误，避免程序崩溃，并统一进行错误上报、用户提示等处理，减少重复的 try/catch 代码。

```javascript
// 错误处理装饰器：支持自定义错误回调
function errorHandler(cb = (err) => {
  console.error("发生错误：", err);
  alert("操作失败，请重试");
}) {
  return function (target, name, descriptor) {
    const original = descriptor.value;
    descriptor.value = async function (...args) {
      try {
        return await original.apply(this, args);
      } catch (err) {
        // 统一处理错误：回调函数支持自定义逻辑
        cb(err, { method: name, args });
      }
    };
    return descriptor;
  };
}

class APIService {
  @errorHandler(async (err, info) => {
    // 自定义错误处理：上报日志到后端
    await fetch("/api/error-report", {
      method: "POST",
      body: JSON.stringify({
        message: err.message,
        stack: err.stack,
        method: info.method,
        args: info.args,
        time: new Date().toLocaleString()
      })
    });
    alert("网络异常，请稍后再试");
  })
  async getUserInfo(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) throw new Error(`HTTP 错误：${response.status}`);
    return response.json();
  }
}
```

### 6. 框架集成（React/Vue）
在类组件中复用框架能力，简化代码。例如 React 中连接 Redux、路由守卫，Vue 中装饰组件选项等。

#### React 中连接 Redux（react-redux）
```javascript
import React from "react";
import { connect } from "react-redux";
import { increment, decrement } from "./store/actions";

// 用装饰器简化 connect 逻辑
@connect(
  // mapStateToProps：将 Redux 状态映射到组件 props
  state => ({ count: state.counter.count }),
  // mapDispatchToProps：将 action 映射到组件 props
  { increment, decrement }
)
class Counter extends React.Component {
  render() {
    const { count, increment, decrement } = this.props;
    return (
      <div>
        <p>计数：{count}</p>
        <button onClick={increment}>+1</button>
        <button onClick={decrement}>-1</button>
      </div>
    );
  }
}

export default Counter;
```

#### Vue 中装饰组件（vue-class-component）
```javascript
import Vue from "vue";
import Component from "vue-class-component";
import { Prop } from "vue-property-decorator";

// 用装饰器定义 Vue 组件
@Component({
  template: `
    <div>
      <p>{{ message }}</p>
      <button @click="sayHello">点击</button>
    </div>
  `
})
export default class HelloWorld extends Vue {
  // 用 Prop 装饰器定义属性
  @Prop({ type: String, default: "Hello Vue" })
  message!: string;
  
  sayHello() {
    console.log(this.message);
  }
}
```

## 使用 Decorator 的注意事项与避坑指南

- **提案兼容性问题**
2025 年 Decorator 仍处于 TC39 Stage 3 阶段，不同工具链的配置需适配最新语。


- **装饰器的执行顺序**
多个装饰器叠加时，执行顺序为 `“从右到左、从上到下`（外层装饰器包裹内层装饰器）
```javascript
@decoratorA
@decoratorB
@decoratorC
class MyClass {}

// 等价于：decoratorA(decoratorB(decoratorC(MyClass)))
```

- **this 上下文绑定**
方法装饰器中，原方法的 `this` 默认指向 `undefined`（严格模式下），需通过 `original.apply(this,args)` 或者`original.call(this, ...args)` 绑定当前实例的 `this`，否则会导致 `this` 丢失。

-  **不可装饰普通函数**
当前 Stage 3 提案仅支持装饰“类”和“类成员”，不支持直接装饰普通函数。若需扩展普通函数，可使用高阶函数替代：
```javascript
// 普通函数的“装饰”（高阶函数形式）
function logFunc(fn) {
  return function (...args) {
    console.log("调用参数：", args);
    return fn.apply(this, args);
  };
}

// 应用高阶函数
function add(a, b) {
  return a + b;
}

const addWithLog = logFunc(add);
addWithLog(1, 2); // 输出：调用参数：[1,2]，返回 3
```

## 写在最后
Decorator 并非 ES6 正式标准，但它已成为前端开发中“非侵入式扩展”的经典方案，其核心价值在于分离通用逻辑与业务逻辑，实现代码复用，同时使得代码更加优雅。

如果你在开发中面临“通用逻辑重复编写”“业务代码被辅助逻辑侵入”等问题，不妨尝试使用 Decorator——它会让你的代码更简洁、更优雅、更具扩展性。

**【往期精彩】**
- [说说JavaScript中ES6 module(模块化)的诞生背景、核心语法及应用场景](https://mp.weixin.qq.com/s/pe4fywIh7WdigHuwRiqDlQ)
- [一文说透ES6 Proxy: 从本质到应用场景](https://mp.weixin.qq.com/s/lYqBgS0GJgQMX5PAW_snnQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文全说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [聊聊ES6里的Promise：简单理解和实际用法](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [为什么 ES6 要新增 Set 和 Map？看完这篇就懂了](https://mp.weixin.qq.com/s/UuJeSErftQY7ee1a54Pv4A)

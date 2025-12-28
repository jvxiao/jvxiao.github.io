---
title: Vue进阶系列第8篇--Vue组件间通信方式都有哪些？
date: 2025-12-26 20:58:02
tags: Vue, JavaScript进阶
category: Vue
---
> 今天内容略粗糙（码字有点累），而且知识点有点多，不想看全文的可以直接看总结，浓缩的就是精华。

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第8篇，今天我们来讲讲Vue面试必问的一个问题--Vue组件间通信方式都有哪些? 

## 一、组件间通信的概念

开始之前，我们先把“组件间通信”这个词拆开来理解：

• **组件**
• **通信**

我们都知道，组件是Vue最强大的功能之一，Vue中每一个`.vue`文件都可以视作一个独立组件。而通信，指的是发送者通过某种媒介、以某种格式将信息传递给接收者，从而达成特定目的——广义来讲，任何信息的交互都属于通信。

所以，组件间通信就是指各个.vue组件通过某种方式传递信息，进而实现特定业务需求。举个例子，我们使用UI框架中的Table组件时，常会往组件里传入数据，这一操作的本质，就是组件间在进行通信。

在Vue中，每个组件都有自己独立的作用域，组件间的数据默认是无法共享的，但实际开发中，我们经常需要让不同组件共享数据、协同工作——这就是组件通信要解决的核心问题。只有让组件间能顺畅通讯，才能搭建出一个有机、完整的前端系统。

## 二、组件间通信的分类

组件间通信主要可分为以下几类：

• **父子组件之间的通信**
• **兄弟组件之间的通信**
• **祖孙与后代组件之间的通信**
• **非关系组件之间的通信**




## 三、组件间通信的方案

下面整理了Vue中8种常规的组件间通信方案：

1. 通过 props 传递
2. 通过 `$emit` 触发自定义事件
3. 使用 ref
4. EventBus
5. `$parent` 或 `$root`
6. `$attrs` 与 `$listeners`
7. Provide 与 Inject
8. Vuex

### 1. props 传递数据

**适用场景**：父组件向子组件传递数据

**核心逻辑**：子组件通过设置props属性，定义需要接收的参数；父组件在使用子组件的标签中，通过字面量形式传递值。



```javascript
// Children.vue
props:{  
    // 字符串形式  
    name: String, // 接收的参数类型为字符串  
    // 对象形式（更规范，支持校验）  
    age:{    
        type: Number, // 接收的类型为数值  
        default: 18,  // 默认值为18  
        required: true // age属性必须传递  
    }  
}
```



```html
<!-- Father.vue 组件 -->
<Children name="jack" age="18" />
```

### 2. $emit 触发自定义事件

**适用场景**：子组件向父组件传递数据

**核心逻辑**：子组件通过 $emit 触发自定义事件，第二个参数为需要传递的数值；父组件通过绑定该事件的监听器，获取子组件传递的参数。



```javascript
//  Children.vue
this.$emit('add', good); // 触发add事件，传递good数据
```



```html
<!-- Father.vue -->
<Children @add="cartAdd($event)" /> // 监听add事件，$event接收传递的数据
```

### 3. ref

**适用场景**：父组件获取子组件实例（含数据、方法）

**核心逻辑**：父组件在使用子组件时设置 ref 属性，之后通过 this.$refs.xxx 直接获取子组件实例，进而拿到子组件的数据或调用其方法。



```html
<!-- 父组件 -->
<Children ref="foo" />
```

```javascript

this.$refs.foo; // 获取子组件实例，可通过该实例访问子组件的数据和方法
```

### 4. EventBus

**适用场景**：兄弟组件传值、无直接关系组件间传值

**核心逻辑**：创建一个中央事件总线（EventBus），需要传值的组件通过 $emit 触发事件并传递数据，需要接收数据的组件通过 $on 监听对应事件，从而获取数据。



```javascript
// Bus.js
// 创建一个中央事件总线类  
class Bus {  
  constructor() {  
    this.callbacks = {}; // 存放事件回调函数  
  }  
  $on(name, fn) {  
    // 注册事件：将回调函数存入对应事件名的数组中  
    this.callbacks[name] = this.callbacks[name] || [];  
    this.callbacks[name].push(fn);  
  }  
  $emit(name, args) {  
    // 触发事件：执行对应事件名的所有回调函数  
    if (this.callbacks[name]) {  
      this.callbacks[name].forEach((cb) => cb(args));  
    }  
  }  
}  

// main.js 中挂载到Vue原型上  
Vue.prototype.$bus = new Bus(); // 将$bus挂载到Vue实例原型  
// 简化方案：Vue本身已实现Bus功能，可直接这样写  
// Vue.prototype.$bus = new Vue();
```



```javascript
// Children1.vue（发送数据的组件）
this.$bus.$emit('foo', data); // 触发foo事件，传递data数据
```



```javascript
//Children2.vue（接收数据的组件）
// 监听foo事件，通过回调函数获取数据  
this.$bus.$on('foo', this.handleReceiveData);
```

### 5. $parent 或 $root

**适用场景**：兄弟组件传值（通过共同祖辈）

**核心逻辑**：通过共同的祖辈组件（`$parent`指向父组件，`$root` 指向根组件）搭建通信桥梁，一个组件触发事件，另一个组件监听祖辈的该事件。

兄弟组件A（接收数据）

```javascript

this.$parent.$on('add', this.add); // 监听父组件的add事件
```

兄弟组件B（发送数据）

```javascript

this.$parent.$emit('add', data); // 向父组件触发add事件，传递数据
```

### 6. $attrs 与 $listeners

**适用场景**：祖先组件向子孙组件隔代传值（无需层层props传递）

**核心逻辑**：`$attrs`包含父级作用域中未被props识别的特性绑定（class和style除外），`$listeners` 包含父级作用域中的事件监听器。可通过` v-bind="$attrs"、v-on="$listeners"` 批量向下传递给子组件。

```html

// 子组件（未在props中声明foo）  
<p>{{ $attrs.foo }}</p>  

// 父组件  
<HelloWorld foo="foo" />  

// 隔代传值：父组件（communication/index.vue）  
<Child2 msg="lalala" @some-event="onSomeEvent"></Child2>  

// 子组件（Child2）：批量传递给孙子组件  
<Grandson v-bind="$attrs" v-on="$listeners"></Grandson>  

// 孙子组件（Grandson）：使用数据并触发事件  
<div @click="$emit('some-event', 'msg from grandson')">  
  {{ msg }}  
</div>
```

### 7. Provide 与 Inject

**适用场景**：祖先组件向任意后代组件传值（跨层级，不限深度）

**核心逻辑**：祖先组件通过 provide 选项返回需要传递的值，后代组件通过 inject 选项接收该值，实现跨层级数据共享。

祖先组件

```javascript

provide() {  
    return {  
        foo: 'foo' // 要传递给后代的参数  
    }  
}
```

后代组件

```javascript

inject: ['foo'] // 接收祖先组件传递的foo值
```

### 8. Vuex

**适用场景**：复杂关系组件间的数据传递（如多组件共享同一状态、跨层级/无关系组件传值）

**核心逻辑**：Vuex 相当于一个全局共享数据容器，专门用于管理Vue应用的共享状态，所有组件都可通过Vuex获取或修改共享数据。

核心模块说明：

• state：存放共享变量的地方（相当于全局数据池）
• getter：派生状态（相当于Vuex中的计算属性），用于获取加工后的共享数据
• mutations：存放修改state的同步方法（唯一能直接修改state的地方）
• actions：存放修改state的异步方法（通过调用mutations间接修改state，不直接操作state）

## 小结

• **父子组件传值**：优先选择 props + $emit，也可使用 ref
• **兄弟组件传值**：优先选择 EventBus，其次可通过 $parent 实现
• **祖孙/跨层级传值**：优先选择 `$attrs` + `$listeners` 或 Provide + Inject
• **复杂场景**（多组件共享状态）：优先使用 Vuex

**【往期精彩】**
- [Vue进阶系列第7篇--对象添加属性之后页面没有刷新，怎么回事？](https://mp.weixin.qq.com/s/ZpwTlAjMMtzmfFZPh52jTw)

- [Vue进阶系列第6篇: Vue实例生命周期详解--组件从创建到销毁的全过程](https://mp.weixin.qq.com/s/48mDp6TnaU4XPNHIi75ebA)


- [Vue进阶系列第5篇--Vue实例挂载的过程中发生了什么?](https://mp.weixin.qq.com/s/WzozYCBd3AeSkPonLlsslQ)

- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)

- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文全说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
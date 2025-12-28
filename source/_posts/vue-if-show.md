---
title: Vue进阶系列第4篇-- v-show 和 v-if 的实用解析：搞懂用法，选对场景
date: 2025-12-12 
tags: Vue, JavaScript进阶
category: Vue
---

<!--# Vue 中 v-show 和 v-if 的实用解析：搞懂用法，选对场景 -->

大家好，我是jvxiao。

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第3篇，今天我们说的是Vue中的两个指令--`v-show`和`v-if`。

在 Vue 开发里，控制元素显示或隐藏是很常见的需求，而 v-show 和 v-if 就是专门干这个活的两个核心指令。它们俩目标一致，都是根据条件（比如一个布尔值、返回布尔值的表达式）来决定元素要不要“露脸”，但用法、原理和适用场景却大不相同。如果用错了，可能会影响页面性能，甚至出现意想不到的问题,今天就把这两个指令的来龙去脉说清楚。

## 两者的共同点：目标一致，核心功能相同
不管是 v-show 还是 v-if，本质上都是“条件渲染工具”。简单说，就是你给它们一个“判断标准”，比如“用户是不是管理员”“开关是不是打开的”，它们就会按照这个标准，决定元素该显示还是隐藏。

比如想让一个按钮只在用户是管理员时显示，用 v-if 可以，用 v-show 也可以（只是背后逻辑不一样），代码写法都很简单：
```html
<!-- 用 v-if 实现管理员按钮显示 -->
<button v-if="isAdmin">删除数据</button>
<!-- 用 v-show 实现管理员按钮显示 -->
<button v-show="isAdmin">删除数据</button>
```
上面两段代码，只要 `isAdmin` 为 `true`，按钮就显示；为 `false` 就隐藏，核心效果完全一致。这是它们最基础的共同点，也是很多新手刚开始会混淆的原因。

## 两者的关键区别：一个“藏着”，一个“删了”
这部分是核心，搞懂区别才能用对。两者最本质的差异，在于“隐藏元素时，到底对 DOM 做了什么”。

### 1. 渲染机制：一个操作 DOM，一个修改样式
v-if 是“来真的”条件渲染。如果条件是 false，它会直接把元素及其子节点从 DOM 树里“删掉”——就像把桌子上的杯子拿走，桌子上彻底没有这个杯子了；如果条件变成 true，它再重新创建这个元素，把它放回 DOM 树里。

看这个示例，当 `isShow` 为 `false` 时，你在浏览器开发者工具的 DOM 面板里，根本找不到这个 `<div>` 节点：
```html
<template>
  <div v-if="isShow">我是 v-if 控制的内容</div>
  <button @click="isShow = !isShow">切换显示</button>
</template>

<script>
export default {
  data() {
    return {
      isShow: false // 初始为 false，DOM 中无此元素
    }
  }
}
</script>
```

而 v-show 是“表面功夫”：不管条件是 true 还是 false，元素都会先被渲染到 DOM 树里（桌子上一直有这个杯子），只是条件为 false 时，给元素加个 CSS 样式 `display: none`，把它“藏起来”；条件为 true 时，再去掉这个样式，让它显示出来。

同样的逻辑，用 v-show 实现时，哪怕 `isShow` 为 `false`，DOM 里依然能看到这个 `<div>`，只是多了 `display: none` 样式：
```html
<template>
  <div v-show="isShow">我是 v-show 控制的内容</div>
  <button @click="isShow = !isShow">切换显示</button>
</template>

<script>
export default {
  data() {
    return {
      isShow: false // 初始为 false，DOM 中有此元素，但样式为 display: none
    }
  }
}
</script>
```

### 2. 性能开销：初始渲染和切换成本不一样
因为渲染机制不同，两者的性能开销也反过来了。

**v-if 的初始渲染成本低**：如果一开始条件就是 false，它根本不会创建 DOM，省了创建元素的功夫；但切换成本高——每次条件变化，都要销毁旧元素、创建新元素，还会触发组件的生命周期钩子（比如创建时的 mounted、销毁时的 destroyed），就像反复拿杯子、放杯子，动作多、费时间。

**v-show 的初始渲染成本高**：不管条件怎么样，都要先把 DOM 建出来，相当于不管用不用，先把杯子放桌子上；但切换成本极低——只是改个 CSS 样式，不用动 DOM 结构，就像把杯子扣上、翻开，动作简单、速度快。

### 3. 其他细节：语法支持和适用元素有差异
v-if 更灵活，支持搭配 v-else、v-else-if 做多分支判断。比如可以做“管理员显示面板、普通用户显示中心、游客显示欢迎页”这种多场景切换，逻辑更完整，代码示例如下：
```html
<template>
  <div v-if="role === 'admin'">管理员面板：可修改所有配置</div>
  <div v-else-if="role === 'user'">用户中心：仅查看个人信息</div>
  <div v-else>游客页面：请登录后使用</div>
</template>

<script>
export default {
  data() {
    return {
      role: 'user' // 可切换为 'admin' 或 'visitor' 测试
    }
  }
}
</script>
```

而 v-show 不支持 v-else 和 v-else-if，只能单独使用，没法做多分支判断。另外，v-if 可以作用于 `<template>` 标签（这个标签本身不渲染成真实 DOM，适合批量控制一组元素），示例如下：
```html
<template v-if="isShowGroup">
  <div>元素1</div>
  <div>元素2</div>
  <div>元素3</div>
</template>
```
但 v-show 不行--因为 `<template>` 没有真实 DOM 节点，没法给它加 `display` 样式，自然没法控制隐藏。

## 两者的使用场景：看“切换频率”，选对才高效
知道了区别，选场景就简单了。核心原则就一个：看“条件会不会频繁变化”，以及“要不要让元素彻底消失”。

### 1. v-if：适合“不常切换、要彻底隐藏”的场景
如果条件很少变化，或者需要让元素彻底从 DOM 中消失（避免残留），就用 v-if。

#### 场景1：权限控制（几乎不切换）
像“删除数据”这种只有管理员能看到的按钮，用户登录后权限就固定了，很少变化，用 v-if 合适——非管理员时，按钮直接不渲染，DOM 里没有这个元素，也能避免敏感功能被恶意访问：
```html
<template>
  <button v-if="user.role === 'admin'" @click="deleteData">删除数据</button>
</template>

<script>
export default {
  data() {
    return {
      user: {
        role: 'visitor' // 游客身份，按钮不会渲染
      }
    }
  },
  methods: {
    deleteData() {
      // 删数据逻辑
    }
  }
}
</script>
```

#### 场景2：多分支界面（需要 v-else-if 配合）
比如根据用户角色显示不同的页面内容，只能选 v-if，示例参考上文多分支代码，这里不再重复。

#### 场景3：首屏优化（减少初始 DOM 节点）
页面加载时，很多不紧急的元素（比如非必填的表单项），如果一开始不需要显示，用 v-if 可以减少初始 DOM 节点数量，让页面加载更快：
```html
<template>
  <form>
    <div>必填项：<input type="text" placeholder="姓名" /></div>
    <!-- 非必填项，初始隐藏，点击后才渲染 -->
    <div v-if="showOptional">
      选填项：<input type="text" placeholder="备注" />
    </div>
    <button type="button" @click="showOptional = true">显示备注</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      showOptional: false // 初始不渲染，减少首屏 DOM 数量
    }
  }
}
</script>
```

### 2. v-show：适合“频繁切换、仅视觉隐藏”的场景
如果条件会反复变化，只是需要视觉上隐藏，不需要删除 DOM，就用 v-show。

#### 场景1：频繁切换的下拉菜单
比如点击按钮反复展开/收起的下拉菜单，用 v-show 切换时不卡，体验更好：
```html
<template>
  <div class="menu-wrap">
    <button @click="isMenuOpen = !isMenuOpen">展开菜单</button>
    <!-- 频繁切换，用 v-show 更高效 -->
    <ul v-show="isMenuOpen">
      <li>菜单1</li>
      <li>菜单2</li>
      <li>菜单3</li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isMenuOpen: false
    }
  }
}
</script>

<style>
ul {
  margin: 0;
  padding: 0;
  list-style: none;
  border: 1px solid #ccc;
  padding: 10px;
}
</style>
```

#### 场景2：Tab 切换（高频切换）
Tab 栏是页面中高频切换的组件，用 v-show 控制每个 Tab 的内容，切换时仅修改样式，性能更优：
```html
<template>
  <div class="tab-wrap">
    <button @click="activeTab = 'tab1'" :class="{ active: activeTab === 'tab1' }">Tab1</button>
    <button @click="activeTab = 'tab2'" :class="{ active: activeTab === 'tab2' }">Tab2</button>
    
    <div v-show="activeTab === 'tab1'">Tab1 的内容</div>
    <div v-show="activeTab === 'tab2'">Tab2 的内容</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'tab1'
    }
  }
}
</script>

<style>
.active {
  background: #42b983;
  color: white;
  border: none;
  padding: 5px 10px;
}
</style>
```

##  总结：记住一句口诀，再也不混淆
最后用一句好记的话总结，方便实际开发中快速选择：

**切换少用 v-if，切换多用 v-show；要销毁用 v-if，仅隐藏用 v-show**


其实不用死记硬背，只要想清楚“隐藏时要不要删 DOM”“会不会经常切换”，就能快速做出选择。v-show 和 v-if 没有绝对的好坏，只有“合不合适”，选对了，页面性能和开发效率都会提升。

**【往期精彩】**
- [Vue进阶系列第3篇：什么是SPA(单页面应用)? 如何实现一个简单的SPA?](https://mp.weixin.qq.com/s/ooLgrrFDhm-uFWpNRJtlUA)
- [Vue进阶系列第2篇：双向绑定是什么？它工作原理是什么？如何实现的？](https://mp.weixin.qq.com/s/i3J5wQmtoix2q7gldX9Hcw)
- [Vue进阶系列第1篇：说说对Vue的理解，Vue是什么，有什么作用?](https://mp.weixin.qq.com/s/I-hOIMdmxws1sukiAicg4w)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文全说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [一文搞懂 JavaScript 里 var、let、const 的区别
](https://mp.weixin.qq.com/s/lqn_51NNxx50YgvWkxCtEg)
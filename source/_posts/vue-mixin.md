---
title: Vue进阶系列第11篇--说说你对vue的mixin的理解，它的本质是什么？有什么应用场景？
date: 2026-01-02
tags: Vue, JavaScript进阶
category: Vue
banner_img: /imgs/baners/vue.jfif
index_img: /imgs/baners/vue.jfif
---

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第11篇，今天来讲讲Vue2中已经弃用的一个特性--`Mixin`。`Mixin` 在Vue2 版本中扮演者非常重要的角色，很多 Hack 技巧和代码的复用上都依赖着Mixin。虽然 Vue3版本弃用了，但是，法拉利老了还是法拉利，一起来认识一下吧。

## Mixin是什么
咱们先说说Mixin到底是啥——它是面向对象编程里的一种类，里面装着能重复用的方法。其他类不用费劲继承这个Mixin类，就能直接拿过来用它里面的方法，特别方便。

**Mixin类一般就是个独立的功能模块，咱们需要用这个功能的时候，把它“混”进需要的地方就行**。这样既能重复用代码、少写冗余内容，又不会有多重继承那种绕来绕去的复杂问题，简直是双赢。

### Vue中的Mixin
先看Vue官方的定义，说白了就是：
mixin（混入），是种特别灵活的方式，能把Vue组件里那些能重复用的功能，单独抽出来分发。

它本质就是个JS对象，里面能放组件里的各种功能，比如`data`、`components`、`methods`、`created`、`computed`这些，你想放啥功能都能放。咱们只要把多个组件都要用的功能，写成一个对象，塞进mixins选项里，这个组件就能用上这个功能了——Mixin里的所有选项，都会自动合并到组件自己的选项里，不用额外写别的代码。

## Mixin的分类

Vue里的Mixin分两种：局部混入和全局混入，咱们一个个说。

#### 1. 局部混入
先定义一个Mixin对象，里面放好组件能用的data、methods这些内容：
```javascript
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}
```

然后组件通过mixins属性，把这个Mixin引进来就能用：
```javascript
Vue.component('componentA',{
  mixins: [myMixin]
})
```

这个组件一用，就会自动包含Mixin里的方法，创建组件的时候会自动执行created钩子，顺带就把hello方法给执行了，不用咱们手动调用。

#### 2. 全局混入
全局混入更简单，用Vue.mixin()方法就能实现，代码长这样：
```javascript
Vue.mixin({
  created: function () {
      console.log("全局混入")
    }
})
```

这里必须提醒一句：全局混入会影响所有的Vue组件，包括第三方组件，所以千万别随便用！一般只有写插件的时候，才会用到全局混入。

#### 3. 注意事项
① 要是组件里的选项和Mixin里的选项重名了，合并的时候就认组件自己的选项，直接把Mixin里的给覆盖了；
② 但如果重名的是生命周期钩子（比如created、mounted），就不会覆盖，而是排成一队执行——先跑Mixin里的钩子，再跑组件自己的，顺序别搞反了。

## Mixin的使用场景
咱们平时做开发的时候，经常会碰到多个组件里有一样或差不多的代码，而且这些代码的功能是独立的，和组件里其他功能没啥关联。这时候就别傻呵呵地在每个组件里都写一遍了，用Mixin把这些重复代码抽出来统一管理，省不少事。

给大家举个简单的例子，一看就懂：

1. 写一个弹窗组件Modal，用isShowing这个变量控制它显示还是隐藏：
```javascript
const Modal = {
  template: '#modal',
  data() {
    return {
      isShowing: false
    }
  },
  methods: {
    toggleShow() {
      this.isShowing = !this.isShowing;
    }
  }
}
```

2. 再写一个提示框组件Tooltip，也是用isShowing控制显示隐藏，代码几乎一样：
```javascript
const Tooltip = {
  template: '#tooltip',
  data() {
    return {
      isShowing: false
    }
  },
  methods: {
    toggleShow() {
      this.isShowing = !this.isShowing;
    }
  }
}
```

一眼就能看出来，这两个组件的核心功能完全一样，都是用isShowing和toggleShow控制显示隐藏。这时候Mixin就该出场了，正好派上用场。

3. 把重复的代码抽出来，写成一个Mixin：
```javascript
const toggle = {
  data() {
    return {
      isShowing: false
    }
  },
  methods: {
    toggleShow() {
      this.isShowing = !this.isShowing;
    }
  }
}
```

4. 两个组件只要引进这个Mixin，就能直接用里面的功能，不用再重复写代码了：
```javascript
const Modal = {
  template: '#modal',
  mixins: [toggle]
};
 
const Tooltip = {
  template: '#tooltip',
  mixins: [toggle]
}
```

从这个小例子就能看出来，Mixin用来封装重复功能是真的香，又方便又实用，还能减少代码冗余。


## 总结
Mixin本身不是什么特别新奇的东西，只不过在不同场景下有不同的名字而已。例如Windows系统重我们称之为补丁，某些游戏中我们称之为外挂，但它们本质都是一样的。它们都是通过在最小改动原有代码的情况下，通过注入新的状态，新的动作，来改变原有的执行逻辑。

最后，吐槽一句，不要让Mixin嵌套层数超过3层。**如果你发现有人嵌套超过3层，那么你最好的处理方式就是--把它加到4层、5层，我称之为同归于尽处理法**。

**【往期精彩】**

- [说说JavaScript中ES6 module(模块化)的诞生背景、核心语法及应用场景](https://mp.weixin.qq.com/s/pe4fywIh7WdigHuwRiqDlQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)
- [Vue进阶系列第1篇：说说对Vue的理解，Vue是什么，有什么作用](https://mp.weixin.qq.com/s/I-hOIMdmxws1sukiAicg4w)

- [聊聊ES6里的Promise：简单理解和实际用法
](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)

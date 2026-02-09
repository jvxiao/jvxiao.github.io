---
title: 图解 Vue2 diff 算法：首尾四向对比核心逻辑全拆解, 告别死记硬背八股文
date: 2026-01-07
tags: Vue, JavaScript进阶
banner_img: /imgs/baners/diff.png
index_img: /imgs/baners/diff.png
---

<!-- # 图解 Vue2 diff 算法：首尾四向对比核心逻辑全拆解 -->

这是[Vue从入门到精通](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU2MjU3MzI0MA==&action=getalbum&album_id=4276738695946223617#wechat_redirect)系列文章的第13篇，上一篇文章中提到了虚拟DOM, 今天花点时间重点讲讲里面非常核心的内容--`diff算法`。

很多人在看到这一块的时候都感觉头大，但为了面试还是硬着头皮去背那些八股文。与其花时间背八股文，不如克服畏难心里，跟着我一起来看看`diff`算法的奥秘，它是否又真的有那么难。


讲`diff算法`首先我们需要明确一个问题，它是为了解决什么问题才引入了。[前文](https://mp.weixin.qq.com/s/hY97PUXy_WQeagpaYtzk9g)我们提到为了解决直接减少直接操作原生DOM引入了虚拟DOM，当数据变化时，Vue 会生成新的虚拟 DOM（VNode），然后通过 Diff 算法对比新旧 VNode，只把真正变化的部分更新到真实的网页上，而不是推倒重来。

是的，这就是为什么引入`diff算法` 原因-- **解决 DOM 操作带来的“昂贵”代价**, 而不是给你增加面试难度。


Vue中的`diff算法`也不是从石头里蹦出来的，也不是尤大从睡一觉从梦中得到灵感想出来。Vue 在 2.0 版本引入虚拟 DOM 时，并没有从零开始写算法，而是深度借鉴了一个名为 **Snabbdom** 的轻量级库，它引入了著名的 `双端 Diff 算法`，而这也是本文要讲的重点。

## diff算法的比较方式

diff算法在比较的策略上有两个特点：**深度优先，同层比较**。

**深度优先**的意思很好理解，如果要比较两个新旧节点，那么就要将他们所有的子节点进行比较，一层一层递归。

**同层比较**即比较只会在同层级进行，不会跨层级比较，如图。


![](https://fastly.jsdelivr.net/gh/bucketio/img12@main/2026/01/07/1767790005524-10a7f3e9-fd41-44d0-ab17-552f2c49a543.png)

## 算法核心

`双端diff算法` 的核心就在于四个指针，通过四个指针从两端向中间“夹击”，尽可能复用已有的 DOM。在 Vue 2 中，处理子节点更新的核心函数是 `updateChildren`, 大家可以对照源码来理解下面的内容。


### **四个指针**
在对比新旧两组子节点（Children）时，Vue 定义了四个变量
- **oldStart**：指向旧列表的第一个节点。
- **oldEnd**： 指向旧列表的最后一个节点。
- **newStart**：指向新列表的第一个节点。
- **newEnd**：指向新列表的最后一个节点。

算法会进入一个 while 循环，条件是：旧头指针 ≤ 旧尾指针 且 新头指针 ≤ 新尾指针

### 五步对比法

旧列表的两个指针指向2个节点，新列表的2个指针也指向，将它们两两进行比较，就有4种情况，外加一种都没匹配上的场景，就是刚好5种场景。

- **旧头对新头**（oldStart vs newStart）：如果这2个指针所指向的节点相同，那么直接复用旧的DOM节点，更新属性即可，`oldStart` 和 `newStart` 都加1，相当于都**右移**了一位。

- **旧尾对新尾**（oldEnd vs newEnd）：如果这2个指针所指向的节点相同，那么也是直接复用旧的DOM节点，更新属性即可，`oldEnd` 和 `newEnd` 都**减1**，相当于都**左移**了一位。

- **新头对旧尾**（oldStart vs newEnd）：如果这两个节点相同，说明`旧列表的第一个节点变成了新列表的最后一个`，需要复用 DOM，并将该 DOM 移动到 oldEnd 对应的真实 DOM 之后。`oldStart`**加1**，`newEnd`**减1**。

- **旧尾对新头**（oldEnd vs newStart）： 如果这两个节点相同，说明`旧列表的最后一个节点 变成了 新列表的第一个`，复用 DOM，并将该 DOM 移动到 oldStart 对应的真实 DOM 之前。`oldEnd`**减1**，`newStart`**加1**.

- **都没匹配上**：那么这是一个全新的元素，直接创建新 DOM 并插入，`newStart`**加1**

你看，整理下来，其实也不复杂啊，觉得难往往是被自己的畏难心理给吓到了。为了让大家更好理解，下面举个例子吧。

## 图解diff算法

我们有新旧两个虚拟节点列表如下图所示

![](https://fastly.jsdelivr.net/gh/bucketio/img15@main/2026/01/07/1767791984485-49f868d1-8682-4970-a108-e66e4fe09756.png)

- 第一次循环后，发现旧节点D与新节点D相同（**旧尾对新头**），直接复用旧节点D作为diff后的第一个真实节点，同时旧节点endIndex移动到C，新节点的 startIndex 移动到了 C。


![](https://fastly.jsdelivr.net/gh/bucketio/img1@main/2026/01/07/1767792183450-43a4fc51-db80-4322-98da-5cc6252cc51b.png)

- 第二次循环后，同样是旧节点的末尾和新节点的开头(都是 C)相同（**旧尾对新头**），同理，diff 后创建了 C 的真实节点插入到第一次创建的 B 节点后面。同时旧节点的 endIndex 移动到了 B，新节点的 startIndex 移动到了 E

![](https://fastly.jsdelivr.net/gh/bucketio/img5@main/2026/01/07/1767792247462-2442aa46-6634-4569-a50f-1cb89863f27d.png)

- 第三次循环中，发现E没有找到（**未匹配到**），这时候只能直接创建新的真实节点 E，插入到第二次创建的 C 节点之后。同时新节点的 startIndex 移动到了 A。旧节点的 startIndex 和 endIndex 都保持不动

![](https://fastly.jsdelivr.net/gh/bucketio/img8@main/2026/01/07/1767792361143-fc35e043-a20c-4657-adf5-00c7bd6abaaa.png)

- 第四次循环中，发现了新旧节点的开头(都是 A)相同（**新头对旧头**），于是 diff 后创建了 A 的真实节点，插入到前一次创建的 E 节点后面。同时旧节点的 startIndex 移动到了 B，新节点的 startIndex 移动到了 B

![](https://fastly.jsdelivr.net/gh/bucketio/img13@main/2026/01/07/1767792423397-5227fd99-c9aa-431a-8208-e8990706e106.png)

- 第五次循环中，情形同第四次循环一样（**新头对旧头**），因此 diff 后创建了 B 真实节点 插入到前一次创建的 A 节点后面。同时旧节点的 startIndex 移动到了 C，新节点的 startIndex 移动到了 F


![](https://fastly.jsdelivr.net/gh/bucketio/img9@main/2026/01/07/1767792447763-d95a7040-063e-4c43-86cb-c43c10775f06.png)

新节点的 startIndex 已经大于 endIndex 了（**未匹配**），需要创建 newStartIdx 和 newEndIdx 之间的所有节点，也就是节点F，直接创建 F 节点对应的真实节点放到 B 节点后面


![](https://fastly.jsdelivr.net/gh/bucketio/img7@main/2026/01/07/1767792496225-d805f280-a626-4972-bdfa-83c00634fb20.png)


## 写在最后

虽然 Vue 2 已于 2023 年底停止官方维护，但其双端 Diff 算法依然是理解虚拟 DOM 演进的关键基础，也是面试中经常考察的点。

学框架，学知识，不要搞个人崇拜，不要有畏难心理。强如尤大这样的大佬想必也写过`hello world`，在一些问题上也磕磕绊绊过，这又算什么呢？

**Stay foolish, stay hungry.** 跟着作者一起披荆斩棘吧。

**【往期精彩】**

- [说说JavaScript中ES6 module(模块化)的诞生背景、核心语法及应用场景](https://mp.weixin.qq.com/s/pe4fywIh7WdigHuwRiqDlQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)
- [Vue进阶系列第1篇：说说对Vue的理解，Vue是什么，有什么作用](https://mp.weixin.qq.com/s/I-hOIMdmxws1sukiAicg4w)

- [聊聊ES6里的Promise：简单理解和实际用法
](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
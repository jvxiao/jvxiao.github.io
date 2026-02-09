---
title: 把大象装进冰箱分几步？Node.js 大文件“切片上传”深度解析
date: 2026-01-13 
banner_img: /imgs/baners/big-file.jfif
index_img: /imgs/baners/big-file.jfif
---
在用 Node.js 和 JavaScript 开发文件上传功能时，如果文件特别大（比如好几个 GB 的高清视频或者压缩包），一次性上传不仅慢，而且万一中途网络断了，就得从头再来，非常让人崩溃。

这时候就需要“断点续传”。**断点续传说白了，就是把大文件切成一小块一小块来传，传完了再拼起来**；中间万一断了，下次只传还没成功的那几块就行。下面用大白话加简单例子，把实现断点续传的几个核心步骤说清楚。

## 文件切片：把大象装进冰箱分几步？
第一步得把大文件“切碎”。在浏览器里，文件对象（File）其实是 Blob 的一种，它自带一个slice 方法，可以像切火腿肠一样把文件切成一个个小片段。

• **做法**：设定一个固定的切片大小（比如 5MB），用循环把文件切开。

• **例子**：
```javascript
const file = bigFile // 用户选择的大文件
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB一块
let chunks = [];
let cur = 0;
while (cur < file.size) {
    chunks.push(file.slice(cur, cur + CHUNK_SIZE));
    cur += CHUNK_SIZE;
}
// 这样你就得到了一个装满“文件块”的数组
```

## 生成指纹：怎么知道这块是你的？

文件切好了，但服务器怎么知道这些碎块属于哪个文件？又怎么知道哪些传过，哪些没传？这就需要给文件办个“身份证”——计算 MD5 哈希值。

**做法**：利用 spark-md5 等库，根据文件内容生成一个唯一的字符串。只要文件内容不变，这个指纹就不变。

 **核心逻辑**：有了这个指纹，服务器就能记录：“身份证号为 XXX 的文件，第 1、2、5 块已经收到了”。
 
## 秒传与续传：已经有的就不传了
这是断点续传最“聪明”的地方。在正式上传前，前端先发个请求问问服务器：“这个指纹的文件你那儿有吗？传到第几块了？”

**秒传**：服务器说“我这儿已经完整存过这个文件了”，前端直接提示上传成功。

 **续传**：服务器说“我这儿只有第 1、2 块”，前端就从第 3 块开始传。

```javascript
// 上传前先校验
const { uploadedList, isExist } = await verifyUpload(fileHash); 

if (isExist) {
    alert("秒传成功！");
    return;
}
// 只过滤掉已经上传过的块，传剩下的
const requestList = chunks.filter(chunk => !uploadedList.includes(chunk.hash))
    .map(chunk => uploadAction(chunk));
```

## 后端合并：传完了得拼回来

Node.js 后端的工作主要是“接货”和“组装”。

 **接货**：用 multer 或 formidable 等中间件，把前端传来的切片存在一个以“文件指纹”命名的临时文件夹里。
 
**组装**：当所有切片都传完了，前端发个“合并”通知。后端用 Node.js 的`fs.createReadStream` 和 `fs.createWriteStream` 把这些切片按顺序读出来、写进同一个文件里。

**清理**：合并完成后，把那一堆没用的临时切片删掉，省空间。

## 总结：该怎么做？
1. **前端切片**：用 file.slice 物理切割。
2. **计算哈希**：给文件生成唯一指纹，作为续传的依据。
3. **校验状态**：传之前问问服务器，跳过已存在的切片。
4. **并发上传**：用 Promise.all 或限制并发数的方式，把切片塞给后端。
5. **后端合并**：Node.js 负责把切片“合体”，大功告成。

有了这一套逻辑，无论网络怎么抖动，大文件上传都能稳如泰山！如果你想直接上手，可以参考开源成熟的库如 Uppy 或 Simple-Uploader 进一步学习。

**【往期精彩】**
- [原生 DOM 真的慢吗？为什么现代框架都迷恋虚拟 DOM？](https://mp.weixin.qq.com/s/hY97PUXy_WQeagpaYtzk9g)
- [Vue进阶系列第9篇--你真的懂nextTick吗？我表示很怀疑](https://mp.weixin.qq.com/s/XATTpbsFwd6vuw5EqTUCfQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)

- [聊聊ES6里的Promise：简单理解和实际用法
](https://mp.weixin.qq.com/s/9pnvL5fTO5U9yT-1mPfeQw)
---
title: 搭建个人博客系列--(2) 动手搭建自己的第一个博客站点
date: 2025-05-27 21:02:53
tags: 
   - 博客搭建
   - 个人IP
keywords: [博客搭建, 免费博客平台, 托管博客, 模板建站]
---

在上文[什么每个人都该有个数字自留地](./build-personal-blog1.md)中讲解了为什么我们要写博客以及为什么要搭建自己的独立博客，今天我们将要撸起袖子，开始干活了。认真看完本文，按照文中的步骤去做，你将拥有一个属于你自己的博客站点。

## 搭建博客的几种方式

说到博客，不熟悉的人对它的印象可能是微博，QQ空间日志，以及如CSDN，博客园之类的站点的样子，功能丰富，用户上手也快，一部手机就能完成所有的事情。

上面所提到的站点，都是标准的商业型应用了，而作为个人，没有必要浪费过多时间在各种花哨功能构建上。

个人博客，最重要的是内容，是价值输出！

目前呢，搭建个人博客的方式主要有以下几种：

  - **社区平台**：像`博客园`，`CSDN`, `知乎`，`微博`，`公众号`，`QQ空间`等，注册一个账号就Okay了。当然了，既然是社区平台，也就意味着没有独立的网站。

  - **独立博客**: 有自己的网站，博客框架的搭建，内容的发布，都是由自己完成。对未接触过编程的人来说，上手有一定难度。比较适合有一定编程基础，然后又想挑战一下自己那种的同学。

作为一个明日的大佬，自己动手整个独立博客，想必对大家来说也不是什么难事。

## 考虑因素

既然选择建独立博客，那么我们需要考虑以下几个问题了：

  - 是否需要建立数据库，使用前后端配合的方式完成博客的增删改查？
  - 是否考虑购买服务器，还是进行服务托管?
  - 打算投入多少时间和金钱去建站?
 

在考虑这些问题的时候，同时问问自己，自己的博客是否需要很多功能，还是作为一个简单记录自己所思所想的自由平台。

个人认为，作为程序员，简洁就是最好的风格！

因此，我更倾向选择的是不使用前后端，使用静态内容托管的方式来搭建自己的博客。正如前面所说的，一个个人博客站重要的是内容输出，戏台子搭得大和漂亮没有多少用，重要的是你的戏是否能够吸引观众。

## 技术选型

既然是使用静态托管平台，那么可供选择的方案有很多：
  
  - **[GitHub Pages](https://pages.github.com/)**: 自带域名可 https 访问; 可配置自定义域名
  - **[Bitbucket Cloud](https://confluence.atlassian.com/bitbucket/publishing-a-website-on-bitbucket-cloud-221449776.html)**: 能且只能通过 https 协议访问; 无法自定义域名
  - **[GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/index.html)**: 同样跟 GitHub Pages 的功能一样
  - **[Netlify](https://www.netlify.com/)**: 可以使用 CLI 上传代码; 支持自动构建

在这里，比较推荐 Github Page 和 Netlify, 主要原因有两个：一个是比较能装，另一个是比较能装！

## Github page 建站

下面演示一下Github Page生成一个简单的个人站点名称，Netlify 可以自行探索一下，这里不做讲解。

使用Github Page 建站，首先，你需要有一个Github 账号，如果没有，先去注册一个，注册时记得取个好一点的名字，后面有用。

- **登录github, 新建一个与你名字同名的repository**

登录到Github首页之后，点击右上角 `+` 号，选择 `new repository` 跳转到新建仓库页面。

![step1](https://image.baidu.com/search/down?url=https://tvax4.sinaimg.cn/large/0071fJItgy1i21b6hl5rlj31i70xpe0n.jpg)

其中，`repository name` 一定要是 `你的英文名称.github.io`，比如我的英文名称是`jvxiaome`， 那么我的仓库名称就是 `jvxiaome.github.io`，这个后面也是访问博客站点首页的网址，所以前面说取一个好听的名字，是有道理的。

仓库的可见性选择 `Public`, 另外下面的 `Add a README file` 也可以勾上，会方便一些，至于剩下的内容，看个人需要，不清楚的可以暂时不填，最后确认 `Create repository`。



![step2](https://image.baidu.com/search/down?url=http://tvax3.sinaimg.cn/large/0071fJItgy1i21ba20naej31aa1ah1ba.jpg)


- **编辑一个你的博客主页**

进入到刚刚新建的仓库，点击`add file` 新建一个文件


![step3](https://image.baidu.com/search/down?url=http://tvax2.sinaimg.cn/large/0071fJItgy1i21batizraj31gl0ufn95.jpg)

文件名称取名为 `index.html`, 这个文件中包含的就是博客主页的内容了，具体想写什么可以自由发挥

![step4](https://image.baidu.com/search/down?url=http://tvax4.sinaimg.cn/large/0071fJItgy1i21bb808bfj31zr0thgzj.jpg)

写完之后点击 `commit changes` 提交内容到仓库，到这一步，你的博客站就算是基本建立起来了
![step5](https://image.baidu.com/search/down?url=http://tvax2.sinaimg.cn/large/0071fJItgy1i21bc83oxuj31eu0vyduh.jpg)


- **访问验证**

既然博客站点已经建好了，此时是不是特别期待访问自己的博客首页呢。

正如前面所说的，你的仓库名称（你的英文名称.github.io）就是你的博客首页名称，以上面的例子来说，访问 `jvxiaome.github.io` 就能直接看见首页效果了。

虽然整个页面都在诉说着朴素(~~丑~~)，但一点也不影响你拥有一个属于自己博客站点的那份喜悦。都18+了，怎么能没有自己的网站呢。

![step6](https://image.baidu.com/search/down?url=http://tvax3.sinaimg.cn/large/0071fJItgy1i21bcpjfruj310d0k2go1.jpg)

## 写在最后

有了自己的博客网站，还有了域名，而这一切，不是个结束，而是开始。

如何使用工具，使用主题，搭建一个简洁，漂亮的[博客网站](![my-sit](https://image.baidu.com/search/down?url=http://tvax1.sinaimg.cn/large/0071fJItgy1i21bddtgjjj32cu1h8x6q.jpg))，这将是下一篇文中的重点。






<!-- https://github.com/lmk123/blog/issues/55 -->


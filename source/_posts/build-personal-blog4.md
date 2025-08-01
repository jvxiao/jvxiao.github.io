---
title: 搭建个人博客系列--(4) 利用Github Actions自动构建博客
date: 2025-06-14 15:37:45
category: 搭建博客
tags: 
   - 博客搭建
   - 个人IP
keywords: [Github Actions, 博客搭建, 免费博客平台, 托管博客, 模板建站]
---

经过前面的系列文章的学习和实践，相信你已经成功的利用Hexo构建自己的博客并且部署到了Github上。

目前整个发布博客的流程是，用markdown文件写好博客，然后使用Hexo编译成html, 最后将public下的内容部署到github上。整个过程虽然不复杂，但每次都要重新在本地编译然后上传，多少有些繁琐。

这个时候我们可以寻求一种方法，实现博客自动编译和内容的部署，它就是 **Github Actions**。

## 什么是Github Actions?

GitHub Actions 是 GitHub 提供的 持续集成与持续部署（CI/CD）平台，允许开发者直接在 GitHub 仓库中自动化构建、测试、打包、发布代码，实现软件开发流程的自动化。通过定义 YAML 格式的配置文件（称为 Workflow），开发者可以定制化代码仓库的自动化工作流程，例如代码提交时自动运行测试、合并分支时自动部署到服务器等。

## 核心概念

### Workflow（工作流）
- 一个 Workflow 是一个自动化流程，由一个或多个 Job 组成，通过 YAML 文件（.github/workflows/ 目录下）定义。

- 可以配置 Workflow 在特定事件（如代码推送、Pull Request 创建、Issue 评论等）触发，或按计划定期运行。

### Job（任务）

- 一个 Job 是 Workflow 中的独立任务，包含一系列 Step，默认在一个虚拟机（Runner）上运行。
- Jobs 可以并行或按顺序执行，支持定义依赖关系（如 Job B 等待 Job A 完成）。

### Step（步骤）

- Step 是 Job 中的最小执行单元，可以是运行 Shell 命令（如 npm install）或调用 Action（预定义的功能模块）。

- 例如：拉取代码、安装依赖、运行测试、部署到服务器等。

### Action（动作）

- 可复用的代码模块，用于完成特定功能（如检出代码、设置环境、发送通知等）。

- GitHub Marketplace 提供大量公开 Actions（如 actions/checkout、actions/setup-node），开发者也可自定义 Actions。

## 举个例子

以下是一个简单的 Workflow 配置（.github/workflows/node.js.yml），用于在 Node.js 项目中执行测试：

```
name: Node.js CI
on: [push] # 当代码推送时触发

jobs:
  build:
    runs-on: ubuntu-latest # 使用 Ubuntu 最新版虚拟机
    steps:
      - name: Checkout code
        uses: actions/checkout@v4 # 拉取代码到 Runner

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20 # 指定 Node.js 版本

      - name: Install dependencies
        run: npm install # 运行 Shell 命令安装依赖

      - name: Run tests
        run: npm test # 运行测试脚本
```

## 编写自动编译并部署博客脚本

如果需要实现博客的自动化部署，那么有两个步骤要走，一是在push新内容的时候编译，二是编译之后部署到Github pages上。

### 编译博客

在之前的内容中，我们知道编译markdown文件成html文件使用的命令是 `hexo build`, 后来我们又将其写入到 `package.json`里的`scripts`中并取名为 `build`, 那么在执行编译的时候可以使用 `npm run build`来替代之前的命令。

下面我们开始来写自动化脚本，首先在根目录下新建一个`./github/workflows/update-blogs.yml`文件，整体框架可以参考上面例子，因为使用的都是 Node 环境。

完成一个build操作，要执行的操作分4步骤
 - 拉取代码
 - 配置Node环境
 - 安装依赖
 - 执行编译操作
 - 上传编译内容
```
name: automaticly update blogs
on: 
 branches:
  - master # master分支
jobs:
  build:
    runs-on: ubuntu-latest # 使用 Ubuntu 最新版虚拟机
    steps:
      - name: Checkout code
        uses: actions/checkout@v4 # 拉取代码到 Runner

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20 # 指定 Node.js 版本

      - name: Install dependencies
        run: |
          npm install
          npm install -g hexo-cli

      - name: Run build
        run: npm run build # 运行编译操作
      
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3 # 上传public目录内容
        with:
          path: ./public

```

修改点：

- 不想任意分支提交都执行该操作，所以选择  `master` 分支有提交的时候执行。

- 安装依赖依赖，需要在全局安装 `hexo-cli`

- 上传内容使用的是Github Actions 官方的一个专用 Action: actions/upload-pages-aartifact, 主要用于 将静态网站文件打包并上传为 GitHub Pages 部署所需的 “工件”。


### 部署博客

在成功编译博客之后，也是通过一个官方的Action来实现博客内容的部署

```
name: automaticly update blogs
on: 
 branches:
  - master # master分支
jobs:
  build:
    ...

  # 部署博客  
  deploy:
    needs: build   # 依赖上一个build操作
    permissions:   # # 需授予 Pages 写入权限和生成部署令牌的权限
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:         # 使用actions 部署
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

## 测试验证

编写完脚本之后，提交代码至 `master` 分支，在Github对应的仓库分支上可以看见已经有对应的workflow，如作者的脚本配置。

![actions](https://www.jvxiao.cn/imgs/build-blog4/actions.png)

如果左侧出现刚才脚本的名称 `automaticly update blogs`, 则说明已经配置成功了，可以通过提交内容到 master分支来检验脚本的执行情况。


## 写在最后

到目前为止，整个搭建个人博客系列文章已经实现了个人博客站点的搭建和自动化部署，相信此时的你也搭建好了自己的博客站点。

正如之前所说，博客站点最重要的就是价值输出，也就是要勤写内容。但是有时候，酒香也怕巷子深，如何让更多的人看到自己写的文章，增加个人影响力，这将是后面将要讨论的问题，请持续关注！
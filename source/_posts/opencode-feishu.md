---
title: 3行命令，将 opencode 与移动端飞书连接起来
date: 2026-06-12 22:28:34
tags: opencode, 飞书
---

使用 opencode 做事已经有大半年，今天尝试使用飞书连接opencode, 实现在手机上直接操控opencode在电脑本地做事。

我尝试了两种方式，目前已经成功跑通了其中一种。

## 方案一： 独立模式，使用 @neomei/opencode-feishu （推荐）

使用这种模式是最快的，我个人也更推荐使用这一个，只需要 3,4行代码，就能够实现opencode与飞书的连接。

### **安装**
```bash
npm install -g @neomei/opencode-feishu
```

### **扫码自动配置**
```bash
opencode-feishu setup
```
执行上述命令将会在屏幕上生成一个二维码，使用手机端飞书扫一扫，在选择的时候不要选择已有应用，直接选择新应用，以免之前的旧应用配置不对，比如权限没配好之类的。

### **启动**
```bash
opencode-feishu start
```
成功启动之后，在手机飞书上打开刚刚新的应用，发送一条消息，在电脑端可以接收到并能够返回结果给移动端，说明配置Okay了。

>如果在这一步报无法连接opencode错误，那么需要做两件事情。
>
>1. 使用 `opencode serve` 开启opencode 服务模式，服务的地址一般是： `http://127.0.0.1:4096`,
>2. 将上述地址填入 `C:\Users\用户名\.config\opencode\feishu.json` 中的 `opencodeUrl`字段中，并保存
>3. 重新执行启动命令

### **后台常驻**
```bash
opencode-feishu start --daemon
```
如果希望该服务常驻在后台，则使用上述命令替换。

## 方案二：使用opencode的飞书插件 opencode-feish

使用该方案需要手动去飞书开发者后台创建一个应用获取 appId 以及 appScret, 同时要将机器人添加到该应用中。并配置对应的权限。

下面是opencode的配置详细过程：

### 1. 创建自建应用

- 访问：https://open.feishu.cn/ → 开发者后台 → 创建「企业自建应用」
- 应用类型：机器人 → 发布为「企业内部应用」


### 2. 开启机器人能力
- 应用能力 → 机器人 → 开启「启用机器人」
- 机器人名称：OpenCode助手；头像自定义

### 3. 开通所需要的权限
申请权限（必须）

im:message（发消息）
im:message.p2p_msg:readonly（读私聊）
im:message.group_at_msg:readonly（读群 @）
im:message:send_as_bot（机器人身份发消息）
contact:user.base:readonly（用户信息）

### 4. 配置事件订阅（长连接）

事件订阅 → 选择「使用长连接接收事件」（不要 Webhook）
添加事件：im.message.receive_v1（接收消息）、im.chat.member.bot.added_v1（机器人进群）

### 5. 发布应用 & 获取凭证

- 版本管理 → 提交审核 → 发布（企业内部）
- 凭证与基础信息 → 复制：
  + App ID（cli_xxxx…）
  + App Secret（长串密钥）

### 6. 配置opencode

1).安装对应插件
```bash
npm install -g opencode-feishu
```

2). 配置启动该插件
编辑 `~/.config/opencode/opencode.json` 文件
```json
{
  "plugin": ["opencode-feishu"]
}
```

3).配置飞书凭证
新建 `~/.config/opencode/plugins/feishu.json`文件，将在飞书后台的凭证信息填写进去

```json
{
  "appId": "cli_xxxxxxxxxxxxxxxxxxxx",
  "appSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

最后重启一下opencode。

遗憾的是目前该方案我还没跑通，暂时还未定位到问题，等后续跑通了再详细说说踩坑过程。


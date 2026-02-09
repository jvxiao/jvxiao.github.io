---
title: 吃透 XSS/CSRF/SQL 注入：Web 安全防护实战手册
date: 2026-01-19
banner_img: /imgs/baners/web-attack.jfif
index_img: /imgs/baners/web-attack.jfif
---


Web应用程序的安全性是任何基于Web业务的核心命脉，哪怕是代码里一个不起眼的小bug，都有可能导致用户隐私信息泄露。下面我们就聊聊常见的Web攻击手段，以及对应的防护思路。

## 一、什么是Web攻击
Web攻击（Web Attack），简单说就是针对用户上网行为或者网站服务器等设备发起的攻击行为。
常见的攻击操作包括植入恶意代码、修改网站权限、窃取网站用户隐私信息等。

而**站点安全**，就是为了保护站点不被未授权访问、使用、修改和破坏，所采取的一系列行为和实践。

我们日常开发中最常遇到的Web攻击方式主要有三种：
- XSS (Cross Site Scripting) 跨站脚本攻击
- CSRF（Cross-site request forgery）跨站请求伪造
- SQL注入攻击

## 二、XSS 跨站脚本攻击
XSS 全称跨站脚本攻击，它的核心是允许攻击者把恶意代码植入到提供给其他用户使用的页面中。

这个攻击过程涉及三方角色：**攻击者**、**客户端**、**Web应用**。
攻击的最终目标很明确——盗取存储在客户端的cookie，或者其他网站用于识别客户端身份的敏感信息。一旦拿到这些信息，攻击者甚至可以直接假冒合法用户，和网站进行交互操作。

### 举个直观的例子
有一个搜索页面，功能是根据URL参数展示搜索关键词，核心代码长这样：
```html
<input type="text" value="<%= getParameter("keyword") %>">
<button>搜索</button>
<div>
  您搜索的关键词是：<%= getParameter("keyword") %>
</div>
```
看起来没啥问题，但如果用户输入的不是正常关键词，而是一段恶意代码呢？

比如输入这个内容：`"><script>alert('XSS');</script>`
这段内容会被直接拼接到HTML里返回给浏览器，最终渲染的代码就变成了这样：
```html
<input type="text" value=""><script>alert('XSS');</script>">
<button>搜索</button>
<div>
  您搜索的关键词是："><script>alert('XSS');</script>
</div>
```
浏览器没办法分辨这段`<script>`代码是恶意的，只能乖乖执行。这只是弹出一个提示框，如果换成**窃取cookie并发送到黑客服务器**的代码，后果不堪设想。

### XSS攻击的分类
根据攻击来源，XSS可以分成三类：存储型、反射型、DOM型。

#### 1. 存储型XSS
这是危害最大的一种XSS攻击，攻击步骤如下：
1. 攻击者把恶意代码提交到目标网站的数据库中
2. 用户打开目标网站时，服务端从数据库取出恶意代码，拼接到HTML里返回给浏览器
3. 用户浏览器解析执行HTML时，混在里面的恶意代码也会被执行
4. 恶意代码窃取用户数据，发送到攻击者的服务器，或者冒充用户调用网站接口

这种攻击常见于有用户数据存储功能的场景，比如论坛发帖、商品评论、用户私信等。

#### 2. 反射型XSS
攻击步骤和存储型类似，但核心区别在于**恶意代码的存储位置**：
1. 攻击者构造包含恶意代码的特殊URL
2. 用户打开这个URL，服务端从URL里取出恶意代码，拼接到HTML返回
3. 浏览器执行HTML，触发恶意代码
4. 完成攻击操作

反射型XSS的恶意代码存在URL里，而存储型的恶意代码存在数据库里。
它常见于通过URL传递参数的功能，比如网站搜索、页面跳转。
因为需要用户主动点击恶意URL才能触发，攻击者通常会用诱导手段让用户点进去。

另外，POST请求也能触发反射型XSS，但需要构造表单提交页面，条件比较苛刻，所以很少见。

#### 3. DOM型XSS
这种攻击和前两种的本质区别是：**恶意代码的取出和执行，全程都在浏览器端完成**，属于前端JavaScript自身的安全漏洞，和服务端无关。

攻击步骤如下：
1. 攻击者构造包含恶意代码的特殊URL
2. 用户打开这个URL
3. 浏览器解析执行页面，前端JS从URL中取出恶意代码并执行
4. 恶意代码窃取用户数据或冒充用户操作

### XSS的预防方案
XSS攻击有两个核心要素：攻击者提交恶意代码、浏览器执行恶意代码。

很多人会想到在前端过滤用户输入，但这种方式不靠谱——攻击者可以绕开前端，直接构造请求提交恶意代码。

如果在后端写入数据库前过滤，又会遇到新问题：不同场景对内容的显示需求不同。比如用户输入`5 < 7`，转义后变成`5 &lt; 7`，在HTML里能正常显示，但赋值给JS变量或者用于计算内容长度时，就会出问题。

所以，预防XSS的核心思路是**防止浏览器执行恶意代码**，具体可以这么做：
1. 尽量不用`.innerHTML`、`.outerHTML`、`document.write()`这类方法插入不可信数据，优先用`.textContent`、`.setAttribute()`
2. 如果用Vue/React技术栈，别随便用`v-html`/`dangerouslySetInnerHTML`，能从渲染阶段规避innerHTML的XSS隐患
3. 警惕DOM中的内联事件监听器（`onclick`、`onerror`等）、`<a>`标签的`href`属性，以及JS的`eval()`、`setTimeout()`、`setInterval()`——这些API会把字符串当代码执行，绝对不能拼接不可信数据

## 三、CSRF 跨站请求伪造
CSRF 全称跨站请求伪造，它的攻击逻辑是：攻击者诱导受害者进入第三方网站，然后在这个第三方网站上，向被攻击网站发送跨站请求。

攻击的核心是**利用受害者在被攻击网站已有的登录凭证**，绕过后台的用户验证，冒充用户执行操作。

### 典型的CSRF攻击流程
1. 受害者登录`a.com`，并且保留了登录凭证（比如Cookie）
2. 攻击者引诱受害者访问自己搭建的`b.com`
3. `b.com`向`a.com`发送一个请求（比如`a.com/act=xx`），浏览器会自动携带`a.com`的Cookie
4. `a.com`接收到请求后，验证Cookie有效，误以为是受害者自己发起的请求
5. `a.com`以受害者的名义执行`act=xx`操作
6. 攻击完成，受害者在不知情的情况下，被冒充执行了攻击者指定的操作

### CSRF的常见攻击方式
CSRF的攻击载体有很多种，常见的有这几种：
1. **GET请求触发**：比如在页面里嵌入一张图片，浏览器加载图片时会自动发送请求
2. **POST请求触发**：构造一个自动提交的表单，用户访问页面就会触发
    ```html
    <form action="http://bank.example/withdraw" method=POST>
        <input type="hidden" name="account" value="xiaoming" />
        <input type="hidden" name="amount" value="10000" />
        <input type="hidden" name="for" value="hacker" />
    </form>
    <script> document.forms[0].submit(); </script>
    ```
3. **超链接触发**：需要用户主动点击链接才会生效
    ```html
    <a href="http://test.com/csrf/withdraw.php?amount=1000&for=hacker" target="_blank">
        重磅消息！！
    </a>
    ```

### CSRF的特点
1. 攻击一般发起在第三方网站，被攻击的网站没办法直接防止攻击发生
2. 攻击是**冒用**用户的登录凭证执行操作，而不是直接窃取用户数据
3. 攻击者全程拿不到受害者的登录凭证，只是借用来用
4. 跨站请求的载体很多样（图片URL、超链接、Form提交等），部分可以嵌入论坛、文章，很难追踪

### CSRF的预防方案
被攻击的网站没法阻止攻击从第三方发起，只能增强自身的防护能力。常见的防护方案有两类：

**第一类：阻止不明外域的访问**
- 同源检测：检查请求的Referer或者Origin头信息，只接受同域名的请求
- Samesite Cookie：设置Cookie的Samesite属性，限制Cookie不能被跨站请求携带

**第二类：提交时要求附加本域才能获取的信息**
- CSRF Token：最常用的方案
- 双重Cookie验证

这里重点说下**CSRF Token**的实现流程：
1. 用户打开页面时，服务器为该用户生成一个唯一的Token
2. 对于GET请求，Token附在请求地址后面；对于POST请求，在form表单里加一个隐藏字段
    ```html
    <input type="hidden" name="csrftoken" value="tokenvalue"/>
    ```
3. 用户提交请求时，携带这个Token
4. 服务器接收请求后，验证Token的有效性，只有Token正确才会处理请求

## 四、SQL注入攻击
SQL注入攻击，就是攻击者把恶意的SQL查询或语句，插入到应用的输入参数中，然后让后台的SQL服务器解析执行，从而实现攻击目的。

### SQL注入的攻击流程
1. 找出Web应用中存在SQL漏洞的注入点
2. 判断目标数据库的类型和版本
3. 猜解数据库中的用户名、密码等敏感信息
4. 利用工具查找Web后台的管理入口
5. 入侵系统并进行破坏操作

### SQL注入的预防方案
1. 严格检查输入变量的类型和格式，比如数字类型的参数强制转换成数字
2. 过滤和转义输入中的特殊字符，比如单引号、双引号、分号等
3. 给访问数据库的Web应用程序配置Web应用防火墙（WAF），拦截恶意请求
4. 尽量使用**预编译语句**和**参数化查询**，这是预防SQL注入最有效的手段

## 最后说两句
上面只是列举了最常见的几种Web攻击方式，实际开发中遇到的安全问题远比这多。对于Web安全，千万不能抱有侥幸心理，每一个细节都不能忽视。


**【往期精彩】**
- [原生 DOM 真的慢吗？为什么现代框架都迷恋虚拟 DOM？](https://mp.weixin.qq.com/s/hY97PUXy_WQeagpaYtzk9g)
- [Vue进阶系列第9篇--你真的懂nextTick吗？我表示很怀疑](https://mp.weixin.qq.com/s/XATTpbsFwd6vuw5EqTUCfQ)
- [JavaScript ES6中的生成器(Generator)是什么？有哪些应用场景？一文说透](https://mp.weixin.qq.com/s/PtFtlNUH-UcLYU5h-xKiQw)
- [JavaScript ES6中的生成器(Decorator)是什么？有哪些应用场景？](https://mp.weixin.qq.com/s/c7IwODBilcxSyozJvjjHRw)

- [一文读懂：什么时候该用防抖，什么时候该用节流？
](https://mp.weixin.qq.com/s/oDbnX3AKVICC5HUpYtxzIA)
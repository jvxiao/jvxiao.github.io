---
title: 糟糕，Vite proxy error ECONNREFUSED
date: 2025-05-23 12:40:00
category: Web开发
tags:
  - Vite
  - Proxy
  - error
--- 

最近正在开发一款应用，前端框架是vite+vue, 后台服务也是使用的node+express组合，当一切工作都在顺利进行的时候，不出意外，意外出现了。

当我尝试使用axios直连本地服务地址的时候，丝滑得如同德芙一样。然而每次请求地址都需要拼写全链接--即host+port+service-path，这是一件很愚蠢的事情。

为了让事情变得简单一些，减少重复工作，我和聪明的各位一样决定使用vite的代理。

因为web的地址9000，本地启动的后台服务地址是8080，所以我最初的配置如下：

``` javascript
export default defineConfig({
  # ...   其它配置
  server: {
    host: '0.0.0.0',
    port: 9000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080/',
        changeOrigin: true,
      }
    }
  }
})

```

然后它却对我说 `Vite proxy error ECONNREFUSED`

那么多大风大浪都过去了，居然在这小阴沟里翻了船，忍不了！

## 排查方向
因为涉及到vite server配置，其本质就是http-server以及http-agent的问题,  于是从从下面几个方向做了一一排查

- **Vite的服务器配置**

- **端口占用或防火墙设置**

- **浏览器或系统代理设置**

- ​**Hosts文件配置问题或DNS解析问题**


## 测试验证
- 首先vite服务配置问题是最先被排除的，因为该项目配置已经经历了时间的考验，在很多项目中都有使用

- 然后通过 `netstat -ano | find 9000` 检查并未发现端口被占用的情况

- 至于代理，确实是开了--科学上网使用。然而在关闭之后，似乎没有起作用，所以问题不在这里。

- 那么还剩最后一条，当我检查我本地的hosts文件时，纳尼，localhost没有配置指向本地的环回地址127.0.0.1，难道是这里出问题了吗。

  为了验证这个想法，我将配置中的target做了以下修改:

  ~~target: 'http://localhost:8080/'~~ </br>
    target: 'http://127.0.0.1:8080/'

  果然，生效了，看来之前的猜测是对的。

</br>

所以目前至少有两种方法可以解决该问题：

  - **在vite配置中将localhost替换成127.0.0.1**
  - **在hosts文件中添加localhost的地址**

当然，第二种方式显然是更合理的，几乎是种一劳永逸的方法，在日后不同项目的开发中，总是难免还会存在使用localhost的情况。因此，建议在hosts文件中配置它。

## 总结
在这个问题中呢，其实也牵扯出更多的关于http server 和 http client相关的知识。不论是server的创建还是client的连接，正确的ip和端口是需要保证的。

当遇到类似的情况时，不妨从底层思考在复杂的封装工具和配置的背后，它的本质是什么，以及正常使用时需要哪些因子的参与。
title: Nginx 使用流程
author: Funny Boy
date: 2023-07-14 14:06:26
tags: ["Nginx"]
categories: ["服务器"]

---

# 前言

Windows 的 Nginx、Tomcat 的使用流程

---

# 一、Nginx

## 1.下载

下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-1.png)

## 2.使用

使用 Vue 项目的 npm run build:prod 命令打包生成 dist 文件夹，将 dist 文件夹下的所有文件移动到 nginx 的 html 文件夹下

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-2.png)
在 nginx 文件夹下运行 cmd 控制台

```powershell
nginx	// 启动nginx服务
nginx -s reload	// 重新加载修改后的html文件或nginx配置文件（不需要重新启动nginx服务）
nginx -s quit	// 关闭nginx服务
```

nginx 服务的默认端口号是 80，conf 文件夹下的 nginx.conf 文件可以修改端口号

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-3.png)
如果 Vue 项目的路由模式是 history 模式，那么刷新会出现 404 错误，因此需要 Nginx 进行反向代理

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-4.png)

## 3.部署多个项目

一个计算机可以同时存在多个 Nginx 服务，只要保证端口号不同即可。如果需要在一个端口号下（https 的 443 端口号）部署多个项目，则通过子目录的方式实现

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-5.png)
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-6.png)
Vue 项目需要修改 vue.config.js（vue-cli 是 publicPath 属性，vite 是 base 属性）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-7.png)

Vue 项目还需要修改路由配置（vue-router3 是新增 base 属性，vue-router4 是使用 API 传参）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-8.png)

# 总结

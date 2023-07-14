title: Nginx使用流程
author: Funny Boy
date: 2023-07-14 14:06:26
tags: ["Nginx"]
categories: ["服务器"]
---
# 前言

Windows的Nginx、Tomcat的使用流程

---
# 一、Nginx
## 1.下载
下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-1.png)

## 2.使用
使用Vue项目的npm run build:prod命令打包生成dist文件夹，将dist文件夹下的所有文件移动到nginx的html文件夹下

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-2.png)
在nginx文件夹下运行cmd控制台

```powershell
nginx	// 启动nginx服务
nginx -s reload	// 重新加载修改后的html文件或nginx配置文件（不需要重新启动nginx服务）
nginx -s quit	// 关闭nginx服务
```

nginx服务的默认端口号是80，conf文件夹下的nginx.conf文件可以修改端口号

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-3.png)
如果Vue项目的路由模式是history模式，那么刷新会出现404错误，因此需要Nginx进行反向代理

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-4.png)
## 3.部署多个项目
一个计算机可以同时存在多个Nginx服务，只要保证端口号不同即可。如果需要在一个端口号下（https的443端口号）部署多个项目，则通过子目录的方式实现

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-5.png)
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-6.png)
Vue项目需要修改vue.config.js（vue-cli是publicPath属性，vite是base属性）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-7.png)


Vue项目还需要修改路由配置（vue-router3是新增base属性，vue-router4是使用API传参）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-8.png)

# 总结
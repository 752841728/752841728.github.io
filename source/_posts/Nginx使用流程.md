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

Nginx 服务的默认端口号是 80，conf 文件夹下的 nginx.conf 文件可以修改端口号

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-3.png)

如果 Vue 项目的路由模式是 history 模式，那么刷新会出现 404 错误，因此需要 Nginx 进行反向代理

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-4.png)

## 3.部署多个项目

一个计算机可以同时存在多个 Nginx 服务，只要保证端口号不同即可。如果需要在一个端口号下（https 的 443 端口号）部署多个项目，则通过子目录的方式实现

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-5.png)
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-6.png)
Vue 项目需要修改 vue.config.js（vue-cli 是 publicPath 属性，vite 是 base 属性）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-8.png)

Vue 项目还需要修改路由配置（vue-router3 是新增 base 属性，vue-router4 是使用 API 传参）

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-7.png)

# 二、Tomcat

## 1.下载

Tomcat 下载地址：[https://tomcat.apache.org/download-80.cgi](https://tomcat.apache.org/download-80.cgil)
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-9.png)
Java 下载地址：[https://www.oracle.com/java/technologies/downloads](https://www.oracle.com/java/technologies/downloads)
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-10.png)

## 2.配置环境变量

新增系统变量，对应 Tomcat 和 Java 安装路径

```powershell
CATALINA_BASE	E:\tomcat\apache-tomcat-8.5.91
CATALINA_HOME	E:\tomcat\apache-tomcat-8.5.91
CATALINA_TMPDIR	E:\tomcat\apache-tomcat-8.5.91\temp
JAVA_HOME	E:\Java\jdk1.8.0_6
CLASSPATH	.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar
```

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-11.png)
编辑 Path 环境变量

```powershell
E:\tomcat\apache-tomcat-8.5.91\bin
%JAVA_HOME%\bin
%JAVA_HOME%\jre\bin
```

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-12.png)

## 3.使用

使用 Vue 项目的 npm run build:prod 命令打包生成 dist 文件夹，将 dist 文件夹下的所有文件移动到 sttss_web 的 html 文件夹下（新建 sttss_web 文件夹，名字随意）
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-13.png)
如果 Vue 项目的路由模式是 history 模式，那么刷新会出现 404 错误，因此需要在 sttss_web 文件夹下添加 WEB_INF/web.xml 文件

```html
<?xml version="1.0" encoding="UTF-8"?>
<web-app
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee/web-app_2_5.xsd"
  id="scplatform"
  version="2.5"
>
  <display-name>/</display-name>
  <error-page>
    <error-code>404</error-code>
    <location>/index.html</location>
  </error-page>
</web-app>
```

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-14.png)
双击 bin/startup.bat 启动 Tomcat 服务即可

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-15.png)
Tomcat 服务的默认端口号是 8080，conf 文件夹下的 server.xml 文件可以修改端口号

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-16.png)
如果请求 Tomcat 的静态资源出现跨域问题，可以在 conf 文件夹下的 web.xml 文件添加配置

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/2-17.png)

# 总结

本文使用的是 Tomcat 8 和 Java 8，不同的版本的 Tomcat 和 Java 会有兼容性问题


title: Vue使用不支持的文件类型
author: Funny Boy
date: 2023-08-28 15:47:00
tags: ["Vue"]
categories: ["前端"]

---

# 前言
Vue使用不支持的文件类型时，报You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file.错误
示例中使用的数据：[Cesium_Air.gltf](https://github.com/752841728/hexo-picture/blob/main/map/Cesium_Air.gltf)

---
# 一、vue-cli
vue-cli4只支持node16版本，node16版本以上的版本需要修改package.json文件

```javascript
"scripts": {
  "serve": "set NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
},
```

## 1.安装依赖

```javascript
npm install url-loader
npm install file-loader
```
## 2.配置vue.config.js文件

```javascript
module: {
  rules: [
    {
      test: /\.(gltf)$/,
      use: {
        loader: "url-loader",
      },
    },
  ],
},
```

# 二、vite
## 1.配置vite.config.js文件

```javascript
assetsInclude: ['**/*.gltf'],
```

# 三、使用方法

```javascript
import gltf from "@/assets/Cesium_Air.gltf";

viewer.scene.primitives.add(
  Cesium.Model.fromGltf({
    url: gltf,
  })
);
```

如果觉得配置麻烦，可以直接将Cesium_Air.gltf文件放在public目录下，但会增加打包后包的大小

```javascript
viewer.scene.primitives.add(
  Cesium.Model.fromGltf({
    url: "./Cesium_Air.gltf",
  })
);
```

---

# 总结
感觉vue-cli4已经逐步进入淘汰阶段了，以后的文章不会再出现vue-cli4的内容了
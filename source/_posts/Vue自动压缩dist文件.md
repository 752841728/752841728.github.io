title: Vue自动压缩dist文件
author: Funny Boy
date: 2023-12-15 17:35:26
tags: ["Nginx"]
categories: ["服务器"]

---

# 前言
最近部署项目频繁，故想省去手动压缩dist文件的步骤

---
# 一、vue-cli4
## 1.安装插件

```powershell
npm install filemanager-webpack-plugin -D
```

## 2.配置vue.config.js

```javascript
const FileManagerPlugin = require("filemanager-webpack-plugin");
const packageName = "dist";
module.exports = defineConfig({
  outputDir: packageName,
  ......
    chainWebpack(config) {
    config
      .plugin("fileManager")
      .use(FileManagerPlugin)
      .tap((args) => [
        {
          events: {
            onEnd: {
              delete: [
                // 首先需要删除项目根目录下的dist.zip
                `./${packageName}.zip`,
              ],
              archive: [
                // 选择文件夹将之打包成xxx.zip并放在根目录
                {
                  source: `./${packageName}`,
                  destination: `./${packageName}.zip`,
                },
              ],
            },
          },
        },
      ]);
  },
});
```

# 二、vite
## 1.安装插件

```powershell
npm install rollup-plugin-compression -D
```

## 2.配置vite.config.js

```javascript
import compresssionBuild from "rollup-plugin-compression";
import type { ICompressionOptions } from "rollup-plugin-compression";
const option: ICompressionOptions = {
  sourceName: `dist`,
  type: "zip",
  targetName: `dist`,
};
export default defineConfig({
  build: {
    outDir: option.sourceName
  },
  ......
  plugins: [vue(), compresssionBuild(option)]
});
```

---

# 总结
最好使用Jenkins自动化部署
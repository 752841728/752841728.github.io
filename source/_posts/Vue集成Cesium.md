title: Vue集成Cesium
author: Funny Boy
date: 2023-06-20 16:31:00
tags: ["Vue","Cesium"]
categories: ["GIS"]
---
# 前言

cesium是非常强大的地图工具，在vue集成cesium的过程中，或多或少会遇到问题，这里稍做整理

---
# 一、cesium使用示例

```html
<template>
  <div>
    <div id="map" style="width: 100vw; height: 100vh"></div>
  </div>
</template>

<script>
import * as Cesium from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";

export default {
  name: "cesium",
  data() {
    return {
      viewer: null,
    };
  },
  methods: {
    initMap() {
      Cesium.Ion.defaultAccessToken =
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI2ZTgxOGNhZC03NThmLTQ0NzMtOTNlYS1kNmM3YzlmZDU3NTMiLCJpZCI6MTI1NTY3LCJpYXQiOjE2NzY5MDI0MzJ9.ulIEz3NCh2cMEBiHrqIuv9I6icn5KTMMnBdy2wassoM";
      this.viewer = new Cesium.Viewer("map", {
        // 配置项
      });
    },
  },
  mounted() {
    this.initMap();
  },
};
</script>
```

# 二、vue-cli4集成cesium
## 1.创建项目

```powershell
vue create vue-cli-4-cesium
```
## 2.安装依赖
因为cesium@1.106.1使用了ES11的语法，所以需要安装babel-loader
```powershell
npm install cesium --save-dev
npm install @open-wc/webpack-import-meta-loader -D
npm install @babel/core @babel/preset-env babel-loader --save-dev
```
## 3.配置vue.config.js

```javascript
const path = require("path");
const webpack = require("webpack");
const CopyWebpackPlugin = require("copy-webpack-plugin");

function resolve(dir) {
  return path.join(__dirname, dir);
}

module.exports = {
  configureWebpack: {
    name: "vue-cesium",
    resolve: {
      alias: {
        "@": resolve("src"),
      },
    },
    plugins: [
      new CopyWebpackPlugin([
        {
          from: "node_modules/cesium/Build/Cesium/Workers",
          to: "cesium/Workers",
        },
        {
          from: "node_modules/cesium/Build/Cesium/ThirdParty",
          to: "cesium/ThirdParty",
        },
        {
          from: "node_modules/cesium/Build/Cesium/Assets",
          to: "cesium/Assets",
        },
        {
          from: "node_modules/cesium/Build/Cesium/Widgets",
          to: "cesium/Widgets",
        },
      ]),
      new webpack.DefinePlugin({
        CESIUM_BASE_URL: JSON.stringify("./cesium"),
      }),
    ],
    module: {
      rules: [
        {
          test: /.js$/,
          include: [resolve("node_modules/cesium/Source")],
          use: {
            loader: require.resolve("@open-wc/webpack-import-meta-loader"),
          },
        },
        {
          test: /.js$/,
          include: [resolve("node_modules/@cesium/engine/Source")],
          use: "babel-loader",
        },
      ],
      unknownContextCritical: false,
    },
  },
};
```

# 二、vue-cli5集成cesium

## 1.创建项目

```powershell
vue create vue-cli-5-cesium
```
## 2.安装依赖

```powershell
npm install cesium --save-dev
npm install copy-webpack-plugin node-polyfill-webpack-plugin @open-wc/webpack-import-meta-loader -D
```
## 3.配置vue.config.js

```javascript
const { defineConfig } = require("@vue/cli-service");

const path = require("path");
const webpack = require("webpack");
const CopyWebpackPlugin = require("copy-webpack-plugin");
const NodePolyfillPlugin = require("node-polyfill-webpack-plugin");

function resolve(dir) {
  return path.join(__dirname, dir);
}

module.exports = defineConfig({
  transpileDependencies: true,
  lintOnSave: false,
  configureWebpack: {
    name: "vue-cesium",
    resolve: {
      alias: {
        "@": resolve("src"),
      },
    },
    plugins: [
      new NodePolyfillPlugin(),
      new CopyWebpackPlugin({
        patterns: [
          {
            from: "node_modules/cesium/Build/Cesium/Workers",
            to: "cesium/Workers",
          },
          {
            from: "node_modules/cesium/Build/Cesium/ThirdParty",
            to: "cesium/ThirdParty",
          },
          {
            from: "node_modules/cesium/Build/Cesium/Assets",
            to: "cesium/Assets",
          },
          {
            from: "node_modules/cesium/Build/Cesium/Widgets",
            to: "cesium/Widgets",
          },
        ],
      }),
      new webpack.DefinePlugin({
        CESIUM_BASE_URL: JSON.stringify("./cesium"),
      }),
    ],
    module: {
      rules: [
        {
          test: /.js$/,
          include: /(cesium)/,
          use: {
            loader: "@open-wc/webpack-import-meta-loader",
          },
        },
      ],
    },
  },
});
```
# 三、vite集成cesium

## 1.创建项目

```powershell
npm create vite@latest
```
## 2.安装依赖

```powershell
npm install cesium --save-dev
npm install cesium vite-plugin-cesium vite -D
```

## 3.配置vite.config.js

```javascript
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import cesium from 'vite-plugin-cesium';

export default defineConfig({
  plugins: [vue(), cesium()],
});
```

# 总结
因为vite使用了vite-plugin-cesium插件，所以vite集成cesium变得非常简单。其实vue-cli也有vue-cli-plugin-cesium插件，可惜已经很久没有维护，高版本的vue-cli已经无法使用。
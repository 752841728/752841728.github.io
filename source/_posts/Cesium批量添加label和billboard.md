
title: Cesium批量添加label和billboard
author: Funny Boy
date: 2023-08-28 11:06:02
tags: ["Cesium"]
categories: ["GIS"]
---

# 前言
使用primitive批量添加label和billboard
示例中使用的数据：[pump.json](https://github.com/752841728/hexo-picture/blob/main/map/pump.json)、[pump_icon.png](https://github.com/752841728/hexo-picture/blob/main/map/pump_icon.png)

---

# 一、配置

```javascript
<template>
  <div>
    <div id="map" style="width: 100vw; height: 100vh"></div>
  </div>
</template>

<script>
import * as Cesium from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";
import pumpData from "@/assets/pump.json";
import pump_icon from "@/assets/pump_icon.jpg";
import { tranformCoordinate } from "@/utils/map.js";    // 转换坐标系的方法

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
        terrainProvider: Cesium.createWorldTerrain(), // 生成地形
        requestRenderMode: true,  // 优化性能
        maximumRenderTimeChange: Infinity,  // 优化性能
      });
      viewer.scene.globe.depthTestAgainstTerrain = true;  // 开启深度检测
      viewer.scene.postProcessStages.fxaa.enabled = true;  // 抗锯齿
    },
  },
  mounted() {
    this.initMap();
    this.add_primitive_icon(pumpData.features, pump_icon, true);
  },
};
</script>
```

# 二、label和billboard

```javascript
const add_primitive_icon = (data, image, withLabel) => {
  let icons = viewer.scene.primitives.add(new Cesium.BillboardCollection());
  let labels;
  if (withLabel) {
    labels = viewer.scene.primitives.add(new Cesium.LabelCollection());
  }
  for (let index = 0; index <= data.length - 1; index++) {
    let coordinates = data[index].geometry.coordinates;
    let text = data[index].properties.CHINESE_NAME;
    let name = data[index].properties.NAME;
    let position = Cesium.Cartesian3.fromDegrees(
      ...tranformCoordinate(coordinates[0], coordinates[1]),
      0
    );
    icons.add({
      position: position,
      image: image,
      disableDepthTestDistance: Number.POSITIVE_INFINITY,   // 避免深度检测被地形遮挡
    });
    if (withLabel) {
      labels.add({
        position: position,
        text: name,
        scale: 0.5, // 放大后缩小，提高清晰度
        fillColor: Cesium.Color.WHITE,
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 12.0,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        font: "40px Source Han Sans CN",
        pixelOffset: new Cesium.Cartesian2(0, 50),  // 偏移量
        disableDepthTestDistance: Number.POSITIVE_INFINITY, // 避免深度检测被地形遮挡
      });
    }
  }
};
```

# 三、效果图
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/8-1.png)

---
# 总结
转换坐标系的tranformCoordinate方法，参考右文：[不同坐标系的经纬度转换](https://752841728.github.io/2023/08/24/%E4%B8%8D%E5%90%8C%E5%9D%90%E6%A0%87%E7%B3%BB%E7%9A%84%E7%BB%8F%E7%BA%AC%E5%BA%A6%E8%BD%AC%E6%8D%A2/)

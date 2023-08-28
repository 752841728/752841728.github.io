title: Cesium添加三维管线
author: Funny Boy
date: 2023-08-24 14:58:17
tags: ["Cesium"]
categories: ["GIS"]

---

# 前言

使用primitive添加三维管线，并自定义材质和颜色
示例中使用的数据：[map-3d.json](https://github.com/752841728/hexo-picture/blob/main/map/map-3d.json)、[surface.jpg](https://github.com/752841728/hexo-picture/blob/main/map/surface.jpg)

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
import mapData from "@/assets/map-3d.json";
import surface_img from "@/assets/surface.jpg";
import { tranformCoordinate } from "@/utils/map.js";	// 转换坐标系的方法

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
    },
  },
  mounted() {
    this.initMap();
  },
};
</script>
```

# 二、三维管线

```javascript
// 添加三维管线
const add3DLine = (withOutline) => {
  const volume = []; // 管线
  const volume_outline = []; // 管线轮廓

  // 管线的横截面，radius半径，side_number几边形
  function computeCircle(radius, side_number) {
    const positions = [];
    for (let i = 0; i < side_number; i++) {
      const radians = Cesium.Math.toRadians(i * (360 / side_number)); // 360边形就是一个圆
      positions.push(
        new Cesium.Cartesian2(
          radius * Math.cos(radians),
          radius * Math.sin(radians)
        )
      );
    }
    return positions;
  }

  mapData.features.forEach((feature, index) => {
    const positions = feature.geometry.coordinates[0];
    volume.push(
      new Cesium.GeometryInstance({
        geometry: new Cesium.PolylineVolumeGeometry({
          polylinePositions: Cesium.Cartesian3.fromDegreesArrayHeights(
            handleData(positions, 5)
          ),
          shapePositions: computeCircle(0.5, 6),
        }),
      })
    );
    // 管线轮廓
    if (withOutline) {
      volume_outline.push(
        new Cesium.GeometryInstance({
          geometry: new Cesium.PolylineVolumeOutlineGeometry({
            polylinePositions: Cesium.Cartesian3.fromDegreesArrayHeights(
              handleData(positions, 5)
            ),
            shapePositions: computeCircle(0.5, 6),
          }),
          attributes: {
            color: Cesium.ColorGeometryInstanceAttribute.fromColor(
              new Cesium.Color(255 / 255, 255 / 255, 255 / 255, 1)
            ),
          },
        })
      );
    }
  });

  viewer.scene.primitives.add(
    new Cesium.Primitive({
      geometryInstances: volume,
      releaseGeometryInstances: false,
      appearance: new Cesium.EllipsoidSurfaceAppearance({
        material: new Cesium.Material({
          fabric: {
            uniforms: {
              image: surface_img,
              color: new Cesium.Color.fromBytes(255, 0, 0, 1),
            },
            source: `
                    uniform vec4 color;

                    czm_material czm_getMaterial(czm_materialInput materialInput)
                    {
                        czm_material material = czm_getDefaultMaterial(materialInput);
                        vec2 st = materialInput.st;
                        float s = st.s;
                        float t = st.t;
                        vec4 colorImage = texture(image, vec2(fract(s), t));
                        material.diffuse = colorImage.rgb * color.rgb;
                        return material;
                    }
                    `,
          },
        }),
      }),
    })
  );
  // 管线轮廓
  if (withOutline) {
    viewer.scene.primitives.add(
      new Cesium.Primitive({
        geometryInstances: volume_outline,
        appearance: new Cesium.PerInstanceColorAppearance({
          flat: true,
          renderState: {
            lineWidth: 1.0,
          },
        }),
      })
    );
  }

  function handleData(data, height) {
    let arr = [];
    data.forEach((item, index) => {
      arr.push(...tranformCoordinate(item[0], item[1]));
      if (height) arr.push(height);
    });
    return arr;
  }
};
```

# 三、效果图
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/5-1.png)

---
# 总结
转换坐标系的tranformCoordinate方法，参考右文：[不同坐标系的经纬度转换](https://752841728.github.io/2023/08/24/%E4%B8%8D%E5%90%8C%E5%9D%90%E6%A0%87%E7%B3%BB%E7%9A%84%E7%BB%8F%E7%BA%AC%E5%BA%A6%E8%BD%AC%E6%8D%A2/)
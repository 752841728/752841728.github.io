title: Cesium添加流动箭头
author: Funny Boy
date: 2023-12-12 12:19:17
tags: ["Cesium"]
categories: ["GIS"]

---

# 前言

添加三维管线的方法和数据，参考右文：[Cesium添加三维管线](https://752841728.github.io/2023/08/24/Cesium%E6%B7%BB%E5%8A%A0%E4%B8%89%E7%BB%B4%E7%AE%A1%E7%BA%BF)
本文是在add3DLine函数的基础上增加withAnimation传参实现添加流动箭头（使用WebGL1.0的语法）
示例中使用的数据：[arrow.png](https://github.com/752841728/hexo-picture/blob/main/map/arrow.png)

---
# 一、根据图片大小分配箭头数量
参考资料：[cesium polyline 自定义材质图片运动线](https://blog.csdn.net/qq_34447899/article/details/123224443)
缺陷：通过计算线的角度解决图片破碎问题，导致部分情况箭头方向相反

```javascript
import arrow_img from "@/assets/arrow.png";

function initMap() {
  Cesium.Ion.defaultAccessToken =
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI2ZTgxOGNhZC03NThmLTQ0NzMtOTNlYS1kNmM3YzlmZDU3NTMiLCJpZCI6MTI1NTY3LCJpYXQiOjE2NzY5MDI0MzJ9.ulIEz3NCh2cMEBiHrqIuv9I6icn5KTMMnBdy2wassoM";
  viewer = new Cesium.Viewer("map", {
    // 配置项
    terrainProvider: Cesium.createWorldTerrain(), // 生成地形
    // requestRenderMode: true, // 优化性能
    // maximumRenderTimeChange: Infinity, // 优化性能
    contextOptions: {
      requestWebgl1: true,  // 使用WebGL1.0的语法
    },
  });
  viewer.scene.globe.depthTestAgainstTerrain = true; // 开启深度检测
}

// 添加三维管线
function add3DLine(withOutline, withAnimation) {
  const volume = []; // 管线
  const volume_outline = []; // 管线轮廓
  const volume_animation = []; // 流动箭头

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
    // 流动箭头
    if (withAnimation) {
      volume_animation.push(
        new Cesium.GeometryInstance({
          geometry: new Cesium.PolylineGeometry({
            positions: Cesium.Cartesian3.fromDegreesArrayHeights(
              handleData(positions, 5.5)
            ),
            width: 10,
          }),
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
  // 流动箭头
  if (withAnimation) {
    viewer.scene.primitives.add(
      new Cesium.Primitive({
        asynchronous: false,
        geometryInstances: volume_animation,
        appearance: new Cesium.PolylineMaterialAppearance({
          material: new Cesium.Material({
            fabric: {
              uniforms: {
                image: arrow_img,
                imageW: 20,
                alpha: 1.0,
              },
              source: `
                varying float v_polylineAngle;
                uniform float alpha;

                mat2 rotate(float rad) {
                  float c = cos(rad);
                  float s = sin(rad);
                  return mat2(c, s, -s, c);
                }
                czm_material czm_getMaterial(czm_materialInput materialInput)
                {
                  czm_material material = czm_getDefaultMaterial(materialInput);
                  vec2 st = materialInput.st;
                  vec2 pos = rotate(v_polylineAngle) * gl_FragCoord.xy;
                  float s = pos.x / (imageW * czm_pixelRatio);
                  s = s - czm_frameNumber / 60.0;
                  float t = st.t;
                  vec4 colorImage = texture2D(image, vec2(fract(s), t));
                  float a = colorImage.a;
                  material.diffuse = colorImage.rgb;
                  material.alpha = a * alpha;
                  return material;
                }
				`,
            },
          }),
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
}
```
---

# 总结
暂时没有完美实现流动箭头的方案
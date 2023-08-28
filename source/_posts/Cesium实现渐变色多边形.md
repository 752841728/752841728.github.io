title: Cesium实现渐变色多边形
author: Funny Boy
date: 2023-07-14 17:21:02
tags: ["Cesium"]
categories: ["GIS"]

---

# 前言

本文添加多边形的方法和数据，参考右文：[Cesium添加点、线、面](https://752841728.github.io/2023/06/30/Cesium%E6%B7%BB%E5%8A%A0%E7%82%B9%E3%80%81%E7%BA%BF%E3%80%81%E9%9D%A2/)

---
# 一、canvas实现渐变色

```html
<canvas id="mapBgCanvas" style="position: fixed"></canvas>
```

```javascript
// 渐变色画布
initCanvas() {
  var canvas = document.getElementById("mapBgCanvas");
  if (canvas && canvas.getContext) {
    let ctx = canvas.getContext("2d");
    let grad = ctx.createLinearGradient(150, 0, 150, 150); // 创建一个渐变色线性对象
    // grad.addColorStop(0, "rgba(28, 146, 210, 1)"); // 设置渐变颜色
    // grad.addColorStop(1, "rgba(76, 161, 175, 1)");
    grad.addColorStop(0, "rgba(44, 74, 157, 1)"); // 设置渐变颜色
    grad.addColorStop(1, "rgba(33, 184, 255, 1)");
    ctx.fillStyle = grad; // 设置fillStyle为当前的渐变对象
    ctx.fillRect(0, 0, 300, 300); // 绘制渐变图形
  }
  // entity
  this.gradientColor = new Cesium.ImageMaterialProperty({
    image: document.getElementById("mapBgCanvas"),
  });
  // primitive
  // this.gradientColor = new Cesium.Material({
  //   fabric: {
  //     type: "Image",
  //     uniforms: {
  //       image: document.getElementById("mapBgCanvas"),
  //     },
  //   },
  // });
},
```

# 二、entity实现渐变色多边形

```javascript
// entity添加面
add_entity_polygon() {
  this.entity_polygon = this.viewer.entities.add({
    polygon: {
      hierarchy: Cesium.Cartesian3.fromDegreesArray(
        handleData(mapData.polygon_vertices)
      ),
      material: this.gradientColor,
      height: 1000, // 一定要设置高度，否则outline不生效
      outline: true,
      outlineColor: new Cesium.Color(255 / 255, 255 / 255, 255 / 255, 1),
      outlineWidth: 2,
    },
    name: "polygon",
  });

  function handleData(data) {
    let arr = [];
    data.forEach((item) => {
      arr.push(Number(item.x), Number(item.y));
    });
    return arr;
  }
},
```
# 三、primitive实现渐变色多边形

```javascript
// primitive添加面
add_primitive_polygon(
  color = new Cesium.Color(0 / 255, 185 / 255, 255 / 255, 1)
) {
  // 多边形
  let polygonGeometry = new Cesium.GeometryInstance({
    geometry: new Cesium.PolygonGeometry({
      height: 1000, // 高度在这里设置
      polygonHierarchy: new Cesium.PolygonHierarchy(
        Cesium.Cartesian3.fromDegreesArray(
          handleData(mapData.polygon_vertices)
        )
      ),
    }),
    id: {
      name: "polygon",
    },
  });
  this.primitive_polygon = new Cesium.Primitive({
    asynchronous: false, // 通过删除再添加实现变色，此过程不会出现卡顿
    geometryInstances: polygonGeometry,
    appearance: new Cesium.EllipsoidSurfaceAppearance({
      material: this.gradientColor,
    }),
  });
  this.viewer.scene.primitives.add(this.primitive_polygon);

  // 多边形边框
  var polygonOutlineGeometry = new Cesium.GeometryInstance({
    geometry: new Cesium.PolylineGeometry({
      positions: Cesium.Cartesian3.fromDegreesArrayHeights(
        handleData(mapData.polygon_vertices, 1000)
      ),
      width: 2,
    }),
    id: {
      name: "polygon",
    },
  });
  this.primitive_polygonOutline = new Cesium.Primitive({
    asynchronous: false,
    geometryInstances: polygonOutlineGeometry,
    appearance: new Cesium.PolylineMaterialAppearance({
      material: Cesium.Material.fromType("Color", {
        color: new Cesium.Color(255 / 255, 255 / 255, 255 / 255, 1),
      }),
    }),
  });
  this.viewer.scene.primitives.add(this.primitive_polygonOutline);

  function handleData(data, height) {
    let arr = [];
    data.forEach((item) => {
      arr.push(Number(item.x), Number(item.y));
      if (height) arr.push(height);
    });
    return arr;
  }
},
```
# 四、效果图
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/4-1.png)

# 总结
[Cesium添加点、线、面](https://752841728.github.io/2023/06/30/Cesium%E6%B7%BB%E5%8A%A0%E7%82%B9%E3%80%81%E7%BA%BF%E3%80%81%E9%9D%A2/)中添加多边形的方法和本文中添加渐变色的方法，区别是将material改为this.gradientColor即可
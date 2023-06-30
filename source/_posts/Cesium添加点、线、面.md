title: Cesium添加点、线、面
author: Funny Boy
date: 2023-06-30 12:08:17
tags: ["Vue","Cesium"]
categories: ["GIS"]

---

# 前言

使用 entity 和 primitive 两种方式添加点、线、面，并通过点击事件修改颜色
示例使用的数据：[map.js](https://gitee.com/funny_boy/vue-cesium/blob/master/vue-cli-5-cesium/src/assets/data/map.js)

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
import mapData from "@/assets/map.js";

export default {
  name: "cesium",
  data() {
    return {
      viewer: null,
      entity_point_list: {},
      entity_line_list: {},
      entity_polygon: null,
      primitive_point_list: {},
      primitive_line_list: {},
      primitive_polygon: null,
      primitive_polygonOutline: null,
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
    // 定位
    setView(x, y, z) {
      if (x instanceof Array || y instanceof Array) {
        // 如果x和y是数组，根据x和y计算出定位的矩形范围
        this.viewer.camera.setView({
          destination: Cesium.Rectangle.fromDegrees(...handleData(x, y)),
          duration: 0,
          complete: () => {},
        });
      } else {
        this.viewer.camera.setView({
          destination: Cesium.Cartesian3.fromDegrees(x, y, z),
          duration: 0,
          complete: () => {},
        });
      }

      function handleData(x_arr, y_arr) {
        var xmax = 0;
        var xmin = 180;
        var ymax = 0;
        var ymin = 90;
        x_arr.forEach((x) => {
          xmax = Math.max(xmax, x);
          xmin = Math.min(xmin, x);
        });
        y_arr.forEach((y) => {
          ymax = Math.max(ymax, y);
          ymin = Math.min(ymin, y);
        });
        var shift_x = (xmax - xmin) / 20;
        var shift_y = (ymax - ymin) / 20;
        return [xmin - shift_x, ymin - shift_y, xmax + shift_x, ymax + shift_y];
      }
    },
  },
  mounted() {
    this.initMap();
    this.setView(mapData.junction_x, mapData.junction_y);
  },
};
</script>
```

# 二、entity

## 1.点

```javascript
// entity加载点
add_entity_point() {
  mapData.junction_id.forEach((item, index) => {
    this.entity_point_list[item] = this.viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(
        mapData.junction_x[index],
        mapData.junction_y[index]
      ),
      point: {
        pixelSize: 5,
        color: new Cesium.Color(20 / 255, 204 / 255, 137 / 255, 1),
      },
      name: "point",
    });
  });
},
```

## 2.线

```javascript
// entity添加线
add_entity_line() {
  mapData.pipe_id.forEach((item, index) => {
    this.entity_line_list[item] = this.viewer.entities.add({
      polyline: {
        positions: Cesium.Cartesian3.fromDegreesArray(
          handleData(mapData.pipe_vertices[index])
        ),
        width: 2,
        material: new Cesium.Color(0 / 255, 185 / 255, 255 / 255, 1),
      },
      name: "line",
    });
  });

  function handleData(data) {
    let arr = [];
    data.forEach((item) => {
      arr.push(item[0], item[1]);
    });
    return arr;
  }
},
```

## 3.面

```javascript
// entity添加面
add_entity_polygon() {
  this.entity_polygon = this.viewer.entities.add({
    polygon: {
      hierarchy: Cesium.Cartesian3.fromDegreesArray(
        handleData(mapData.polygon_vertices)
      ),
      material: new Cesium.Color(0 / 255, 185 / 255, 255 / 255, 1),
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

## 4.点击变色

```javascript
// 添加点击事件
addHandler() {
  const handler = new Cesium.ScreenSpaceEventHandler(
    this.viewer.scene.canvas
  );

  handler.setInputAction((movement) => {
    let pick = this.viewer.scene.pick(movement.position);
    if (!pick) return;
    console.log("pick:", pick);
    // entity
    if (pick.id.name == "point") {
      pick.id.point.color = new Cesium.Color(
        255 / 255,
        0 / 255,
        0 / 255,
        1
      );
    }
    if (pick.id.name == "line") {
      pick.id.polyline.material.color = new Cesium.Color(
        255 / 255,
        0 / 255,
        0 / 255,
        1
      );
    }
    if (pick.id.name == "polygon") {
      pick.id.polygon.material.color = new Cesium.Color(
        255 / 255,
        0 / 255,
        0 / 255,
        1
      );
    }
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
},
```

# 三、primitive

## 1.点

```javascript
// primitive添加点
add_primitive_point() {
  const data = mapData;
  const pointGeometry = [];
  const length = data.junction_id.length;
  for (let index = 0; index <= length - 1; index++) {
    pointGeometry.push(
      new Cesium.GeometryInstance({
        geometry: new Cesium.CircleGeometry({
          center: Cesium.Cartesian3.fromDegrees(
            data.junction_x[index],
            data.junction_y[index]
          ),
          radius: 5,
        }),
        id: { id: data.junction_id[index], name: "point" },
        attributes: {
          color: new Cesium.ColorGeometryInstanceAttribute(
            20 / 255,
            204 / 255,
            137 / 255,
            1
          ),
        },
      })
    );
  }
  this.primitive_point_list = new Cesium.Primitive({
    geometryInstances: pointGeometry,
    appearance: new Cesium.PerInstanceColorAppearance({
      flat: true,
    }),
  });

  this.viewer.scene.primitives.add(this.primitive_point_list);
},
```

## 2.线

```javascript
// primitive添加线
add_primitive_line() {
  const data = mapData;
  const lineGeometry = [];
  const length = data.pipe_id.length;
  for (let index = 0; index <= length - 1; index++) {
    lineGeometry.push(
      new Cesium.GeometryInstance({
        geometry: new Cesium.PolylineGeometry({
          positions: Cesium.Cartesian3.fromDegreesArray(
            handleData(data.pipe_vertices[index])
          ),
          width: 2,
        }),
        id: { id: data.pipe_id[index], name: "line" },
        attributes: {
          color: Cesium.ColorGeometryInstanceAttribute.fromColor(
            new Cesium.Color(0 / 255, 185 / 255, 255 / 255, 1)
          ),
        },
      })
    );
  }
  this.primitive_line_list = new Cesium.Primitive({
    geometryInstances: lineGeometry,
    appearance: new Cesium.PolylineColorAppearance(),
  });

  this.viewer.scene.primitives.add(this.primitive_line_list);

  function handleData(data) {
    let arr = [];
    data.forEach((item) => {
      arr.push(item[0], item[1]);
    });
    return arr;
  }
},
```

## 3.面

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
    asynchronous: false, // 如果通过删除再添加实现变色，添加此属性不会出现卡顿
    geometryInstances: polygonGeometry,
    appearance: new Cesium.EllipsoidSurfaceAppearance({
      material: Cesium.Material.fromType("Color", {
        color: color,
      }),
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

## 4.点击变色

```javascript
// 添加点击事件
addHandler() {
  const handler = new Cesium.ScreenSpaceEventHandler(
    this.viewer.scene.canvas
  );

  handler.setInputAction((movement) => {
    let pick = this.viewer.scene.pick(movement.position);
    if (!pick) return;
    console.log("pick:", pick);
    // primitive
    if (pick.id.name == "point") {
      const attributes = pick.primitive.getGeometryInstanceAttributes(
        pick.id
      );
      attributes.color = Cesium.ColorGeometryInstanceAttribute.toValue(
        new Cesium.Color(255 / 255, 0 / 255, 0 / 255, 1)
      );
    }
    if (pick.id.name == "line") {
      const attributes = pick.primitive.getGeometryInstanceAttributes(
        pick.id
      );
      attributes.color = Cesium.ColorGeometryInstanceAttribute.toValue(
        new Cesium.Color(255 / 255, 0 / 255, 0 / 255, 1)
      );
    }
    if (pick.id.name == "polygon") {
      pick.primitive.appearance.material.uniforms.color = new Cesium.Color(
        255 / 255,
        0 / 255,
        0 / 255,
        1
      );
    }
  }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
},
```

# 四、完整代码

[add_point_line_polygon.vue](https://gitee.com/funny_boy/vue-cesium/blob/master/vue-cli-5-cesium/src/components/add_point_line_polygon.vue)

# 五、效果图

![add_point_line_polygon.png](http://rx1tfk0h0.hn-bkt.clouddn.com/images/add_point_line_polygon.png)

---

# 总结

因为 entity 是基于 primitive 封装的，所以 primitive 性能比 entity 好太多了，但是对于 cesium 和 webgl 的熟练度较高。数据量小的话，使用 entity 可以快速上手。数据量大的话，推荐使用 primitive 提高性能。

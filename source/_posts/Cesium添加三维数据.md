title: Cesium添加三维数据
author: Funny Boy
date: 2023-07-14 16:24:03
tags: ["Cesium"]
categories: ["GIS"]
---
# 前言
Cesium添加gltf、3dtiles、digital terrain model、超图等三维数据

---
# 一、gltf

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
    //动态动画
    shouldAnimate: true,
})
let position = Cesium.Cartesian3.fromDegrees(114.162813, 22.279327, 100);
// 设置模型方向
let hpRoll = new Cesium.HeadingPitchRoll(Cesium.Math.toRadians(0), Cesium.Math.toRadians(0), Cesium.Math.toRadians(0));//弧度
let orientation = Cesium.Transforms.headingPitchRollQuaternion(position, hpRoll);
let model = viewer.entities.add({
    // 模型id
    id: 'model',
    // 模型位置
    position: position,
    // 模型方向
    orientation: orientation,
    // 模型资源
    model: {
        // 模型路径
        uri: '../gltf/Cesium_Air.gltf',
    }
})

viewer.zoomTo(model)

var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
handler.setInputAction(function (click) {
    var pickedObject = viewer.scene.pick(click.position);
    if (Cesium.defined(pickedObject)) {
        pickedObject.id.model.color = Cesium.Color.RED  // 改变颜色
        pickedObject.id.model.silhouetteColor = Cesium.Color.BLUE   // 轮廓颜色
        pickedObject.id.model.silhouetteSize = 5   // 轮廓宽度
    }
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-1.png)

# 二、3dtiles

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
  // 生成地形
  terrainProvider: Cesium.createWorldTerrain()
})
// 加载倾斜示范数据
var palaceTileset = new Cesium.Cesium3DTileset({
  url: './3dtiles/tile_32_17/tileset.json',
})
palaceTileset.readyPromise.then(function (palaceTileset) {
  viewer.scene.primitives.add(palaceTileset);
  // 控制模型的位置
  var heightOffset = 20.0; // 可以改变3Dtiles的高度
  var boundingSphere = palaceTileset.boundingSphere;
  var cartographic = Cesium.Cartographic.fromCartesian(boundingSphere.center);
  var surface = Cesium.Cartesian3.fromRadians(cartographic.longitude, cartographic.latitude, 0.0);
  var offset = Cesium.Cartesian3.fromRadians(cartographic.longitude, cartographic.latitude, heightOffset);
  var translation = Cesium.Cartesian3.subtract(offset, surface, new Cesium.Cartesian3());
  palaceTileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation);
  // 定位到三维模型
  viewer.zoomTo(palaceTileset);
});

var temp;

var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
handler.setInputAction(function (movement) {
  // 在这里处理单击事件
  var pickedObject = viewer.scene.pick(movement.position);
  if (temp) temp.detail.model.color = undefined;
  pickedObject.detail.model.color = Cesium.Color.RED;  // 改变颜色
  temp = pickedObject
}, Cesium.ScreenSpaceEventType.LEFT_CLICK);
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-2.png)

# 三、digital terrain model

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
    terrainProvider: new Cesium.CesiumTerrainProvider({
        url: "./DigitalTerrainModelDTM_GEOTIFF"
    })
})
viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(113.824174404144, 22.1379446983337, 1000)
});
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-3.png)

# 四、SuperMap
[SuperMap的安装包](http://support.supermap.com.cn/DownloadCenter/DownloadPage.aspx?id=2336)

[SuperMap的Demo](http://support.supermap.com.cn/DataWarehouse/WebDocHelp/iServer/webgl/examples/webgl/examples.html#layer)

[SuperMap的教程](https://www.bilibili.com/video/BV1Dh41137x5/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=7db51e154f5eebccdf418a5264852559)

---

# 总结
推荐一个gltf转glb、ogsb转3dtiles的工具：[3dtiles](https://github.com/fanvanzh/3dtiles)
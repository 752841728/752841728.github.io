# 前言
Cesium添加gltf、3dtiles、geotiff、超图等三维数据

---
# 一、gltf
## 1.entity

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
    //动态动画
    shouldAnimate: true,
})
// 设置模型位置
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
        uri: './gltf/Cesium_Air.gltf',
    }
})

viewer.zoomTo(model)
```
## 2.primitive
```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
    //动态动画
    shouldAnimate: true,
})
// 设置模型位置
let position = Cesium.Cartesian3.fromDegrees(114.162813, 22.279327, 100);
// 设置模型方向
let hpRoll = new Cesium.HeadingPitchRoll(Cesium.Math.toRadians(0), Cesium.Math.toRadians(0), Cesium.Math.toRadians(0));//弧度
let modelMatrix = Cesium.Transforms.headingPitchRollToFixedFrame(
    position,
    hpRoll,
    Cesium.Ellipsoid.WGS84,
    Cesium.Transforms.eastNorthUpToFixedFrame,
    new Cesium.Matrix4()
);
let model = viewer.scene.primitives.add(
    Cesium.Model.fromGltf({
        url: "./gltf/Cesium_Air.gltf",
        modelMatrix: modelMatrix,
        scale: 1,
        color: new Cesium.Color(1, 0, 0, 1),
    })
);

viewer.camera.setView({
    destination: position,
    duration: 0,
    complete: () => {},
});
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-1.png)

# 二、3dtiles

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
  // 生成地形
  terrainProvider: Cesium.createWorldTerrain(),
});

// 加载倾斜示范数据
var palaceTileset = new Cesium.Cesium3DTileset({
  url: "./3dtiles/W20229-MWBRWPS-M-B-000/W20229-MWBRWPS-M-B-000/tileset.json",
});
palaceTileset.readyPromise.then(function (palaceTileset) {
  viewer.scene.primitives.add(palaceTileset);
  // 控制模型的位置
  update3dtilesMaxtrix(palaceTileset, {
    tx: 114.1314395, // 模型中心X轴坐标（经度，单位：十进制度）
    ty: 22.5404925, // 模型中心Y轴坐标（纬度，单位：十进制度）
    tz: 0, // 模型中心Z轴坐标（高程，单位：米）
    rx: 0, // X轴（经度）方向旋转角度（单位：度）
    ry: 0, // Y轴（纬度）方向旋转角度（单位：度）
    rz: 182.5, // Z轴（高程）方向旋转角度（单位：度）
    scale: 1, // 缩放比例
  });
  // 定位到三维模型
  viewer.zoomTo(palaceTileset);
});
```
## 1.控制位置

```javascript
// 控制位置
function update3dtilesMaxtrix(tileset, position_params) {
  var cartographic = Cesium.Cartographic.fromCartesian(
    tileset.boundingSphere.center
  );
  var surface = Cesium.Cartesian3.fromDegrees(
    Cesium.Math.toDegrees(cartographic.longitude),
    Cesium.Math.toDegrees(cartographic.latitude),
    cartographic.height
  );
  var offset = Cesium.Cartesian3.fromDegrees(
    position_params.tx,
    position_params.ty,
    position_params.tz
  );
  var translation = Cesium.Cartesian3.subtract(
    offset,
    surface,
    new Cesium.Cartesian3()
  );
  // 赋值给tileset
  tileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation);
}
```

## 2.控制位置、角度、缩放
```javascript
// 控制位置、角度、缩放
function update3dtilesMaxtrix(tileset, position_params) {
  // 旋转
  var mx = Cesium.Matrix3.fromRotationX(
    Cesium.Math.toRadians(position_params.rx)
  );
  var my = Cesium.Matrix3.fromRotationY(
    Cesium.Math.toRadians(position_params.ry)
  );
  var mz = Cesium.Matrix3.fromRotationZ(
    Cesium.Math.toRadians(position_params.rz)
  );
  var rotationX = Cesium.Matrix4.fromRotationTranslation(mx);
  var rotationY = Cesium.Matrix4.fromRotationTranslation(my);
  var rotationZ = Cesium.Matrix4.fromRotationTranslation(mz);
  // 缩放
  var scale = Cesium.Matrix4.fromUniformScale(position_params.scale);
  // 平移
  var position = Cesium.Cartesian3.fromDegrees(
    position_params.tx,
    position_params.ty,
    position_params.tz
  );
  var m = Cesium.Transforms.eastNorthUpToFixedFrame(position);
  // 旋转、平移、缩放矩阵相乘
  Cesium.Matrix4.multiply(m, scale, m);
  Cesium.Matrix4.multiply(m, rotationX, m);
  Cesium.Matrix4.multiply(m, rotationY, m);
  Cesium.Matrix4.multiply(m, rotationZ, m);
  // 赋值给tileset
  tileset._root.transform = m;
}
```

![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-2.png)

# 三、geotiff

```javascript
var viewer = new Cesium.Viewer("cesiumcontainer", {
    terrainProvider: new Cesium.CesiumTerrainProvider({
        url: "./DigitalTerrainModelDTM_GEOTIFF"
    })
})
viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(113.824174404144, 22.1379446983337, 1000),
    duration: 0,
    complete: () => {},
});
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/3-3.png)

# 四、SuperMap

[SuperMap的安装包](http://support.supermap.com.cn/DownloadCenter/DownloadPage.aspx?id=2336)

[SuperMap的Demo](http://support.supermap.com.cn/DataWarehouse/WebDocHelp/iServer/webgl/examples/webgl/examples.html#layer)

[SuperMap的文档](http://support.supermap.com.cn/DataWarehouse/WebDocHelp/iServer/webgl/web/apis/3dwebgl.html)

[SuperMap的教程](https://www.bilibili.com/video/BV1Dh41137x5/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=7db51e154f5eebccdf418a5264852559)

[SuperMap技术问答社区](http://qa.supermap.com/)

---

# 总结
推荐一个glb转gltf、ogsb转3dtiles的工具包：[3dtiles](https://github.com/fanvanzh/3dtiles)
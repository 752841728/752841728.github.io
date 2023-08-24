title: Cesium自定义底图
author: Funny Boy
date: 2023-08-24 16:43:02
tags: ["Vue","Cesium"]
categories: ["GIS"]
---

# 前言
近期在做大屏的项目，需要蓝色系的地图，让我头疼了很久，这里提供一些方案

---
# 一、配置

```javascript
// Cesium的token
Cesium.Ion.defaultAccessToken =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI2ZTgxOGNhZC03NThmLTQ0NzMtOTNlYS1kNmM3YzlmZDU3NTMiLCJpZCI6MTI1NTY3LCJpYXQiOjE2NzY5MDI0MzJ9.ulIEz3NCh2cMEBiHrqIuv9I6icn5KTMMnBdy2wassoM";
// SuperMap的token
Cesium.Credential.CREDENTIAL = new Cesium.Credential(
  "ILPgjURUGKYLUyyTvQpjzZKkmmJaH0P1YwvShK4xShMthIA3JbY68vvfyxqVitXLDV7l24XaWzG35SzVnCwNWg.."
);

viewer = new Cesium.Viewer("map", {
  infoBox: false,
  animation: false, // 禁用动画控件
  timeline: false, // 禁用时间线控件
  homeButton: false, // 禁用Home按钮
  sceneModePicker: false, // 禁用场景模式选择器
  baseLayerPicker: false, // 禁用图层选择器
  navigationHelpButton: false, // 禁用帮助按钮
  fullscreenButton: false, // 禁用全屏按钮
  geocoder: false, // 禁用搜索框
  requestRenderMode: true, // 显示渲染
  maximumRenderTimeChange: 1.0,
  navigation: false,
});
```

# 二、Cesium底图
## 1、设置透明度
terrainProvider也会变得透明，用于展示地下管线
```javascript
// cesium方法
viewer.scene.screenSpaceCameraController.enableCollisionDetection = false;
viewer.scene.globe.translucency.enabled = true;
viewer.scene.globe.translucency.frontFaceAlpha = 0.8;
// supermap方法
viewer.scene.terrainProvider.isCreateSkirt = false;
viewer.scene.undergroundMode = true;
viewer.scene.globe.globeAlpha = 0.8;
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-6.png)

# 三、天地图

## 1、使用方法

```javascript
let imageryLayers = viewer.imageryLayers;
// 移出Cesium底图
imageryLayers.removeAll();
// 添加天地图
let imagery = imageryLayers.addImageryProvider(
  new Cesium.WebMapTileServiceImageryProvider({
    url: "http://t0.tianditu.gov.cn/vec_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=vec&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk=f69df935f2b4e5d06629d91c56be2809",
    format: "image/png",
    layer: "",
    style: "",
    tileMatrixSetID: "",
  })
);
imageryLayers.addImageryProvider(
  new Cesium.WebMapTileServiceImageryProvider({
    url: "http://t0.tianditu.gov.cn/cia_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=cia&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk=f69df935f2b4e5d06629d91c56be2809",
    format: "image/png",
    layer: "",
    style: "",
    tileMatrixSetID: "",
  })
);
```


## 2、自定义颜色
通过Cesium的API调整，缺点：只有黑色效果好，其他颜色效果差

```javascript
imagery.hue = 3;
imagery.contrast = -1.5;
imagery.alpha = 0.2;
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-1.png)


通过Cesium的API调整，球体设置颜色，天地图设置透明度，缺点：清晰度差

```javascript
viewer.scene.globe.baseColor = new Cesium.Color(0, 40 / 255, 102 / 255);
imagery.contrast = -1.5;
imagery.alpha = 0.2;
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-2.png)


通过修改imageryLayers的着色器代码，缺点：imageryLayers的所有图层都会变色

```javascript
function imageryTheme(viewer, color) {
  // 设置两个变量，用来判断是否进行颜色的翻转和过滤
  const options = {
    invertColor: true,
    filterRGB: color,
  };
  // 更改地图着色器代码
  const baseFragShader =
    viewer.scene.globe._surfaceShaderSet.baseFragmentShaderSource.sources;
  for (let i = 0; i < baseFragShader.length; i++) {
    const strS = "color = czm_saturation(color, textureSaturation);\n#endif\n";
    let strT = "color = czm_saturation(color, textureSaturation);\n#endif\n";
    if (options.invertColor) {
      strT += `
      color.r = 1.0 - color.r;
      color.g = 1.0 - color.g;
      color.b = 1.0 - color.b;
      `;
    }
    if (options.filterRGB.length > 0) {
      strT += `
      color.r = color.r * ${options.filterRGB[0]}.0/255.0;
      color.g = color.g * ${options.filterRGB[1]}.0/255.0;
      color.b = color.b * ${options.filterRGB[2]}.0/255.0;
      `;
    }
    baseFragShader[i] = baseFragShader[i].replace(strS, strT);
  }
}
imageryTheme(viewer, [0, 107, 207]);
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-3.png)

# 四、智图

```javascript
// 添加geoq地图
imageryLayers.addImageryProvider(
  new Cesium.UrlTemplateImageryProvider({
    url: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer/tile/{z}/{y}/{x}",
  })
);
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-4.png)


# 五、Mapbox

```javascript
// 添加mapbox地图
imageryLayers.addImageryProvider(
  new Cesium.MapboxStyleImageryProvider({
    url: "https://api.mapbox.com/styles/v1",
    username: "752841728",
    styleId: "cllgeh81w000601pehdb5e02r",
    accessToken:
      "pk.eyJ1IjoiNzUyODQxNzI4IiwiYSI6ImNsbGdiNnJoNTB6ZjIzcW8xMjVudjJtOGUifQ.u6q8oswvLpgvjdp3nW76jg",
    scaleFactor: true,
  })
);
```
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/7-5.png)


---

# 总结
天地图改不出理想的效果，智图不清楚是否可以商用，最后选择了Mapbox
[Mapbox注册教程](https://baijiahao.baidu.com/s?id=1769460780111679075&wfr=spider&for=pc)
[Mapbox官方样式](https://www.mapbox.com/gallery/)
还有一些尝试失败的文章
[Cesium 高性能扩展之自定义地图主题](https://zhuanlan.zhihu.com/p/480835644)
[天地图修改主题颜色修改背景色](https://segmentfault.com/a/1190000041703873)
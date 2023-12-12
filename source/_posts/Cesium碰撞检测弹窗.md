title: Cesium碰撞检测弹窗
author: Funny Boy
date: 2023-12-12 10:19:17
tags: ["Cesium"]
categories: ["GIS"]

---

# 前言
弹窗在不互相遮挡的情况下，随着cesium相机高度升高而减少，随着cesium相机高度降低而增加

---
# 一、代码

```javascript
<template>
  <div>
    <div id="map" style="width: 100vw; height: 100vh"></div>
    <div
      v-for="(item, index) in popupData"
      :key="index"
      class="popup"
      :style="{
        left: item.x + 'px',
        top: item.y + 'px',
      }"
    ></div>
  </div>
</template>

<script>
import * as Cesium from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";

let viewer;

export default {
  name: "cesium",
  data() {
    return {
      popupData: [],
    };
  },
  methods: {
    initMap() {
      let that = this;

      Cesium.Ion.defaultAccessToken =
        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI2ZTgxOGNhZC03NThmLTQ0NzMtOTNlYS1kNmM3YzlmZDU3NTMiLCJpZCI6MTI1NTY3LCJpYXQiOjE2NzY5MDI0MzJ9.ulIEz3NCh2cMEBiHrqIuv9I6icn5KTMMnBdy2wassoM";
      viewer = new Cesium.Viewer("map");

      viewer.scene.postRender.addEventListener((e) => {
        that.addPopup();
      });
    },
    setView(x, y, z = 100000) {
      if (x instanceof Array || y instanceof Array) {
        viewer.camera.setView({
          destination: Cesium.Rectangle.fromDegrees(
            ...handleRectangleDegrees(x, y)
          ),
          duration: 0,
          complete: () => {},
        });
      } else {
        viewer.camera.setView({
          destination: Cesium.Cartesian3.fromDegrees(x, y, z),
          duration: 0,
          complete: () => {},
        });
      }
    },
    addPopup() {
      let arr = [
        {
          show: true,
          postiton: [113.516554832785, 22.7815541566398],
        },
        {
          show: true,
          postiton: [113.565595110563, 22.8119042280145],
        },
        {
          show: true,
          postiton: [113.585566261118, 22.7467959068676],
        },
        {
          show: true,
          postiton: [113.585561647606, 22.7467922756939],
        },
        {
          show: true,
          postiton: [113.585544380734, 22.7468031264434],
        },
        {
          show: true,
          postiton: [113.5250091611, 22.796468624824],
        },
        {
          show: true,
          postiton: [113.53501380961, 22.7752535345139],
        },
        {
          show: true,
          postiton: [113.606360650159, 22.7673058506878],
        },
        {
          show: true,
          postiton: [113.575970393566, 22.8011478781795],
        },
        {
          show: true,
          postiton: [113.48319856831, 22.8434974846255],
        },
        {
          show: true,
          postiton: [113.483183010224, 22.8434759099068],
        },
        {
          show: true,
          postiton: [113.521532982952, 22.7202557655554],
        },
        {
          show: true,
          postiton: [113.53274534813, 22.7955110445971],
        },
        {
          show: true,
          postiton: [113.584525474457, 22.7676399422363],
        },
        {
          show: true,
          postiton: [113.54157885687, 22.8101993251604],
        },
        {
          show: true,
          postiton: [113.540344472305, 22.7983192314786],
        },
        {
          show: true,
          postiton: [113.536264743559, 22.7850303729224],
        },
        {
          show: true,
          postiton: [113.5897553783, 22.7504601423855],
        },
        {
          show: true,
          postiton: [113.513549642677, 22.8011357132934],
        },
        {
          show: true,
          postiton: [113.665260156322, 22.6558535329566],
        },
        {
          show: true,
          postiton: [113.665242814055, 22.6540658381607],
        },
        {
          show: true,
          postiton: [113.540721493798, 22.8003300912433],
        },
        {
          show: true,
          postiton: [113.526640076431, 22.6870157770143],
        },
        {
          show: true,
          postiton: [113.494998793222, 22.8129609109105],
        },
        {
          show: true,
          postiton: [113.593364200102, 22.743924187381],
        },
        {
          show: true,
          postiton: [113.540681930867, 22.8078782923931],
        },
        {
          show: true,
          postiton: [113.550853128856, 22.7614672586024],
        },
        {
          show: true,
          postiton: [113.517482495539, 22.7849363232933],
        },
        {
          show: true,
          postiton: [113.662701458716, 22.6466855872565],
        },
        {
          show: true,
          postiton: [113.594501573329, 22.7505207744991],
        },
        {
          show: true,
          postiton: [113.597951948579, 22.7769371796389],
        },
        {
          show: true,
          postiton: [113.550305763681, 22.7620551543094],
        },
        {
          show: true,
          postiton: [113.568432803916, 22.7563407873987],
        },
        {
          show: true,
          postiton: [113.525835234783, 22.7976826914486],
        },
        {
          show: true,
          postiton: [113.536859093935, 22.8207334899539],
        },
        {
          show: true,
          postiton: [113.533484184956, 22.8106796700382],
        },
        {
          show: true,
          postiton: [113.561286394469, 22.6576939900548],
        },
        {
          show: true,
          postiton: [113.517806458323, 22.7815962791158],
        },
        {
          show: true,
          postiton: [113.516595454663, 22.6914001747332],
        },
        {
          show: true,
          postiton: [113.542308151213, 22.7973896718941],
        },
        {
          show: true,
          postiton: [113.603754064104, 22.7708103375522],
        },
        {
          show: true,
          postiton: [113.513533350975, 22.8012006043278],
        },
        {
          show: true,
          postiton: [113.514669619202, 22.7985178250092],
        },
        {
          show: true,
          postiton: [113.518584605796, 22.8266911342335],
        },
        {
          show: true,
          postiton: [113.514671591971, 22.7985149161823],
        },
        {
          show: true,
          postiton: [113.514672523459, 22.7985136999894],
        },
        {
          show: true,
          postiton: [113.514673302842, 22.7985119821778],
        },
        {
          show: true,
          postiton: [113.525831173412, 22.797677088093],
        },
        {
          show: true,
          postiton: [113.525822803781, 22.7976688379218],
        },
        {
          show: true,
          postiton: [113.525828709136, 22.7976751872076],
        },
        {
          show: true,
          postiton: [113.525826808323, 22.7976734135123],
        },
        {
          show: true,
          postiton: [113.52583358882, 22.7976798836908],
        },
        {
          show: true,
          postiton: [113.566106350839, 22.801166199545],
        },
        {
          show: true,
          postiton: [113.566111262961, 22.801242665269],
        },
        {
          show: true,
          postiton: [113.543064161582, 22.783263992993],
        },
        {
          show: true,
          postiton: [113.598001030574, 22.7770088898555],
        },
        {
          show: true,
          postiton: [113.54068188311, 22.8078570901731],
        },
        {
          show: true,
          postiton: [113.540569108785, 22.7957774034675],
        },
        {
          show: true,
          postiton: [113.517826225587, 22.7816572777368],
        },
        {
          show: true,
          postiton: [113.536864075158, 22.8207324208513],
        },
        {
          show: true,
          postiton: [113.536869022012, 22.8207302807108],
        },
        {
          show: true,
          postiton: [113.533496690806, 22.810679262368],
        },
        {
          show: true,
          postiton: [113.540554780068, 22.7958167856627],
        },
        {
          show: true,
          postiton: [113.54905183958, 22.8055537524493],
        },
        {
          show: true,
          postiton: [113.56644295678, 22.800544710883],
        },
        {
          show: true,
          postiton: [113.544432042623, 22.8030296675129],
        },
        {
          show: true,
          postiton: [113.495158864878, 22.797248092325],
        },
        {
          show: true,
          postiton: [113.516579222339, 22.6914439526023],
        },
        {
          show: true,
          postiton: [113.508136024199, 22.8503121640053],
        },
        {
          show: true,
          postiton: [113.533490103578, 22.8106792430909],
        },
        {
          show: true,
          postiton: [113.533503339525, 22.8106780998189],
        },
        {
          show: true,
          postiton: [113.533507664509, 22.8106782867513],
        },
        {
          show: true,
          postiton: [113.535007260258, 22.7752425504507],
        },
        {
          show: true,
          postiton: [113.575976679637, 22.8011399549163],
        },
        {
          show: true,
          postiton: [113.566459714017, 22.8005430389587],
        },
        {
          show: true,
          postiton: [113.525965071626, 22.8307287822828],
        },
        {
          show: true,
          postiton: [113.513273902813, 22.7829777203857],
        },
        {
          show: true,
          postiton: [113.532746424803, 22.7955034671885],
        },
      ];
      this.popupData = [];
      for (let i = 0; i <= arr.length - 1; i++) {
        let position = Cesium.Cartesian3.fromDegrees(...arr[i].postiton, 0);
        let box1 = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
          viewer.scene,
          position
        );
        for (let j = i + 1; j <= arr.length - 1; j++) {
          let position = Cesium.Cartesian3.fromDegrees(...arr[j].postiton, 0);
          let box2 = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
            viewer.scene,
            position
          );
          let flag = this.collision(box1, box2);
          if (flag) {
            arr[j].show = false;
          }
        }
      }
      arr.forEach((item) => {
        if (item.show) {
          let position = Cesium.Cartesian3.fromDegrees(...item.postiton, 0);
          let box = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
            viewer.scene,
            position
          );
          this.popupData.push(box);
        }
      });
    },
    collision(box1, box2) {
      let width = 75;
      let height = 75;
      let left = Math.max(box1.x, box2.x);
      let top = Math.max(box1.y, box2.y);
      let right = Math.min(box1.x + width, box2.x + width);
      let bottom = Math.min(box1.y + height, box2.y + height);
      if (left < right && top < bottom) {
        return true;
      } else {
        return false;
      }
    },
  },
  mounted() {
    this.initMap();
    this.setView(113.44, 22.73); // 南沙区中心点
    this.addPopup();
  },
};
</script>

<style>
.popup {
  position: absolute;
  background-color: #002967;
  border: 0.05rem solid #fff;
  width: 50px;
  height: 50px;
  border-radius: 0.5rem;
  padding: 1rem;
}
</style>

```

---

# 效果图
![在这里插入图片描述](https://raw.githubusercontent.com/752841728/hexo-picture/main/img/10-1.gif)
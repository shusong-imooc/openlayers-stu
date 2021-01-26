<template>
  <div id="map" ref="rootmap" class="map"></div>
</template>
<script>
import "ol/ol.css";
import { get as getProjection } from "ol/proj";
// import { transform } from "ol/proj";
import { getWidth, getTopLeft } from "ol/extent";
import View from "ol/View";
import Map from "ol/Map";
import TileLayer from "ol/layer/Tile";
import WMTS from "ol/source/WMTS";
import WMTSTileGrid from "ol/tilegrid/WMTS";
import VectorSource from "ol/source/Vector";
import VectorLayer from "ol/layer/Vector";
import Feature from "ol/Feature";
// import GeoJSON from "ol/format/GeoJSON";
import Point from "ol/geom/Point";
import LineString from "ol/geom/LineString";
// import Polygon from "ol/geom/Polygon";
// import { fromCircle } from "ol/geom/Polygon";
import Circle from "ol/geom/Circle";
import olStyle from "ol/style/Style";
// import olCircle from "ol/style/Circle";
import olstyleIcon from "ol/style/Icon";
import olstyleText from "ol/style/Text";
import olstyleFill from "ol/style/Fill";
import olstyleStroke from "ol/style/Stroke";
import { defaults as defaultControls } from "ol/control";
import MousePosition from "ol/control/MousePosition";
import { createStringXY } from "ol/coordinate";

import icon from "@/assets/enterprise-waste-atmosphere-icon.png";
export default {
  data() {
    return {
      map1: null,
      view: null,
      // vectorSource: null,
      mousePositionControl: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      let projection = getProjection("EPSG:4326");
      let projectionExtent = projection.getExtent();
      let size = getWidth(projectionExtent) / 256;
      let resolutions = new Array(18);
      let matrixIds = new Array(18);
      for (let z = 1; z < 19; ++z) {
        resolutions[z] = size / Math.pow(2, z);
        matrixIds[z] = z;
      }
      let webKey = "49534f36b1e3ccba7baac6a27ebd2832";
      let wmtsUrl_1 = "http://t{0-7}.tianditu.gov.cn/img_c/wmts?tk="; //矢量底图
      let wmtsUrl_2 = "http://t{0-7}.tianditu.gov.cn/cva_c/wmts?tk="; //矢量注记
      this.view = new View({
        // center: [94.98882144722495, 43.6948398142437],
        center: [94.98882144722495, 43.6948398142437],
        projection: projection,
        // zoom: 8,
        zoom: 8,
        maxZoom: 21, // 最大缩放级别
        minZoom: 5, // 最小缩放级别
      });
      this.mousePositionControl = new MousePosition({
        coordinateFormat: createStringXY(4),
        projection: "EPSG:4326",
        className: "custom-mouse-position",
        target: document.getElementById("mouse-position"),
        undefinesHTML: "&nbsp;",
      });
      this.map1 = new Map({
        controls: defaultControls().extend([this.mousePositionControl]),
        layers: [
          new TileLayer({
            opacity: 0.7,
            source: new WMTS({
              url: wmtsUrl_1 + webKey,
              layer: "vec",
              matrixSet: "c",
              format: "tiles",
              style: "default",
              projection: projection,
              tileGrid: new WMTSTileGrid({
                origin: getTopLeft(projectionExtent),
                resolutions: resolutions,
                matrixIds: matrixIds,
              }),
              wrapX: true,
            }),
          }),
          new TileLayer({
            opacity: 0.7,
            source: new WMTS({
              url: wmtsUrl_2 + webKey,
              layer: "cva",
              matrixSet: "c",
              format: "tiles",
              style: "default",
              projection: projection,
              tileGrid: new WMTSTileGrid({
                origin: getTopLeft(projectionExtent),
                resolutions: resolutions,
                matrixIds: matrixIds,
              }),
              wrapX: true,
            }),
          }),
        ],
        target: "map",
        view: this.view,
      });
      let point = {
        latitude: "43.70632365345955",
        longitude: "94.97835762798788",
      };
      let data = { deadRadius: 1000 };
      this.showActSimDominoRadiusLayers(point, data);
    },

    showActSimDominoRadiusLayers(point, data) {
      let pointFeature, circleFeature, lineFeature;
      // 死亡
      if (data.deadRadius) {
        console.log(data.deadRadius);
        let color = "rgba(255, 0, 0, 0.6)";
        circleFeature = this.setCircleFeature(point, data.deadRadius, color);
        pointFeature = this.setPointFeature(point, color);
        lineFeature = this.setLineFeature(point, color);
      }
      let vectorLayer = new VectorLayer({
        zIndex: 10,
        visible: false,
        source: new VectorSource({
          features: [pointFeature, circleFeature, lineFeature],
        }),
      });
      vectorLayer.setVisible(true);
      this.map1.addLayer(vectorLayer);
      return vectorLayer;
    },
    setPointFeature(point, color) {
      // geom
      let centerPoint = new Point([point.longitude, point.latitude]);
      // feature
      let feature = new Feature({
        geometry: centerPoint,
        name: "测试点",
      });
      feature.setStyle(
        new olStyle({
          image: new olstyleIcon({
            src: icon,
            scale: 0.6,
            anchor: [0.5, 1],
          }),
          // image:new olCircle({
          //   radius:100,
          //   fill: new olstyleFill({
          //     color: color,
          //   }),
          // }),
          text: new olstyleText({
            text: "测试点",
            offsetY: -65,
            font: "14px sans-serif",
            //  标签的背景填充
            backgroundStroke: new olstyleStroke({
              color: "#23F4F9",
              width: 1,
            }),
            backgroundFill: new olstyleFill({
              color: "rgba(8, 62, 152, 0.4)",
            }),
            fill: new olstyleFill({
              color: color,
            }),
            padding: [10, 10, 7, 10],
          }),
        })
      );
      return feature;
    },
    setLineFeature(point, color) {
      let geomLine = new LineString([
        [point.longitude, point.latitude],
        [point.longitude, "43.80632365345955"],
      ]);
      let f = new Feature({
        geometry: geomLine,
        name: "测试点位",
      });
      f.setStyle(
        new olStyle({
          stroke: new olstyleStroke({
            color: color,
          }),
        })
      );
      return f;
    },
    setCircleFeature(point, radius, color) {
      let geomCircle = new Circle(
        [Number(point.longitude), Number(point.latitude)],
        // [point.longitude, point.latitude],
        // transform([point.longitude, point.latitude], 'EPSG:4326', 'EPSG:3857'),
        radius / 100000
      );
      let f = new Feature({
        geometry: geomCircle,
        name: "测试点位",
      });
      f.setStyle(
        new olStyle({
          stroke: new olstyleStroke({
            color: color,
          }),
          fill: new olstyleFill({
            color: "blue",
          }),
        })
      );
      return f;
    },
  },
};
</script>
<style scoped>
.map {
  position: relative;
  width: 100%;
  height: 100vh;
}
</style>
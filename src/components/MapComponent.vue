<template>
  <div class="map-container">
    <div id="map"></div>
    <div id="popup" class="ol-popup">
      <a-radio-group>
        <a-radio-button v-for="(item, index) in drawGroup" :value="item.value" :key="index" @click="handleDraw(item.value)">{{ item.name }}</a-radio-button>
      </a-radio-group>
    </div>
  </div>
</template>

<script>
import proj4 from 'proj4'

const ol = window.ol
export default {
  name: "MapComponent",
  mounted() {
    //初始化地图
    this.init_3395()
    //添加控件
    this.addControl()
    //添加绘制图层
    this.addVector()
  },
  data() {
    return {
      url: 'http://localhost:8090/iserver/services/map-kh_L2/rest/maps/China_Province3395@kh_L2',
      map: undefined,
      draw: undefined,
      vector: undefined,
      projection: undefined,
      drawGroup: [
        {
          name: '绘制点',
          value: 'Point'
        }, {
          name: '绘制线',
          value: 'LineString'
        }, {
          name: '绘制面',
          value: 'Polygon'
        }, {
          name: '清除',
          value: 'Clear'
        }
      ]
    }
  },
  methods: {
    init_3395() {
      const projection = this.definedProjection()
      this.projection = projection
      const resolutions = this.getResolutions(0, 0.000000028774, 0, 10);
      this.map = new ol.Map({
        controls: [new ol.control.ScaleLine()],
        target: 'map',
        view: new ol.View({
          center: [11609969.27, 3896311.62],
          zoom: 0,
          projection,
          resolutions
        })
      })
      window.map = this.map
      const layer = new ol.layer.Tile({
        source: new ol.source.TileSuperMapRest({
          url: this.url,
          //手动构建自定义目标的TileGrid
          tileGrid: new ol.tilegrid.TileGrid({
            extent: [8182092.31, 705312.23, 15037846.24, 7087311.0],
            resolutions: resolutions
          }),
          wrapX: true
        }),
        projection
      })
      this.map.addLayer(layer)
    },
    addControl() {
      const mousePositionControl = new ol.control.MousePosition({
        // 坐标格式  createStringXY(2)指定显示 2 个小数位
        coordinateFormat: ol.coordinate.createStringXY(2),
        // 投影
        projection: this.projection,
      });
      // 添加控件
      this.map.addControl(mousePositionControl);
      // 设置添加到哪个图层上
      //this.map.setTarget('map');

      //const scaleControl =
      //this.map.addControl(scaleControl)
    },
    addVector() {
      this.vector = new ol.layer.Vector({
        source: new ol.source.Vector({wrapX: false})
      });
      this.map.addLayer(this.vector)
    },
    /**
     * 自定义坐标系
     * @returns {Projection}
     */
    definedProjection() {
      proj4.defs("EPSG:3395", "+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +datum=WGS84 +units=m +no_defs");
      ol.proj.proj4.register(proj4);
      return new ol.proj.Projection({
        code: 'EPSG:3395',
        extent: [-20026376.39, -15496570.74, 20026376.39, 18764656.23],
        units: 'm'
      });
    },
    getResolutions(zoom, scale, targetMinZoom, targetMaxZoom) {
      const res = this.scaleToResolution(scale);
      const minRes = res * Math.pow(2, zoom - targetMinZoom);
      const resolutions = [];
      for (let index = 0; index < targetMaxZoom - targetMinZoom + 1; index++) {
        resolutions.push(minRes / Math.pow(2, index));
      }
      return resolutions;
    },
    scaleToResolution(scale) {
      const inchPerMeter = 1 / 0.0254;
      return 1 / (scale * 96 * inchPerMeter);
    },
    handleDraw(drawType) {
      this.clearInteraction()
      if (drawType === 'Clear') {
        this.vector.getSource().clear();
      } else {
        this.draw = new ol.interaction.Draw({
          source: this.vector.getSource(),
          type: drawType
        })
        this.draw.on('drawstart', (e) => {
          console.log({'绘制开始': e})
        })
        this.draw.on('drawend', (e) => {
          console.log({'绘制结束': e})
          console.log(new ol.format.GeoJSON({
            dataProjection: 'EPSG:4326',
            featureProjection: this.projection
          }).writeFeature(e.feature))

          this.clearInteraction()
        })
        this.map.addInteraction(this.draw);
      }
    },
    clearInteraction() {
      if (this.draw) {
        this.map.removeInteraction(this.draw)
      }
    },
  }
}
</script>

<style scoped>
.map-container, #map {
  width: 100%;
  height: 100%;
}

.ol-popup {
  position: absolute;
  top: 20px;
  right: 20px;
}

</style>
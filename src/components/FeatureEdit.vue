<template>
  <div id="map"></div>
</template>

<script>
const ol = window.ol
const SuperMap = window.SuperMap
export default {
  name: "FeatureEdit",
  mounted() {
    const map = new ol.Map({
      target: 'map',
      controls: ol.control.defaults({attributionOptions: {collapsed: false}})
          .extend([new ol.supermap.control.Logo()]),
      view: new ol.View({
        center: [ 110.69 , 20.08],
        zoom: 2,
        projection: 'EPSG:4326',
        multiWorld: true
      })
    });
    const layer = new ol.layer.Tile({
      source: new ol.source.TileSuperMapRest({
        url: this.mapUrl,
        wrapX: true
      }),
      projection: 'EPSG:4326'
    });
    map.addLayer(layer);
    map.addControl(new ol.supermap.control.ScaleLine());
    this.map = map
    window.map = map
    this.edit()
  },
  data () {
    return {
      mapUrl: 'http://localhost:8090/iserver/services/map-kh_L2/rest/maps/Airport_pt4326@kh_L2',
      dataUrl: 'http://localhost:8090/iserver/services/data-kh_L2/rest/data',
      map: undefined
    }
  },
  methods: {
    edit() {
      var sqlParam = new SuperMap.GetFeaturesBySQLParameters({
        queryParameter: {
          name: "Airport_pt4326@kh_L2",
          attributeFilter: "NAME = '海口美兰国际机场'"
        },
        datasetNames: ["kh_L2:Airport_pt4326"]
      });
      const featureService = new ol.supermap.FeatureService(this.dataUrl);
      featureService.getFeaturesBySQL(sqlParam, function (serviceResult) {
        console.log(serviceResult)
        var geoJson = serviceResult.result.features
        geoJson.features[0].geometry.coordinates=[121.33, 18.45]
        geoJson.features[0].properties.NAME = '海口美兰货运国际机场'
        console.log(geoJson)
        var newPointFeature = new ol.Feature(new ol.geom.Point([121.33, 18.45]))
        const properties = geoJson.features[0].properties
        properties.NAME = '海口美兰货运国际机场'
        newPointFeature.setProperties(properties)
        var addFeatureParams = new SuperMap.EditFeaturesParameters({
          features: newPointFeature, //new SuperMap.Format.GeoJSON().fromGeoJSON(geoJson)
          dataSourceName: "kh_L2",
          dataSetName: "Airport_pt4326",
          editType: SuperMap.EditType.UPDATE,
          returnContent: true
        });
        featureService.editFeatures(addFeatureParams, function (serviceResult) {
          // 获取返回数据
          var result = serviceResult.result;
          console.log(result)
        });
        var vectorSource = new ol.source.Vector({
          features: [newPointFeature],
          wrapX: false
        });
        const resultLayer = new ol.layer.Vector({
          source: vectorSource
        });
        window.map.addLayer(resultLayer);
      });
    }
  }
}
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
}
</style>
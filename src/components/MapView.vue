<template>
  <div class="map-container">
    <div id="map" ref="mapRef"></div>
    <div class="map-instructions">
      <p>点击地图选择物品位置</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, defineEmits, defineExpose } from 'vue';
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat, toLonLat } from 'ol/proj';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Style, Icon, Text, Fill, Stroke } from 'ol/style';
import Overlay from 'ol/Overlay';
import {defaults as defaultControls, ScaleLine, ZoomSlider, FullScreen, Attribution} from 'ol/control';

const emit = defineEmits(['map-click']);
const mapRef = ref(null);
let map = null;
let vectorSource = null;
let vectorLayer = null;

// 德国柏林市中心的经纬度坐标
const berlinCoordinates = [13.404954, 52.520008];

onMounted(() => {
  // 创建矢量图层和数据源，用于显示标记
  vectorSource = new VectorSource();
  vectorLayer = new VectorLayer({
    source: vectorSource,
    style: function(feature) {
      return new Style({
        image: new Icon({
          anchor: [0.5, 1],
          src: 'https://cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.5.0/examples/data/icon.png',
          scale: 0.5
        }),
        text: new Text({
          text: feature.get('title') || '',
          offsetY: -20,
          font: '12px Calibri,sans-serif',
          fill: new Fill({
            color: '#000'
          }),
          stroke: new Stroke({
            color: '#fff',
            width: 2
          })
        })
      });
    }
  });

  // 创建OpenLayers地图
  map = new Map({
    target: mapRef.value,
    layers: [
      new TileLayer({
        source: new OSM()
      }),
      vectorLayer
    ],
    view: new View({
      center: fromLonLat(berlinCoordinates),
      zoom: 13
    }),
    controls: defaultControls({
      attribution: false, // 隐藏默认的归因控件
      zoom: true,
      rotate: false
    }).extend([
      new ScaleLine({
        units: 'metric',
        bar: true,
        steps: 4,
        text: true,
        minWidth: 140
      }),
      new ZoomSlider(),
      new FullScreen(),
      new Attribution({
        collapsible: true,
        collapsed: true
      })
    ])
  });

  // 添加地图点击事件，用于获取点击位置
  map.on('click', function(evt) {
    const coordinate = evt.coordinate;
    const lonLat = toLonLat(coordinate);
    console.log('点击位置坐标: ', lonLat);
    
    // 发送坐标到父组件
    emit('map-click', lonLat);
  });
});

// 添加物品标记
const addItemMarker = (item) => {
  if (!map || !item.location) return;
  
  const coordinates = fromLonLat(item.location);
  
  // 创建一个新特征（标记）
  const feature = new Feature({
    geometry: new Point(coordinates),
    title: item.title,
    description: item.description,
    id: item.id
  });
  
  // 将标记添加到矢量图层
  vectorSource.addFeature(feature);
};

// 根据ID删除标记
const removeMarker = (itemId) => {
  if (!vectorSource) return;
  
  const features = vectorSource.getFeatures();
  const feature = features.find(f => f.get('id') === itemId);
  
  if (feature) {
    vectorSource.removeFeature(feature);
  }
};

// 获取当前地图视图中心坐标
const getCurrentCenter = () => {
  if (!map) return null;
  
  const view = map.getView();
  const center = view.getCenter();
  return toLonLat(center);
};

onUnmounted(() => {
  // 组件卸载时清理地图对象
  if (map) {
    map.setTarget(null);
    map = null;
  }
});

// 导出方法，供父组件调用
defineExpose({
  addItemMarker,
  removeMarker,
  getCurrentCenter
});
</script>

<style>
.map-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}

#map {
  width: 100%;
  height: 100%;
}

.map-instructions {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px 20px;
  border-radius: 20px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

.map-instructions p {
  margin: 0;
  font-size: 14px;
  color: #333;
}

/* OpenLayers控件样式 */
:deep(.ol-zoom) {
  top: 10px;
  left: 10px;
}

:deep(.ol-zoom-in), :deep(.ol-zoom-out) {
  background-color: rgba(255, 255, 255, 0.8) !important;
  color: #333 !important;
}

:deep(.ol-scale-line) {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 4px;
  bottom: 8px;
  left: 8px;
  padding: 2px;
}

:deep(.ol-full-screen) {
  top: 10px;
  right: 10px;
}

:deep(.ol-attribution) {
  bottom: 3px;
  right: 3px;
}

:deep(.ol-attribution button) {
  background-color: rgba(255, 255, 255, 0.8);
}

:deep(.ol-zoomslider) {
  top: 80px;
  left: 10px;
  height: 150px;
  background-color: rgba(255, 255, 255, 0.5);
}
</style> 
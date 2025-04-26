<template>
  <div class="map-container">
    <div id="map" ref="mapRef"></div>
    <div class="map-instructions">
      <p>Klicken Sie auf die Karte, um den Standort des Objekts auszuwählen</p>
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
import {defaults as defaultControls, ScaleLine, ZoomSlider, FullScreen, Attribution} from 'ol/control';

const emit = defineEmits(['map-click']);
const mapRef = ref(null);
let map = null;
let vectorSource = null;
let vectorLayer = null;

// Koordinaten des Stadtzentrums von Berlin, Deutschland
const berlinCoordinates = [13.404954, 52.520008];

onMounted(() => {
  // Erstellen Sie eine Vektorschicht und Datenquelle, um Markierungen anzuzeigen
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

  // Erstellen Sie die OpenLayers-Karte
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
      attribution: false, // Standard-Attributionssteuerung ausblenden
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

  // Fügen Sie ein Klickereignis zur Karte hinzu, um die Klickposition zu erhalten
  map.on('click', function(evt) {
    const coordinate = evt.coordinate;
    const lonLat = toLonLat(coordinate);
    console.log('Klickposition Koordinaten: ', lonLat);
    
    // Senden Sie die Koordinaten an die übergeordnete Komponente
    emit('map-click', lonLat);
  });
});

// Fügen Sie eine Objektmarkierung hinzu
const addItemMarker = (item) => {
  if (!map || !item.location) return;
  
  const coordinates = fromLonLat(item.location);
  
  // Erstellen Sie ein neues Feature (Markierung)
  const feature = new Feature({
    geometry: new Point(coordinates),
    title: item.title,
    description: item.description,
    id: item.id
  });
  
  // Fügen Sie die Markierung zur Vektorschicht hinzu
  vectorSource.addFeature(feature);
};

// Entfernen Sie die Markierung anhand der ID
const removeMarker = (itemId) => {
  if (!vectorSource) return;
  
  const features = vectorSource.getFeatures();
  const feature = features.find(f => f.get('id') === itemId);
  
  if (feature) {
    vectorSource.removeFeature(feature);
  }
};

// Holen Sie sich die aktuellen Koordinaten des Kartenansichtsmitte
const getCurrentCenter = () => {
  if (!map) return null;
  
  const view = map.getView();
  const center = view.getCenter();
  return toLonLat(center);
};

onUnmounted(() => {
  // Bereinigen Sie das Kartenobjekt beim Entladen der Komponente
  if (map) {
    map.setTarget(null);
    map = null;
  }
});

// Exportieren Sie Methoden zur Verwendung durch die übergeordnete Komponente
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

/* OpenLayers-Steuerelement-Stil */
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
<script setup>
import { ref, onMounted, computed } from 'vue';
import MapView from './components/MapView.vue'
import ItemForm from './components/ItemForm.vue'

const itemFormRef = ref(null);
const mapViewRef = ref(null);
const showForm = ref(false); // Steuerung der Formanzeige

// Fenstergrößenänderung überwachen
onMounted(() => {
  window.addEventListener('resize', () => {
    windowWidth.value = window.innerWidth;
  });
});

// Überprüfen, ob es sich um ein mobiles Gerät handelt
const isMobile = computed(() => {
  // Eine genauere Methode zur Erkennung mobiler Geräte verwenden
  const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;
  return isTouchDevice && window.innerWidth <= 768;
});

// Wechseln zwischen Karten- und Formularansicht auf mobilen Geräten
const toggleMobileView = () => {
  showMap.value = !showMap.value;
};

// Speichern aller hinzugefügten Gegenstände
const items = ref([]);

// Verarbeitung der Formularübermittlung
const handleItemSubmit = (itemData) => {
  const newItem = {
    id: Date.now(),  // Einfach eine eindeutige ID generieren
    ...itemData
  };
  items.value.push(newItem);
  console.log('Gegenstand hinzugefügt:', newItem);
  if (mapViewRef.value) {
    mapViewRef.value.addItemMarker(newItem);
  }
  showForm.value = false; // Formular nach der Übermittlung ausblenden
};

// Verarbeitung des Kartenklicks, um die Koordinaten an das Formular zu übergeben und das Formular anzuzeigen
const handleMapClick = (coords) => {
  if (itemFormRef.value) {
    itemFormRef.value.setLocation(coords);
  }
  showForm.value = true; // Formular beim Klicken auf die Karte anzeigen
};

// Einen Button hinzufügen, um das Formular manuell zu öffnen (optional)
const openAddItemForm = () => {
  // Mögliche alte Standortinformationen löschen oder Standardwerte festlegen
  if (itemFormRef.value) {
    itemFormRef.value.setLocation(null); // Oder die Kartenmitte übergeben
  }
  showForm.value = true;
};

// Methode zum Schließen des Formulars hinzufügen
const closeForm = () => {
  showForm.value = false;
};
</script>

<template>
  <div class="app-container">
    <MapView 
      ref="mapViewRef"
      @map-click="handleMapClick"
    />
    <button class="add-item-btn" @click="openAddItemForm">Gegenstand hinzufügen</button>
    <ItemForm 
      v-if="showForm"
      ref="itemFormRef"
      @submit-item="handleItemSubmit"
      @close-form="closeForm"
      class="item-form-overlay"
    />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app, .app-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.add-item-btn {
  position: fixed;
  top: 10px;
  right: 10px;
  padding: 8px 15px;
  background-color: white;
  color: #4CAF50;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  z-index: 1000;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.item-form-overlay {
  position: fixed;
  top: 50px;
  right: 10px;
  width: 350px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.2);
  z-index: 1000;
}
</style>

<script setup>
import { ref, onMounted, computed } from 'vue';
import MapView from './components/MapView.vue'
import ItemForm from './components/ItemForm.vue'

const itemFormRef = ref(null);
const mapViewRef = ref(null);
const showForm = ref(false); // 控制表单显示

// 监听窗口大小变化
onMounted(() => {
  window.addEventListener('resize', () => {
    windowWidth.value = window.innerWidth;
  });
});

// 判断是否为移动设备
const isMobile = computed(() => {
  // 使用更准确的移动设备检测方法
  const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;
  return isTouchDevice && window.innerWidth <= 768;
});

// 在移动设备上切换地图和表单视图
const toggleMobileView = () => {
  showMap.value = !showMap.value;
};

// 存储所有添加的物品
const items = ref([]);

// 处理表单提交
const handleItemSubmit = (itemData) => {
  const newItem = {
    id: Date.now(),  // 简单生成唯一ID
    ...itemData
  };
  items.value.push(newItem);
  console.log('物品已添加:', newItem);
  if (mapViewRef.value) {
    mapViewRef.value.addItemMarker(newItem);
  }
  showForm.value = false; // 提交后隐藏表单
};

// 处理地图点击，将坐标传递给表单并显示表单
const handleMapClick = (coords) => {
  if (itemFormRef.value) {
    itemFormRef.value.setLocation(coords);
  }
  showForm.value = true; // 点击地图显示表单
};

// 添加一个按钮来手动打开表单 (可选)
const openAddItemForm = () => {
  // 清除可能存在的旧位置信息或设置默认值
  if (itemFormRef.value) {
    itemFormRef.value.setLocation(null); // 或者传递地图中心
  }
  showForm.value = true;
};

// 添加关闭表单的方法
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
    <button class="add-item-btn" @click="openAddItemForm">添加物品</button>
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

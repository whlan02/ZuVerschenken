<template>
  <div class="item-form">
    <button class="close-btn" @click="closeForm">X</button>
    <h2>添加免费物品</h2>
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label for="title">标题</label>
        <input 
          type="text" 
          id="title" 
          v-model="formData.title" 
          required
          placeholder="物品名称"
        />
      </div>

      <div class="form-group">
        <label for="description">描述</label>
        <textarea 
          id="description" 
          v-model="formData.description" 
          placeholder="物品描述，状态等"
          rows="3"
        ></textarea>
      </div>

      <div class="form-group">
        <label>位置</label>
        <p class="location-display" v-if="formData.location">
          经度: {{ formData.location[0].toFixed(6) }}, 
          纬度: {{ formData.location[1].toFixed(6) }}
        </p>
        <p v-else class="location-hint">请在地图上选择一个位置</p>
      </div>

      <div class="form-group">
        <button type="submit" :disabled="!formData.location">提交</button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, defineEmits, defineExpose } from 'vue';

const emit = defineEmits(['submit-item', 'close-form']);

const formData = ref({
  title: '',
  description: '',
  location: null
});

// 从父组件接收的位置坐标
const setLocation = (coords) => {
  formData.value.location = coords;
};

const submitForm = () => {
  if (!formData.value.location) return;
  
  // 提交表单数据到父组件
  emit('submit-item', { ...formData.value });
};

// 关闭表单的方法
const closeForm = () => {
  emit('close-form');
};

// 导出方法，供父组件调用
defineExpose({
  setLocation
});
</script>

<style scoped>
.item-form {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.1);
  padding: 15px;
  width: 100%;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  background: transparent;
  border: none;
  font-size: 1.2rem;
  font-weight: bold;
  color: #aaa;
  cursor: pointer;
}

.close-btn:hover {
  color: #333;
}

h2 {
  color: #4CAF50;
  margin-top: 0;
  margin-bottom: 15px;
  text-align: center;
  font-size: 1.4rem;
}

.form-group {
  margin-bottom: 12px;
}

label {
  display: block;
  margin-bottom: 4px;
  font-weight: bold;
  color: #555;
  font-size: 0.9rem;
}

input, textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 0.9rem;
}

textarea {
  resize: vertical;
}

button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  width: 100%;
  font-weight: bold;
}

button:hover {
  background-color: #45a049;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.location-display {
  background-color: #f0f8ff;
  padding: 8px;
  border-radius: 4px;
  margin: 0;
  font-size: 0.85rem;
}

.location-hint {
  color: #888;
  margin: 0;
  font-style: italic;
  font-size: 0.85rem;
}
</style> 
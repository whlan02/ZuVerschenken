<template>
  <div class="item-form">
    <div class="form-header">
      <h2>Kostenlose Artikel veröffentlichen</h2>
      <button class="close-btn" @click="closeForm">
        <span class="material-icons">close</span>
      </button>
    </div>

    <form @submit.prevent="submitForm">
      <div class="form-grid">
        <!-- Bild-Upload-Bereich -->
        <div class="image-upload-section">
          <div class="image-preview" 
               :class="{ 'has-images': formData.images.length > 0 }"
               @dragover.prevent 
               @drop.prevent="handleDrop">
            <div v-if="formData.images.length === 0" class="upload-placeholder">
              <span class="material-icons">add_photo_alternate</span>
              <p>Klicken oder ziehen Sie Bilder hoch</p>
              <small>Unterstützt jpg, png-Formate, maximal 5 Bilder</small>
            </div>
            <div v-else class="image-grid">
              <div v-for="(image, index) in formData.images" :key="index" class="image-item">
                <img :src="image.preview" alt="Vorschau" />
                <button type="button" class="remove-image" @click="removeImage(index)">
                  <span class="material-icons">close</span>
                </button>
              </div>
              <div v-if="formData.images.length < 5" 
                   class="add-more-images"
                   @click="triggerImageUpload">
                <span class="material-icons">add</span>
              </div>
            </div>
          </div>
          <input
            type="file"
            ref="fileInput"
            multiple
            accept="image/*"
            class="hidden"
            @change="handleImageSelect"
          />
        </div>

        <!-- 表单输入区域 -->
        <div class="form-content">
          <div class="form-group">
            <label for="title">Titel</label>
            <input 
              type="text" 
              id="title" 
              v-model="formData.title" 
              required
              placeholder="Bitte den Artikelnamen eingeben"
            />
          </div>

          <div class="form-group">
            <label for="description">Beschreibung</label>
            <textarea 
              id="description" 
              v-model="formData.description" 
              placeholder="Bitte beschreiben Sie den Artikel detailliert, z.B. Nutzungsdauer, Zustand usw."
              rows="4"
            ></textarea>
          </div>

          <div class="form-group">
            <label>Standortinformationen</label>
            <div class="location-card" :class="{ 'has-location': formData.location }">
              <span class="material-icons">location_on</span>
              <div v-if="formData.location" class="location-info">
                <p>经度: {{ formData.location[0].toFixed(6) }}</p>
                <p>纬度: {{ formData.location[1].toFixed(6) }}</p>
              </div>
              <p v-else class="location-hint">Bitte wählen Sie einen Standort auf der Karte aus</p>
            </div>
          </div>

          <button type="submit" 
                  :disabled="!isFormValid" 
                  class="submit-button">
            <span class="material-icons">send</span>
            Post
          </button>
        </div>
      </div>
    </form>
  </div>
</template>
<script setup>
import { ref, computed, defineEmits, defineExpose } from 'vue';

const emit = defineEmits(['submit-item', 'close-form']);
const fileInput = ref(null);

const formData = ref({
  title: '',
  description: '',
  location: null,
  images: []
});

const isFormValid = computed(() => {
  return formData.value.title.trim() && 
         formData.value.location && 
         formData.value.images.length > 0;
});

const setLocation = (coords) => {
  formData.value.location = coords;
};

const triggerImageUpload = () => {
  fileInput.value.click();
};

const handleImageSelect = (event) => {
  const files = Array.from(event.target.files);
  processImages(files);
};

const handleDrop = (event) => {
  const files = Array.from(event.dataTransfer.files).filter(file => 
    file.type.startsWith('image/'));
  processImages(files);
};

const processImages = (files) => {
  const remainingSlots = 5 - formData.value.images.length;
  const filesToProcess = files.slice(0, remainingSlots);

  filesToProcess.forEach(file => {
    const reader = new FileReader();
    reader.onload = (e) => {
      formData.value.images.push({
        file: file,
        preview: e.target.result
      });
    };
    reader.readAsDataURL(file);
  });
};

const removeImage = (index) => {
  formData.value.images.splice(index, 1);
};

const submitForm = () => {
  if (!isFormValid.value) return;
  
  const formDataToSubmit = new FormData();
  formDataToSubmit.append('title', formData.value.title);
  formDataToSubmit.append('description', formData.value.description);
  formDataToSubmit.append('location', JSON.stringify(formData.value.location));
  
  formData.value.images.forEach((image, index) => {
    formDataToSubmit.append(`image${index}`, image.file);
  });

  emit('submit-item', formDataToSubmit);
};

const closeForm = () => {
  emit('close-form');
};

defineExpose({
  setLocation
});
</script>

<style scoped>
.item-form {
  background-color: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  padding: 24px;
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
}

.form-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

h2 {
  color: #2c3e50;
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
}

.close-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  cursor: pointer;
  padding: 8px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.close-btn:hover {
  background-color: #f1f5f9;
  color: #64748b;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.image-upload-section {
  grid-column: 1;
}

.form-content {
  grid-column: 2;
}

.image-preview {
  border: 2px dashed #e2e8f0;
  border-radius: 12px;
  padding: 16px;
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
}

.image-preview:hover {
  border-color: #94a3b8;
}

.upload-placeholder {
  text-align: center;
  color: #64748b;
}

.upload-placeholder .material-icons {
  font-size: 48px;
  margin-bottom: 12px;
  color: #94a3b8;
}

.image-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 12px;
  width: 100%;
}

.image-item {
  position: relative;
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
}

.image-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.remove-image {
  position: absolute;
  top: 4px;
  right: 4px;
  background: rgba(0, 0, 0, 0.5);
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  cursor: pointer;
}

.add-more-images {
  border: 2px dashed #e2e8f0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  aspect-ratio: 1;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #334155;
}

input, textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.2s;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.location-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background-color: #f8fafc;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.location-card .material-icons {
  color: #ef4444;
}

.location-info {
  font-size: 0.9rem;
  color: #475569;
}

.location-info p {
  margin: 0;
}

.location-hint {
  color: #94a3b8;
  margin: 0;
}

.submit-button {
  background-color: #3b82f6;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.2s;
}

.submit-button:hover {
  background-color: #2563eb;
}

.submit-button:disabled {
  background-color: #94a3b8;
  cursor: not-allowed;
}

.hidden {
  display: none;
}

@media (max-width: 768px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
  
  .image-upload-section,
  .form-content {
    grid-column: 1;
  }
}
</style> 

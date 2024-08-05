<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
    <div class="bg-white p-10 rounded-lg w-full max-w-3xl relative shadow-lg">
      <!-- Close Button -->
      <button @click="$emit('close')" class="absolute top-4 right-4 text-gray-600 hover:text-gray-800 transition duration-200">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>
      </button>

      <h2 class="text-3xl font-bold text-center mb-6">Upload Image</h2>
      
      <!-- Image Preview -->
      <div class="mb-6">
        <div
          @drop.prevent="handleDrop"
          @dragover.prevent
          class="border-2 border-dashed border-gray-300 p-6 text-center cursor-pointer hover:bg-gray-50 transition duration-200"
        >
          <p v-if="!file" class="text-lg text-gray-500">Drop your image here or click to upload</p>
          <p v-else class="text-lg text-gray-800">{{ file.name }}</p>
          <!-- Display Image Preview -->
          <img v-if="imagePreview" :src="imagePreview" class="mt-4 max-w-full h-auto rounded-lg shadow-md" />
        </div>
        <input type="file" @change="handleFileChange" class="hidden" ref="fileInput" />
      </div>

      <!-- Buttons -->
      <div class="flex justify-center mt-4">
        <button @click="triggerFileInput" class="bg-gray-800 hover:bg-gray-700 text-white px-6 py-3 rounded transition duration-200">
          Select File
        </button>
        <button @click="uploadImage" class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-3 rounded transition duration-200 ml-2" :disabled="!file">
          Save Image
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      file: null,
      imagePreview: null,
    };
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
      this.file = event.target.files[0];
      this.createImagePreview(this.file);
    },
    handleDrop(event) {
      this.file = event.dataTransfer.files[0];
      this.createImagePreview(this.file);
    },
    createImagePreview(file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.imagePreview = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    async uploadImage() {
      try {
        const formData = new FormData();
        formData.append('image', this.file);
        await axios.post('https://img-zcm3.onrender.com/api/images/upload', formData);
        this.$emit('uploaded');
        this.$emit('close');
      } catch (error) {
        console.error('Failed to upload image:', error);
      }
    },
  },
};
</script>

<style scoped>
input[type="file"] {
  display: none;
}
</style>

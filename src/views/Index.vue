<template>
  <div class="container mx-auto p-6 bg-gray-100 rounded-lg shadow-lg">
    <h1 class="text-4xl font-bold text-center text-gray-800 mb-6">Image Gallery</h1>

    <button @click="showUploadModal = true" class="bg-blue-600 hover:bg-blue-700 text-white font-semibold px-6 py-2 rounded transition duration-200 flex items-center">
      <!-- SVG Icon -->
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
      </svg>
      New Image
    </button>
    
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mt-6">
      <div v-for="image in images" :key="image._id" class="relative group rounded-lg overflow-hidden shadow-md transition-transform transform hover:scale-105">
        <img :src="image.url" class="w-full h-64 object-cover" />
        <div class="absolute top-2 right-2 flex space-x-2 opacity-0 group-hover:opacity-100 transition-opacity">
          <button @click="showEditModal(image)" class=" text-blue-600 font-semibold px-3 py-1 rounded">
            Edit
          </button>
          <button @click="deleteImage(image._id)" class="text-red-600 font-semibold px-3 py-1 rounded">
            Delete
          </button>
        </div>
      </div>
    </div>

    <!-- Modals -->
    <UploadModal v-if="showUploadModal" @close="showUploadModal = false" @uploaded="fetchImages" />
    <EditModal v-if="showEditModalComponent" :image="selectedImage" @close="showEditModalComponent = false" @edited="fetchImages" />
  </div>
</template>

<script>
import axios from 'axios';
import UploadModal from './UploadModal.vue';
import EditModal from './EditModal.vue';

export default {
  components: {
    UploadModal,
    EditModal,
  },
  data() {
    return {
      images: [],
      showUploadModal: false,
      showEditModalComponent: false,
      selectedImage: null,
    };
  },
  methods: {
    async fetchImages() {
      try {
        const response = await axios.get('https://imageview.onrender.com/api/images');
        this.images = response.data;
      } catch (error) {
        console.error('Failed to fetch images:', error);
      }
    },
    showEditModal(image) {
      this.selectedImage = image;
      this.showEditModalComponent = true;
    },
    async deleteImage(id) {
      try {
        await axios.delete(`https://imageview.onrender.com/api/images/${id}`);
        this.fetchImages();
      } catch (error) {
        console.error('Failed to delete image:', error);
      }
    },
  },
  mounted() {
    this.fetchImages();
  },
};
</script>

<style scoped>
.group:hover .group-hover\:opacity-100 {
  opacity: 1;
}
</style>

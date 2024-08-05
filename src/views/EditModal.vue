<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
    <div class="bg-white p-10 rounded-lg w-full max-w-3xl relative">
      <h2 class="text-3xl mb-6">Editing Image</h2>
      
      
      <!-- Image Preview and Controls -->
      <div class="relative">
        <img ref="image" :src="imagePreview" class="resizable-draggable" />
      </div>

      <div class="mb-6">
        <label class="block text-lg mb-2">Width (px)</label>
        <input v-model.number="width" type="number" class="border p-3 rounded w-full" placeholder="Width" />
        <label class="block text-lg mb-2">Height (px)</label>
        <input v-model.number="height" type="number" class="border p-3 rounded w-full" placeholder="Height" />
        <!-- <label class="block text-lg mb-2">Left (px)</label>
        <input v-model.number="left" type="number" class="border p-3 rounded w-full" placeholder="Left" />
        <label class="block text-lg mb-2">Top (px)</label>
        <input v-model.number="top" type="number" class="border p-3 rounded w-full" placeholder="Top" /> -->
      </div>

      <button @click="editImage" class="bg-blue-500 text-white px-6 py-3 rounded mt-4">
        Save Changes
      </button>
      <button @click="$emit('close')" class="text-red-600 px-6 py-3 rounded mt-4 ml-2">
        Cancel
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import interact from 'interactjs';

export default {
  props: {
    image: Object,
  },
  data() {
    return {
      width: null,
      height: null,
      left: 0,
      top: 0,
      imagePreview: null,
      originalWidth: null,
      originalHeight: null,
    };
  },
  async created() {
    await this.fetchImagePreview();
    this.initializeInteractions();
  },
  methods: {
    async fetchImagePreview() {
      try {
        const response = await axios.get(this.image.url, { responseType: 'blob' });
        const url = URL.createObjectURL(response.data);
        this.imagePreview = url;

        const img = new Image();
        img.onload = () => {
          this.originalWidth = img.width;
          this.originalHeight = img.height;
          this.width = img.width;
          this.height = img.height;
        };
        img.src = url;
      } catch (error) {
        console.error('Failed to fetch image preview:', error);
      }
    },
    initializeInteractions() {
      interact('.resizable-draggable')
        .draggable({
          listeners: {
            move: (event) => {
              const { target } = event;
              const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
              const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;

              target.style.transform = `translate(${x}px, ${y}px)`;
              target.setAttribute('data-x', x);
              target.setAttribute('data-y', y);

              this.left = x;
              this.top = y;
            },
          },
          modifiers: [
            interact.modifiers.restrictRect({
              restriction: 'parent',
            }),
          ],
          inertia: true,
        })
        .resizable({
          edges: { left: true, right: true, bottom: true, top: true },
          listeners: {
            move: (event) => {
              const { target } = event;
              const x = (parseFloat(target.getAttribute('data-x')) || 0);
              const y = (parseFloat(target.getAttribute('data-y')) || 0);

              this.width = event.rect.width;
              this.height = event.rect.height;

              target.style.width = `${event.rect.width}px`;
              target.style.height = `${event.rect.height}px`;
              target.style.transform = `translate(${x}px, ${y}px)`;
            },
          },
        });
    },
    async editImage() {
      if (
        this.width <= 0 || this.height <= 0 || 
        this.left < 0 || this.top < 0 || 
        this.width + this.left > this.originalWidth || 
        this.height + this.top > this.originalHeight
      ) {
        alert('Invalid dimensions or position');
        return;
      }
      try {
        const editData = {
          width: this.width,
          height: this.height,
          left: this.left,
          top: this.top,
        };
        await axios.put(`https://imageview.onrender.com/api/images/edit/${this.image._id}`, editData);
        this.$emit('edited');
        this.$emit('close');
      } catch (error) {
        console.error('Failed to edit image:', error);
      }
    },
  },
};
</script>

<style scoped>
.resizable-draggable {
  border: 1px solid #ddd;
  cursor: pointer;
}
</style>

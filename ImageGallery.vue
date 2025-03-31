<template>
  <div>
    <label class="block text-sm font-medium text-gray-700">{{ label }}</label>
    <div class="mt-2">
      <input accept="image/*" type="file" id="galleryImageField" name="galleryImageField" lang="es" class="hidden"
             ref="fileInput"
             @change="handleFileChange"
      />
      <div>
        <div v-for="(image, index) in images" :key="index" class="mu-image-container">
          <a :href="image.url" target="_blank"><img class="w-full" :src="image.url" :alt="image.id" /></a>
          <div class="absolute inset-y-4 right-4 rounded-md">
            <button type="button"
                    @click="handleDelete(image.id)"
                    class="rounded-full bg-gray-50 p-1 text-gray-500 border-2 border-white hover:bg-primary hover:text-white focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary">
              <XCircleIcon class="h-8 w-8" aria-hidden="true" />
            </button>
          </div>
        </div>
        <div v-if="loading" class="mu-image-container">
          <i class="fas fa-spinner fa-spin"></i>
        </div>
        <div v-if="!loading && images.length < maxImages" class="mu-image-container" style="text-align: center; padding-top: 35px;">
          <button @click="openUpload" type="button"
                  class="rounded-full bg-gray-50 p-1 text-gray-500 border-2 border-white hover:bg-primary hover:text-white focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary">
            <PlusCircleIcon class="h-10 w-10 text-green-500" aria-hidden="true" />
          </button>
          <p>{{ instructions }}</p>
        </div>
      </div>
    </div>
    <p class="mt-1 text-sm text-red-600" v-if="error">{{ error }}</p>
  </div>
</template>

<script>
import axios from "axios";
import { XCircleIcon, PlusCircleIcon } from '@heroicons/vue/24/outline'

export default {
  name: 'ImageGallery',
  components: {
    XCircleIcon, PlusCircleIcon
  },
  props: {
    apiUrl: { type: String, required: true },
    token: { type: String, required: true },
    maxImages: { type: Number, default: 20 },
    allowedTypes: { type: Array, default: () => ['image/png', 'image/jpg', 'image/jpeg', 'image/gif'] },
    maxSizeMB: { type: Number, default: 2 },
    label: { type: String, default: 'Gallery' },
    instructions: { type: String, default: 'PNG, JPG, GIF until 2MB' },
  },
  data() {
    return {
      loadingDelete: false,
      loading: false,
      error: false,
      images: [],
    };
  },
  async mounted() {
    this.configAxios = axios.create({
      headers: {
        Accept: "application/ld+json",
        "Content-Type": "application/merge-patch+json",
      },
    });
    this.configAxios.defaults.headers.Authorization = `Bearer ${this.token}`;
    this.load();
  },
  methods: {
    openUpload() {
      if (this.loading || this.loadingDelete) {
        return false;
      }
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        if (file.size > this.maxSizeMB * 1024 * 1024) {
          alert(`File size should be less than ${this.maxSizeMB}MB`);
          return;
        }
        if (!this.allowedTypes.includes(file.type)) {
          alert("Invalid format");
          return false;
        }
        const formData = new FormData();
        formData.append('file', file);
        this.uploadImage(formData);
      }
    },
    async uploadImage(formData) {
      try {
        this.loading = true;
        await this.configAxios.post(this.apiUrl, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
            'Authorization': `Bearer ${this.token}`,
          }
        }).then(response => {
          this.images.push(response.data);
        }).catch(error => {
          console.log(error);
          alert("Problem to upload the image");
        });
      } catch (error) {
        console.log(error);
        alert("Problem to upload the image");
      }
      this.loading = false;
    },
    findImage(imageId) {
      return this.images.find(image => image.id === imageId) || false;
    },
    removeImage(imageId) {
      this.images = this.images.filter(image => image.id !== imageId);
    },
    async handleDelete(imageId) {
      if (this.loading || this.loadingDelete) {
        return false;
      }
      try {
        const image = this.findImage(imageId);
        if (image) {
          this.loadingDelete = true;
          await this.configAxios.delete(`${this.apiUrl}/${image.id}`)
            .then(() => {
              this.removeImage(image.id);
              this.loadingDelete = false;
            })
            .catch(error => {
              this.loadingDelete = false;
              alert("Problem to remove the image");
            });
        }
      } catch (error) {
        alert("Problem to remove the image");
      }
    },
    async load() {
      this.loading = true;
      await this.configAxios.get(this.apiUrl)
        .then(response => {
          this.images = response.data;
          this.loading = false;
        })
        .catch(error => {
          this.loading = false;
          console.log(error);
          alert("Problem to load the images of the gallery");
        });
    }
  }
}
</script>

<style scoped>
.mu-image-container {
  position: relative;
  margin-bottom: 10px;
}
</style>

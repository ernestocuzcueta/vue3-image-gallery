<template>
  <div>
    <label class="block text-sm font-medium text-gray-700">{{$t('Gallery')}}</label>
    <div class="mt-2">
      <input accept="image/*" type="file" id="galleryImageField" name="galleryImageField" lang="es" class="hide"
             ref="fileInput"
             @change="handleFileChange"
      />
      <div  >
        <div v-for="(image, index) in images" :key="index" class="mu-image-container" >
          <a :href="image.url" target="_blank"><img class=" w-full" :src="image.url" :alt="image.id"/></a>
          <div class="absolute inset-y-4 right-4 rounded-md">
            <button type="button"
                    @click="handleDelete(image.id)"
                    class="rounded-full bg-gray-50 p-1 text-gray-500 border-2 border-white hover:!bg-primary  hover:text-white focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary">
              <XCircleIcon class="h-8 w-8" aria-hidden="true"/>
            </button>
          </div>
        </div>
        <div v-if="loading" class="mu-image-container" >
          <i class="fas fa-spinner fa-spin"></i>
        </div>
        <div v-if="!loading && images.length<20" class="mu-image-container" style="text-align: center; padding-top: 35px;">
          <button @click="openUpload()" type="button"
                  class="rounded-full bg-gray-50 p-1 text-gray-500 border-2 border-white hover:!bg-primary  hover:text-white focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary">
            <PlusCircleIcon class="h-10 w-10 text-green-500" aria-hidden="true"/>
          </button>
          <p>{{$t('PNG, JPG, GIF until 2MB')}}</p>
        </div>
      </div>
    </div>
    <p class="mt-1 text-sm text-red-600" v-if="error">{{ error }}</p>
  </div>
</template>

<script>
import axios from "axios";
import { XMarkIcon, PhotoIcon, XCircleIcon, PlusCircleIcon } from '@heroicons/vue/24/outline'

export default {
  name: 'ImageGallery',
  components: {
    XMarkIcon, PhotoIcon, XCircleIcon,PlusCircleIcon
  },
  props: {
    businessId: String
  },
  data() {
    return {
      loadingDelete: false,
      loading: false,
      open: false,
      buttonTitle: 'Select image',
      errorBase64Message: false,
      error: false,
      selected: false,
      path: '/api/widget-gallery/' + this.businessId,
      images: [],
    }
  },
  async mounted() {
    this.configAxios = axios.create({
      headers: {
        Accept: "application/ld+json",
        "Content-Type": "application/merge-patch+json",
      },
    });
    this.configAxios.defaults.headers.Authorization = `Bearer ${window.token}`;
    this.load();
    console.log(this.images);
  },
  methods: {
    openUpload(){
      if(this.loading || this.loadingDelete){
        return false;
      }
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
      const file = event.target.files[0];  // Obtener el archivo seleccionado
      if (file) {
        if (file.size > 2 * 1024 * 1024) {  // 2MB
          alert(this.$t("Invalid size"));
          return;
        }
        if (!['image/png', 'image/jpg', 'image/jpeg', 'image/gif'].includes(file.type)) {
          alert(this.$t("Invalid format"));
          return false;
        }
        const formData = new FormData();
        formData.append('file', file); // 'file' es el nombre del campo que el servidor espera
        this.uploadImage(formData);
      }
    },
    async uploadImage(formData) {
      try {
        this.loading=true;
        // Realizar la solicitud POST con el archivo
        await this.configAxios.post(this.path, formData, {
          headers: {
            'Content-Type': 'multipart/form-data',  // Indicar que estamos enviando datos en formato de formulario (incluyendo archivos)
            'Authorization': `Bearer ${window.token}`,  // Si necesitas autenticación
          }
        }).then(response => {
          this.images.push(response.data);
        }).catch(error => {
          console.log(error);
          alert(this.$t("Problem to upload the image"));
        });
      } catch (error) {
        console.log(error);
        alert(this.$t("Problem to upload the image"));
      }
      this.loading=false;
    },
    findImage(imageId) {
      for (let image of this.images) {
        if(image.id == imageId){
          return image;
        }
      };
      return false;
    },
    removeImage(imageId){
      let imagesTemp=[];
      for (let image of this.images) {
        if(image.id != imageId){
          imagesTemp.push(image);
        }
      }
      this.images = imagesTemp;
    },
    async handleDelete(imageId) {
      if(this.loading || this.loadingDelete){
        return false;
      }
      try {
        const image = this.findImage(imageId);
        if(image){
          this.loadingDelete=true;
          await this.configAxios.delete(`${this.path}/${image.id}`)
              .then(response=>{
                this.removeImage(image.id);
                this.loadingDelete=false;
              })
              .catch(error => {
                this.loadingDelete=false
                alert(this.$t("Problem to remove the image"));
              });
        }
      } catch (error) {
        alert(this.$t("Problem to remove the image"));
      }



    },
    async load() {
      this.loading=true;
      await this.configAxios.get(`${this.path}`)
          .then(response => {
            console.log(response);
            this.images = response.data;
            this.loading=false;
          })
          .catch(error => {
            this.loading=false;
            console.log(error);
            alert(this.$t("Problem to load the images of the gallery"));
          });
    }
  }
}


</script>

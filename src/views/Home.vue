<script setup>
  import { ref, reactive, computed } from 'vue'
  import axios from 'axios'
  import ImageEmpty from './../components/ImageEmpty.vue'

  const errorMessage = ref('')
  const successMessage = ref('')
  const disabledBtn = ref(false)
  const loading = ref(false)
  const getImages = ref(JSON.parse(localStorage.getItem('gallery')) || [])

  const images = reactive({
    image: '',
    previewImage: '',
  })

  const updateImage = () => {
    getImages.value = JSON.parse(localStorage.getItem('gallery')) || []
  }

  const getBase64StringFromDataURL = dataURL =>
    dataURL.replace('data:', '').replace(/^.+,/, '')

  const createBase64Image = file => {
    const render = new FileReader()
    render.onload = e => {
      images.image = getBase64StringFromDataURL(e.target.result)
    }
    render.readAsDataURL(file)
  }

  const handleFileInput = e => {
    images.previewImage = URL.createObjectURL(e.target.files[0])
    const getImageFile = e.target.files[0]

    if (getImageFile) {
      //validation extension file
      const allowedExtensions = ['jpg', 'jpeg', 'bmp', 'png', 'gif']

      const fileExtension = getImageFile.name.split('.').pop().toLowerCase()

      if (allowedExtensions.indexOf(fileExtension) === -1) {
        errorMessage.value = 'Hanya boleh file gambar'
        return
      }

      const maxSize = 2 * 1024 * 1024

      if (getImageFile.size > maxSize) {
        errorMessage.value = 'File Image Max 2MB'
        disabledBtn.value = true
        return
      }
      errorMessage.value = ''
      disabledBtn.value = false
    }
    createBase64Image(getImageFile)
  }
  const uploadImage = async () => {
    const apiKey = '18d6c5553c33dd5c1f07ef6c645b0372'
    const formData = new FormData()
    formData.append('image', images.image)

    await axios
      .post(
        `https://api.imgbb.com/1/upload?expiration=600&key=${apiKey}`,
        formData
      )
      .then(function (res) {
        loading.value = true

        const existingData = localStorage.getItem('gallery')
        const galleryData = existingData ? JSON.parse(existingData) : []

        const newResponse = {
          id: galleryData.length + 1,
          display_url: res.data.data.display_url,
          url_viewer: res.data.data.url_viewer,
        }

        galleryData.push(newResponse)
        localStorage.setItem('gallery', JSON.stringify(galleryData))

        successMessage.value = 'Gambar Berhasil Di Upload'

        setTimeout(() => {
          successMessage.value = ''
          images.image = ''
          images.previewImage = ''
          loading.value = false
          document.getElementById('imageUpload').value = ''
          updateImage()
        }, 2000)
        // console.log('response', res.data.data)
      })
      .catch(function (error) {
        errorMessage.value = error.response.data.error.message

        setTimeout(() => {
          errorMessage.value = ''
        }, 3000)
      })
  }
</script>

<template>
  <v-layout>
    <v-app-bar class="rounded rounded-md" title="Rens Image"> </v-app-bar>
    <v-main class="d-flex align-center justify-center">
      <v-container>
        <v-row>
          <v-col sm="12" lg="6" class="mt-10 mx-auto">
            <v-card class="w-lg-50 w-100 mx-auto px-4 py-4">
              <h5 class="text-h5 d-flex align-center">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="32"
                  height="32"
                  class="mr-2"
                  viewBox="0 0 24 24"
                >
                  <path
                    fill="none"
                    stroke="currentColor"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="1.5"
                    d="M12 9.75v6.75m0 0l-3-3m3 3l3-3m-8.25 6a4.5 4.5 0 0 1-1.41-8.775a5.25 5.25 0 0 1 10.233-2.33a3 3 0 0 1 3.758 3.848A3.752 3.752 0 0 1 18 19.5H6.75Z"
                  />
                </svg>
                <span>Upload Image</span>
              </h5>
              <form @submit.prevent="uploadImage" enctype="multipart/form-data">
                <div class="mb-5 d-flex justify-center">
                  <div class="">
                    <v-img
                      v-if="images.previewImage"
                      :src="images.previewImage"
                      width="200"
                      class="rounded rounded-lg"
                    />
                  </div>
                </div>
                <v-alert
                  :text="successMessage ? successMessage : errorMessage"
                  v-if="successMessage ? successMessage : errorMessage"
                  :type="successMessage ? 'success' : 'error'"
                  class="mb-3"
                ></v-alert>
                <v-file-input
                  clearable
                  id="imageUpload"
                  v-on:change="handleFileInput"
                  accept="image/jpg,image/jpeg,image/bmp,image/png,image/gif"
                  label="Upload Image"
                  variant="outlined"
                ></v-file-input>
                <span class="text-caption font-weight-bold"
                  >File extension:
                  <span class="text-danger"
                    >.jpg, .jpeg, .bmp, .png, .gif</span
                  ></span
                >
                <v-card-actions>
                  <v-btn
                    :disabled="disabledBtn"
                    :loading="loading"
                    @click="uploadImage()"
                    class="bg-primary"
                    >Upload</v-btn
                  >
                </v-card-actions>
              </form>
            </v-card>
          </v-col>
          <v-col cols="12" class="mt-10">
            <div class="mb-5">
              <h3 class="text-h5">Gallery</h3>
              <p>Daftar List Foto</p>
            </div>
            <v-row>
              <v-col
                v-if="getImages.length"
                v-for="image in getImages"
                :key="image.id"
                cols="4"
              >
                <v-card>
                  <v-img :src="image.display_url" aspect-ratio="16/9" cover>
                  </v-img>
                </v-card>
              </v-col>
              <v-col
                v-else
                class="d-flex justify-center align-center py-10"
                cols="12"
              >
                <div class="w-100 text-center">
                  <ImageEmpty class="w-25 h-25 mb-5" />
                  <p>Belum Tersedia Foto.</p>
                </div>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-layout>
</template>

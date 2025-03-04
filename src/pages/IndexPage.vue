<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section>
        <div class="q-gutter-md">
          <div class="row items-center q-mb-md">
            <!-- Video and Canvas Elements -->
            <video ref="video" autoplay playsinline class="q-mb-md" style="width: 100%; max-height: 300px;"></video>
            <canvas ref="canvas" style="display: none;"></canvas>
            <q-btn @click="captureImage" label="Capture Image" color="primary" class="full-width" />
          </div>

          <!-- Capture -->
          <div class="row items-center q-mt-md">
            <q-btn v-if="imageSrc" @click="uploadImage" label="Upload Image" color="secondary" class="full-width" />
            <q-img v-if="imageSrc" :src="imageSrc" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_images }}
          </div>

          <!-- File picker and image previews -->
          <div class="row items-center q-mt-md">
            <q-file v-model="image" label="Pick one file" filled class="full-width"
              @update:model-value="handleUpload" />
            <q-img v-if="imageUrl" :src="imageUrl" spinner-color="white" class="q-mb-md"
              style="width: 100%; max-height: 300px;" />
            {{ result_files }}
          </div>

          <div>
          </div>
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import axios from 'axios';
import { ref } from 'vue';

export default {
  data() {
    return {
      imageSrc: null,
    };
  },
  setup() {
    const image = ref(null);
    const imageUrl = ref('');
    const ModelUrl = ref('https://corn-diseases.onrender.com/predict_image');
    const result_files = ref('');
    const result_images= ref('');
    
    const handleUpload = async () => {
      console.log('handleUpload is triggered');
      const image_name = image.value.name;
      console.log('image.name: ', image_name);
      if (image.value) {
        imageUrl.value = URL.createObjectURL(image.value);
        const dataURL = imageUrl.value;
        const blob = await fetch(dataURL).then((res) => res.blob());
        console.log(blob);

        // Create FormData object and append the image Blob
        const formData = new FormData();
        formData.append('file', blob, image_name); // 'file' is the key here, matching Python's files parameter
        console.log(formData);

        // Retry logic for uploading the image
        const response = await makeRequestWithRetry(formData);
        if (response) {
          result_files.value = response.data;
          console.log('Response from server:', response.data);
        } else {
          console.error('Failed to upload image after multiple attempts');
        }
      }
    };

    // Retry mechanism for axios request
    const makeRequestWithRetry = async (formData, retries = 3, delay = 3000) => {
      for (let attempt = 0; attempt < retries; attempt++) {
        try {
          const response = await axios.post(ModelUrl.value, formData, {
            // No need for 'Content-Type': 'multipart/form-data'
            // Axios will automatically handle it when using FormData
            timeout: 10000, // Increase timeout to 10 seconds for slow responses
          });
          return response;
        } catch (error) {
          console.error(`Attempt ${attempt + 1} failed: ${error.message}. Retrying in ${delay / 1000} seconds...`);
          await new Promise((resolve) => setTimeout(resolve, delay));
        }
      }
      console.error('Request failed after all retry attempts');
      return null;
    };
    return {
      image,
      imageUrl,
      ModelUrl,
      handleUpload,
      result_files,
      result_images
    };
  },
  mounted() {
    this.startCamera();
  },
  methods: {
    async startCamera() {
      const video = this.$refs.video;
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;
        video.play();
      } catch (err) {
        console.error('Error accessing webcam:', err);
      }
    },
    captureImage() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      const context = canvas.getContext('2d');
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      this.imageSrc = canvas.toDataURL('image/png');
    },
    async uploadImage() {
      // Convert the data URL to a Blob
      const dataURL = this.imageSrc;
      console.log(dataURL);
      const blob = await fetch(dataURL).then((res) => res.blob());
      console.log(blob);

      // Create FormData object and append the image Blob
      const formData = new FormData();
      formData.append('file', blob, 'captured-image.png'); // 'file' as the key, matching the Python code
      console.log(formData);

      // Retry logic for uploading the image
      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_images = response.data;
        console.log('Response from server:', this.result_images);
      } else {
        console.error('Failed to upload image after multiple attempts');
      }
    },
  },
};
</script>


<style scoped>
/* Additional styles to enhance UI */
.q-page {
  display: flex;
  justify-content: center;
  align-items: center;
}

.q-card {
  width: 100%;
  max-width: 600px;
}

.q-card-section {
  padding: 20px;
}

.q-file {
  margin-top: 10px;
}

.q-img {
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
</style>

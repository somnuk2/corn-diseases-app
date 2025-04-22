<template>
  <q-page class="q-pa-md">
    <q-card>
      <q-card-section>
        <div class="q-gutter-md">
          <!-- Camera Selection -->
          <div class="row items-center q-mb-md">
            <q-select
              v-model="selectedCamera"
              :options="cameraOptions"
              label="Select Camera"
              outlined
              dense
              class="col"
            />
            <q-btn 
              @click="switchCamera" 
              label="Switch Camera" 
              color="grey" 
              class="q-ml-md"
            />
          </div>
          
          <!-- Video and Canvas Elements -->
          <div class="row items-center q-mb-md">
            <video ref="video" autoplay playsinline class="q-mb-md" style="width: 100%; max-height: 300px;"></video>
            <canvas ref="canvas" style="display: none;"></canvas>
            <q-btn @click="captureImage" label="Capture Image" color="primary" class="full-width" />
          </div>
          
          <!-- Capture Preview -->
          <div class="row items-center q-mt-md">
            <q-btn v-if="imageSrc" @click="uploadImage" label="Upload Image" color="secondary" class="full-width" />
            <q-img v-if="imageSrc" :src="imageSrc" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_images }}
          </div>
          
          <!-- File Upload -->
          <div class="row items-center q-mt-md">
            <q-file v-model="image" label="Pick one file" filled class="full-width" @update:model-value="handleUpload" />
            <q-img v-if="imageUrl" :src="imageUrl" spinner-color="white" class="q-mb-md" style="width: 100%; max-height: 300px;" />
            {{ result_files }}
          </div>
          
          <!-- Disease Info -->
          <div class="q-pa-md">
            <q-list>
              <q-item>
                <q-item-section>
                  <q-item-label>สาเหตุ:</q-item-label>
                  <q-item-label caption lines="2">{{ Caused_by }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>อาการ:</q-item-label>
                  <q-item-label caption> {{ Symptoms }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>ผลกระทบ:</q-item-label>
                  <q-item-label caption> {{ Impact }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator spaced inset />

              <q-item>
                <q-item-section>
                  <q-item-label>การควบคุม:</q-item-label>
                  <q-item-label caption> {{ Management }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
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
      selectedCamera: 'environment', // Default to rear camera
      cameraOptions: [
        { label: 'Rear Camera', value: 'environment' },
        { label: 'Front Camera', value: 'user' }
      ],
      currentStream: null,
      image: ref(null),
      imageUrl: ref(''),
      ModelUrl: ref('https://corn-diseases.onrender.com/predict_image'),
      result_files: ref(''),
      result_images: ref(''),
      Caused_by: ref(''),
      Symptoms: ref(''),
      Impact: ref(''),
      Management: ref('')
    };
  },

  mounted() {
    this.startCamera();
  },

  beforeUnmount() {
    this.stopCamera();
  },

  methods: {
    async startCamera() {
      this.stopCamera(); // Stop any existing stream first

      const video = this.$refs.video;
      try {
        const constraints = {
          video: {
            facingMode: this.selectedCamera,
            width: { ideal: 1280 },
            height: { ideal: 720 }
          }
        };
        
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        this.currentStream = stream;
        video.srcObject = stream;
        video.play();
      } catch (err) {
        console.error('Error accessing camera:', err);
        // Fallback to any available camera
        try {
          const stream = await navigator.mediaDevices.getUserMedia({ video: true });
          this.currentStream = stream;
          video.srcObject = stream;
          video.play();
        } catch (fallbackErr) {
          console.error('Could not access any camera:', fallbackErr);
        }
      }
    },

    stopCamera() {
      if (this.currentStream) {
        this.currentStream.getTracks().forEach(track => track.stop());
        this.currentStream = null;
      }
    },

    switchCamera() {
      this.selectedCamera = this.selectedCamera === 'user' ? 'environment' : 'user';
      this.startCamera();
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
      if (!this.imageSrc) return;

      const blob = await fetch(this.imageSrc).then((res) => res.blob());
      const formData = new FormData();
      formData.append('file', blob, 'captured-image.png');

      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_images = this.checkDiseasePrediction(response);
      } else {
        console.error('Failed to upload image');
      }
    },

    checkDiseasePrediction(predictedLabel) {
      const diseases = {
        'CDM': {
          Caused_by: "เชื้อราในดิน เช่น Fusarium หรือ Pythium",
          Symptoms: "มักจะพบในต้นกล้าข้าวโพดที่ยังอ่อนอยู่ โดยต้นกล้าจะตาย หรือใบเหลืองและมีอาการเน่าในส่วนของลำต้นหรือราก",
          Impact: "โรคนี้ทำให้เกิดการงอกของเมล็ดที่ไม่สมบูรณ์หรือมีอัตราการงอกต่ำ จึงส่งผลให้จำนวนต้นข้าวโพดลดลง",
          Management: "การหมุนเวียนพืช การใช้สารเคมีป้องกันเชื้อราในการรักษาเมล็ดพันธุ์ และการระบายน้ำที่ดีในดิน"
        },
        'HT': {
          Caused_by: "การถูกทำลายโดยลูกเห็บจากพายุ",
          Symptoms: "ใบข้าวโพดจะมีรอยขาดหรือถูกฉีกขาดจากแรงกระแทกของลูกเห็บ หากเกิดความเสียหายรุนแรง อาจทำให้ลำต้นหรือฝักข้าวโพดถูกทำลาย",
          Impact: "ความเสียหายจากลูกเห็บไม่ได้เกิดจากเชื้อโรค แต่จะทำให้ต้นข้าวโพดอ่อนแอต่อการติดเชื้อรอง และลดผลผลิต",
          Management: "ไม่สามารถป้องกันความเสียหายจากลูกเห็บได้ แต่การจัดการฟิลด์อย่างเหมาะสม เช่น การเก็บเกี่ยวในเวลาที่เหมาะสม จะช่วยลดผลกระทบได้"
        },
        // ... other disease definitions ...
      };

      if (diseases[predictedLabel]) {
        const disease = diseases[predictedLabel];
        this.Caused_by = disease.Caused_by;
        this.Symptoms = disease.Symptoms;
        this.Impact = disease.Impact;
        this.Management = disease.Management;
        return predictedLabel;
      } else {
        console.error('Unknown disease prediction');
        return null;
      }
    },

    async handleUpload() {
      if (!this.image) return;

      this.imageUrl = URL.createObjectURL(this.image);
      const blob = await fetch(this.imageUrl).then((res) => res.blob());
      const formData = new FormData();
      formData.append('file', blob, this.image.name);

      const response = await this.makeRequestWithRetry(formData);
      if (response) {
        this.result_files = this.checkDiseasePrediction(response);
      } else {
        console.error('Failed to upload file');
      }
    },

    async makeRequestWithRetry(formData, retries = 3, delay = 3000) {
      const class_names = ['CDM', 'HT', 'MDMV', 'NCLB', 'SCLB', 'SCMV', 'SR'];
      
      for (let attempt = 0; attempt < retries; attempt++) {
        try {
          const response = await axios.post(this.ModelUrl, formData, {
            timeout: 10000,
          });
          
          if (response?.data?.prediction && class_names.includes(response.data.prediction)) {
            return response.data.prediction;
          }
        } catch (error) {
          console.error(`Attempt ${attempt + 1} failed:`, error);
          if (attempt < retries - 1) {
            await new Promise(resolve => setTimeout(resolve, delay));
          }
        }
      }
      return null;
    }
  },

  watch: {
    selectedCamera() {
      this.startCamera();
    }
  }
};
</script>

<style scoped>
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
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
          <div class="q-pa-md" style="max-width: 350px">
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
    };
  },
  setup() {
    const image = ref(null);
    const imageUrl = ref('');
    const ModelUrl = ref('https://corn-diseases.onrender.com/predict_image');
    const result_files = ref('');
    const result_images = ref('');
    const Caused_by = ref('');
    const Symptoms = ref('');
    const Impact = ref('');
    const Management = ref('');

    // Function to check the disease prediction
    const checkDiseasePrediction = (predictedLabel) => {
      // Check each disease with an if statement
      if (predictedLabel === 'CDM') {
        console.log('Prediction: Corn Disease - CDM');
        Caused_by = "เชื้อราในดิน เช่น Fusarium หรือ Pythium";
        Symptoms = "มักจะพบในต้นกล้าข้าวโพดที่ยังอ่อนอยู่ โดยต้นกล้าจะตาย หรือใบเหลืองและมีอาการเน่าในส่วนของลำต้นหรือราก";
        Impact = "โรคนี้ทำให้เกิดการงอกของเมล็ดที่ไม่สมบูรณ์หรือมีอัตราการงอกต่ำ จึงส่งผลให้จำนวนต้นข้าวโพดลดลง";
        Management = "การหมุนเวียนพืช การใช้สารเคมีป้องกันเชื้อราในการรักษาเมล็ดพันธุ์ และการระบายน้ำที่ดีในดิน";
        return 'CDM (Corn Damping-Off Disease)';
      } else if (predictedLabel === 'HT') {
        console.log('Prediction: Corn Disease - HT');
        Caused_by = "การถูกทำลายโดยลูกเห็บจากพายุ";
        Symptoms = "ใบข้าวโพดจะมีรอยขาดหรือถูกฉีกขาดจากแรงกระแทกของลูกเห็บ หากเกิดความเสียหายรุนแรง อาจทำให้ลำต้นหรือฝักข้าวโพดถูกทำลาย";
        Impact = "ความเสียหายจากลูกเห็บไม่ได้เกิดจากเชื้อโรค แต่จะทำให้ต้นข้าวโพดอ่อนแอต่อการติดเชื้อรอง และลดผลผลิต";
        Management = "ไม่สามารถป้องกันความเสียหายจากลูกเห็บได้ แต่การจัดการฟิลด์อย่างเหมาะสม เช่น การเก็บเกี่ยวในเวลาที่เหมาะสม จะช่วยลดผลกระทบได้";
        return 'HT (Hail Damage)';
      } else if (predictedLabel === 'MDMV') {
        console.log('Prediction: Corn Disease - MDMV');
        Caused_by = "ไวรัสมอซาย์ข้าวโพดมิสซูรี่ (MDMV) ที่ถูกส่งผ่านโดยเพลี้ย";
        Symptoms = "จะทำให้ใบข้าวโพดมีลายเส้นเหลืองหรือเขียวในลักษณะมอซาอิก โดยต้นข้าวโพดอาจเติบโตช้าและพัฒนาฝักไม่สมบูรณ์";
        Impact = "การติดเชื้อ MDMV อาจทำให้ผลผลิตลดลงโดยเฉพาะถ้าต้นข้าวโพดติดเชื้อในช่วงต้นของการเจริญเติบโต";
        Management = "ควบคุมเพลี้ยด้วยการใช้ยาฆ่าแมลง หรือการใช้พันธุ์ข้าวโพดที่ทนทานต่อโรค";
        return 'MDMV (Maize Dwarf Mosaic Virus)';
      } else if (predictedLabel === 'NCLB') {
        console.log('Prediction: Corn Disease - NCLB');
        Caused_by = "เชื้อราที่ชื่อ Exserohilum turcicum";
        Symptoms = "ใบข้าวโพดจะมีรอยไหม้ยาวและเป็นสีเทา-เขียว ซึ่งจะเริ่มจากจุดเล็กๆ แล้วขยายใหญ่ขึ้น";
        Impact = "โรคนี้จะทำให้พื้นที่การสังเคราะห์แสงของใบลดลงและผลผลิตลดลง ถ้าเกิดการติดเชื้ออย่างรุนแรง";
        Management = "การใช้พันธุ์ข้าวโพดที่ทนทาน การฉีดพ่นยาฆ่าเชื้อรา และการหมุนเวียนพืชสามารถช่วยในการควบคุมโรคนี้";
        return 'NCLB (Northern Corn Leaf Blight)';
      } else if (predictedLabel === 'SCLB') {
        console.log('Prediction: Corn Disease - SCLB');
        Caused_by = "เชื้อรา Cochliobolus heterostrophus";
        Symptoms = "ทำให้ใบข้าวโพดมีรอยไหม้ที่ขอบใบ ซึ่งจะมีสีน้ำตาลถึงสีเทาและมีขอบสีเหลือง";
        Impact = "โรคนี้สามารถลดผลผลิตได้อย่างมากถ้าเกิดขึ้นในช่วงต้นฤดูกาลที่อากาศร้อนและมีความชื้น";
        Management = "การใช้พันธุ์ข้าวโพดที่ทนทาน การพ่นยาฆ่าเชื้อรา และการหมุนเวียนพืช";
        return 'SCLB (Southern Corn Leaf Blight)';
      } else if (predictedLabel === 'SCMV') {
        console.log('Prediction: Corn Disease - SCMV');
        Caused_by = "ไวรัสมอซาย์อ้อย (SCMV) ที่ถูกส่งผ่านโดยเพลี้ย";
        Symptoms = "จะทำให้ใบข้าวโพดมีลายเส้นหรือมอซาอิกที่เป็นสีเหลืองและขอบใบที่ตาย เนื่องจากการติดเชื้อ";
        Impact = "SCMV สามารถลดผลผลิตและทำให้ข้าวโพดมีคุณภาพต่ำได้";
        Management = "การควบคุมเพลี้ยและการใช้พันธุ์ข้าวโพดที่ทนทานเป็นวิธีการควบคุมโรคนี้";
        return 'SCMV (Sugarcane Mosaic Virus)';
      } else if (predictedLabel === 'SR') {
        console.log('Prediction: Corn Disease - SR');
        Caused_by = "เชื้อแบคทีเรีย Pantoea stewartii";
        Symptoms = "ทำให้ใบข้าวโพดมีรอยเส้นสีเหลืองหรือสีน้ำตาลตามแนวเส้นกลางใบ โดยบางครั้งก็ทำให้ต้นข้าวโพดเติบโตช้าและฝักไม่สมบูรณ์";
        Impact = "โรคนี้สามารถลดผลผลิตได้อย่างมากถ้าต้นข้าวโพดติดเชื้อในช่วงเริ่มต้นของการเจริญเติบโต";
        Management = "โรคนี้แพร่กระจายผ่านเพลี้ยข้าวโพด ดังนั้นการควบคุมเพลี้ยและการใช้พันธุ์ข้าวโพดที่ทนทานจะช่วยลดความเสียหายจากโรคนี้ได้";
        return "SR (Stewart's Wilt)";
      } else {
        // If the prediction doesn't match any known diseases, log an error
        console.error('Prediction does not match any known diseases.');
        return null;
      }
    };

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
          result_files.value = checkDiseasePrediction(response.data);
          console.log('Response from server:', response.data);
        } else {
          console.error('Failed to upload image after multiple attempts');
        }
      }
    };

    // Retry mechanism for axios request
    const makeRequestWithRetry = async (formData, retries = 3, delay = 3000) => {
      const class_names = ['CDM', 'HT', 'MDMV', 'NCLB', 'SCLB', 'SCMV', 'SR'];
      for (let attempt = 0; attempt < retries; attempt++) {
        try {
          const response = await axios.post(ModelUrl.value, formData, {
            // No need for 'Content-Type': 'multipart/form-data'
            // Axios will automatically handle it when using FormData
            timeout: 10000, // Increase timeout to 10 seconds for slow responses
          });
          // Check if the response contains valid prediction data
          if (response && response.data && response.data.prediction) {
            const predictedLabel = response.data.prediction;
            // Check if the predicted label is one of the 7 diseases in the class_names array
            if (class_names.includes(predictedLabel)) {
              // If it matches a disease in the list, log and return the prediction
              console.log(`Prediction: ${predictedLabel}`);
              return predictedLabel;
            } else {
              // If the prediction doesn't match any disease, log an error
              console.error('Prediction does not match any known diseases.');
              return null;
            }
          } else {
            // If the response does not contain the expected data, log an error
            console.error('Invalid response data:', response.data);
            return null;
          }
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
      result_images,
      Caused_by,
      Symptoms,
      Impact,
      Management,
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

<template>
  <div>
    <div class="m-5">
      <div :class="text == '' ? '' : 'row m-2'">
        <h1>TExtractImage</h1>
        <div class="col">
          <div class="input-group mb-2">
            <input
              class="form-control"
              type="file"
              @change="selectImage"
              ref="fileInput"
              accept=".jpeg, .jpg, .png"
            />
            <button
              :disabled="text == ''"
              class="btn btn-danger input-group-text"
              @click="clearText"
            >
              &times;
            </button>
          </div>
        </div>
        <div v-if="text != ''" class="col border rounded">
          <div class="d-flex justify-content-center">
            <img class="shadow m-2" :src="base64" height="200" width="200" />
          </div>
        </div>
      </div>
      <div class="clipboard-container">
        <button class="clipboard-btn" @click="copyText">
          <i class="bi bi-clipboard"></i>
        </button>
        <textarea class="form-control mb-3" v-model="text" rows="10" cols="100" readonly></textarea>
      </div>

      <p>TExtractImage is an online application in which you can image convert to text.</p>
    </div>
  </div>
</template>

<script>
import { createWorker } from 'tesseract.js'

export default {
  data() {
    return {
      base64: '',
      inputFile: [],
      text: '',
      file: [],
    }
  },

  methods: {
    copyText() {
      navigator.clipboard.writeText(this.text)
    },

    clearText() {
      this.text = ''
      const dataTransfer = new DataTransfer()
      this.$refs.fileInput.files = dataTransfer.files
    },

    pasteImage(e) {
      const file = e.clipboardData.files[0]
      this.file = file
      const fileType = file.type
      if (fileType === 'image/jpeg' || fileType === 'image/jpg' || fileType === 'image/png') {
        const dataTransfer = new DataTransfer()
        dataTransfer.items.add(file)
        this.$refs.fileInput.files = dataTransfer.files

        this.convertImageToText()

        const reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = async () => {
          const base64 = reader.result
          this.base64 = base64
        }
        return
      }

      console.log('Please select an image only')
    },

    selectImage(e) {
      const file = e.target.files[0]
      this.file = file

      this.convertImageToText()

      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = async () => {
        const base64 = reader.result
        this.base64 = base64
      }
    },

    async convertImageToText() {
      const worker = await createWorker('eng')
      const ret = await worker.recognize(this.file)
      this.text = ret.data.text
      await worker.terminate()
    },
  },

  mounted() {
    window.addEventListener('paste', this.pasteImage)
  },
}
</script>

<style scoped>
.clipboard-container {
  position: relative;
}

.clipboard-btn {
  position: absolute;
  top: 5px;
  right: 10px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #007bff;
}

.clipboard-btn:hover {
  color: #0056b3;
}
</style>

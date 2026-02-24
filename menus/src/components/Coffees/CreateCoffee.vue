<template>
  <div class="container">
    <h1>Create Coffee</h1>

    <form @submit.prevent="createCoffee">

      <!-- NAME -->
      <div class="mb-3">
        <label>Name:</label>
        <input v-model="coffee.name" class="form-control" required>
      </div>

      <!-- PRICE -->
      <div class="mb-3">
        <label>Price:</label>
        <input v-model.number="coffee.price" type="number" class="form-control" required>
      </div>

      <!-- TYPE -->
      <div class="mb-3">
        <label>Type:</label>
        <select v-model="coffee.type" class="form-control">
          <option value="ร้อน">ร้อน</option>
          <option value="เย็น">เย็น</option>
          <option value="ปั่น">ปั่น</option>
        </select>
      </div>

      <!-- STATUS -->
      <div class="mb-3">
        <label>Status:</label>
        <select v-model="coffee.status" class="form-control">
          <option value="มีจำหน่าย">มีจำหน่าย</option>
          <option value="หมด">หมด</option>
        </select>
      </div>

      <!-- ⭐ Upload Images -->
      <div class="mb-3">
        <label>Upload Images:</label>
        <UploadImage @uploaded="onUploaded" />
      </div>

      <!-- ⭐ Thumbnail -->
      <div class="mb-3">
        <label>Thumbnail:</label>
        <div v-if="coffee.image">
          <img :src="BASE_URL + coffee.image" width="200">
        </div>
      </div>

      <!-- ⭐ CKEditor -->
      <div class="mb-3">
        <label>รายละเอียดสินค้า:</label>
        <ckeditor :editor="editor" v-model="coffee.content" :config="editorConfig"></ckeditor>
      </div>

      <button type="submit">Create Coffee</button>

    </form>
  </div>
</template>

<script>
import CoffeeService from '../../services/CoffeeService'
import UploadService from '../../services/UploadService'
import UploadImage from '../Utils/Upload.vue'
import ClassicEditor from '@ckeditor/ckeditor5-build-classic'

export default {

  components: { UploadImage },

  data() {
    return {
      editor: ClassicEditor,

      // ⭐ CKEditor ต้องมี uploadUrl
      editorConfig: {
        ckfinder: {
          uploadUrl: "http://localhost:8081/coffee-upload-image"
        }
      },

      coffee: {
        name: '',
        price: 0,
        type: 'ร้อน',
        status: 'มีจำหน่าย',
        image: '',
        gallery: '',
        content: ''
      },

      pictures: [],
      pictureIndex: 0,

      BASE_URL: 'http://localhost:8081/assets/uploads/'
    }
  },

  methods: {

    onUploaded(fileData) {

      // ⭐ ตั้งเป็นรูปหลักทันที
      this.coffee.image = fileData.filename

      const picture = {
        id: this.pictureIndex,
        name: fileData.filename
      }

      this.pictures.push(picture)
    },

    // ตั้งเป็นรูปหลัก
    useThumbnail(filename) {
      this.coffee.image = filename
    },

    // ลบรูป
    async delFile(picture) {
      if (!confirm("Delete image?")) return

      await UploadService.delete(picture.name)
      this.pictures = this.pictures.filter(p => p.id !== picture.id)
    },

    // บันทึก coffee
    async createCoffee() {
      try {

        if (!this.coffee.image) {
          alert("กรุณาอัปโหลดรูปก่อน")
          return
        }

        this.coffee.gallery = JSON.stringify(this.pictures.map(p => p.name))

        await CoffeeService.post(this.coffee)

        alert("เพิ่มเมนูสำเร็จ")
        this.$router.push({ name: 'coffees' })

      } catch (err) {
        console.error(err)
        alert("บันทึกไม่สำเร็จ (ดู console)")
      }
    }
  }
}
</script>

<style scoped>
.pictures {
  display: flex;
  flex-wrap: wrap;
}

.pictures li {
  list-style: none;
  margin: 10px;
  border: 1px solid #ddd;
  padding: 10px;
}

.pictures img {
  width: 160px;
  height: auto;
}
</style>
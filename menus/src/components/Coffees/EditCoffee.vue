<template>
  <div class="container">
    <h1>Edit Coffee</h1>

    <form v-if="loaded" @submit.prevent="updateCoffee">

      <!-- NAME -->
      <p>
        <label>Name</label><br>
        <input v-model="coffee.name" required>
      </p>

      <!-- PRICE -->
      <p>
        <label>Price</label><br>
        <input v-model.number="coffee.price" type="number" required>
      </p>

      <!-- TYPE -->
      <p>
        <label>Type</label><br>
        <select v-model="coffee.type">
          <option value="ร้อน">ร้อน</option>
          <option value="เย็น">เย็น</option>
          <option value="ปั่น">ปั่น</option>
        </select>
      </p>

      <!-- STATUS -->
      <p>
        <label>Status</label><br>
        <select v-model="coffee.status">
          <option value="มีจำหน่าย">มีจำหน่าย</option>
          <option value="หมด">หมด</option>
        </select>
      </p>

      <!-- ⭐ Upload new image -->
      <p>
        <label>เปลี่ยนรูปสินค้า</label><br>
        <Upload @selected="onFile" />
      </p>

      <!-- ⭐ Current Thumbnail -->
      <div v-if="coffee.image">
        <h3>รูปปัจจุบัน</h3>
        <img :src="BASE_URL + coffee.image" width="220">
      </div>

      <!-- ⭐ CKEditor -->
      <p>
        <label>รายละเอียดสินค้า</label>
        <ckeditor :editor="editor" v-model="coffee.content" :config="editorConfig">
        </ckeditor>
      </p>

      <br>
      <button type="submit">บันทึก</button>
      <button type="button" @click="navigateTo('/coffees')">ยกเลิก</button>

    </form>

    <p v-else>Loading...</p>
  </div>
</template>

<script>
import CoffeeService from "../../services/CoffeeService"
import Upload from "../Utils/Upload.vue"
import ClassicEditor from "@ckeditor/ckeditor5-build-classic"

export default {

  components: { Upload },

  data() {
    return {
      coffee: {},
      file: null,
      loaded: false,

      editor: ClassicEditor,

      editorConfig: {
        ckfinder: {
          uploadUrl: "http://localhost:8081/coffee-upload-image"
        }
      },

      BASE_URL: "http://localhost:8081/assets/uploads/"
    }
  },

  // โหลดข้อมูลเดิม
  async created() {
    try {
      const res = await CoffeeService.show(this.$route.params.coffeeId)

      // รองรับทั้ง res.data และ res.data.data
      this.coffee = res.data.data || res.data

      this.loaded = true

    } catch (err) {
      alert("ไม่พบเมนู")
      this.$router.push({ name: "coffees" })
    }
  },

  methods: {

    navigateTo(route) {
      this.$router.push(route)
    },

    // รับไฟล์จาก Upload.vue
    onFile(file) {
      this.file = file
    },

    // ⭐ บันทึกแก้ไข
    async updateCoffee() {
      try {

        const formData = new FormData()

        formData.append("name", this.coffee.name)
        formData.append("price", this.coffee.price)
        formData.append("type", this.coffee.type)
        formData.append("status", this.coffee.status)
        formData.append("content", this.coffee.content || "")

        // ⭐ อัปโหลดรูปใหม่
        if (this.file) {
          formData.append("image", this.file)
        }

        await CoffeeService.put(this.coffee.id, formData)

        alert("แก้ไขสำเร็จ")
        this.$router.push({ name: "coffees" })

      } catch (err) {
        console.error(err)
        alert("แก้ไขไม่สำเร็จ")
      }
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 700px;
  margin: auto;
}

img {
  border-radius: 10px;
  border: 1px solid #ddd;
  margin-top: 10px;
}

button {
  margin-right: 8px;
  padding: 8px 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
</style>
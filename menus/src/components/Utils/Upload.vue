<template>
  <div>
    <input type="file" accept="image/*" @change="onFileChange">
  </div>
</template>

<script>
import axios from "axios"

export default {
  emits: ['uploaded'],

  methods: {
    async onFileChange(e){
      const file = e.target.files[0]
      if(!file) return

      const formData = new FormData()
      formData.append("image", file)

      try{
        const res = await axios.post(
          "http://localhost:8081/upload-image",
          formData,
          {
            headers: { "Content-Type":"multipart/form-data" }
          }
        )

        // ส่ง filename กลับไป CreateCoffee.vue
        this.$emit("uploaded", res.data)

      }catch(err){
        console.error(err.response?.data || err)
        alert("Upload failed")
      }
    }
  }
}
</script>
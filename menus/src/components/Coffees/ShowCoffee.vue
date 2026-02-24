<template>
  <div>
    <h1>Show Coffee</h1>

    <div v-if="coffee" class="card">

      <!-- ⭐ รูปใหญ่ -->
      <div class="main-image">
        <img v-if="mainImage" :src="BASE_URL + mainImage">
      </div>

      <hr>

      <!-- ข้อมูล -->
      <p><b>ID</b>: {{ coffee.id }}</p>
      <p><b>Name</b>: {{ coffee.name }}</p>
      <p><b>Type</b>: {{ coffee.type }}</p>
      <p><b>Price</b>: {{ coffee.price }} บาท</p>
      <p><b>Status</b>: {{ coffee.status }}</p>

      <!-- ⭐ เนื้อหาจาก CKEditor -->
      <p><b>รายละเอียดสินค้า</b>:{{ coffee.content }}</p>

      <p>
        <button @click="navigateTo('/coffees')">กลับ</button>
      </p>

    </div>

    <p v-else>Loading...</p>
  </div>
</template>

<script>
import CoffeeService from "../../services/CoffeeService";

export default {

  data() {
    return {
      coffee: null,
      mainImage: null,
      BASE_URL: "http://localhost:8081/assets/uploads/"
    }
  },

  methods: {

    navigateTo(route) {
      this.$router.push(route)
    },

    changeImage(img) {
      this.mainImage = img
    }
  },

  async created() {
    try {
      const res = await CoffeeService.show(this.$route.params.coffeeId)
      this.coffee = res.data

      // รูปหลัก
      this.mainImage = this.coffee.image

      // gallery
      if (this.coffee.gallery) {
        this.gallery = JSON.parse(this.coffee.gallery)
      }

    } catch (err) {
      console.log(err)
      alert("Not found")
      this.$router.push({ name: "coffees" })
    }
  }
}
</script>

<style scoped>
/* รูปใหญ่ */
.main-image {
  text-align: center;
  margin-bottom: 20px;
}

.main-image img {
  width: 420px;
  max-width: 100%;
  border-radius: 14px;
  border: 1px solid #ddd;
}

/* รูปเล็ก */
.thumbs {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.thumbs img {
  width: 90px;
  height: 90px;
  object-fit: cover;
  border-radius: 10px;
  cursor: pointer;
  border: 2px solid transparent;
  transition: 0.2s;
}

.thumbs img:hover {
  transform: scale(1.05);
}

.thumbs img.active {
  border: 2px solid #6b4f3f;
}

/* เนื้อหา CKEditor */
.content {
  margin-top: 20px;
  line-height: 1.7;
}

.content img {
  max-width: 100%;
}
</style>
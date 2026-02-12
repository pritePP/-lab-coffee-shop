<template>
  <div>
    <h1>Get All Menus</h1>
    <p><button v-on:click="logout">Logout</button></p>
    <h4>จำนวน blog {{ menus.length }}</h4>
      <button v-if="isAdminLoggedIn" @click="navigateTo('/menu/create')">สร้างเมนูใหม่</button>
      <button v-else @click="navigateTo('/login')">Login (Admin)</button>
      <button v-if="isAdminLoggedIn" @click="onLogout">Logout</button>
    

    <div v-if="menus.length">
      <h4>จำนวนเมนู {{ menus.length }}</h4>

      <div v-for="menu in menus" :key="menu.id" class="card">
        <p><b>ID</b> : {{ menu.id }}</p>
        <p><b>Name</b>: {{ menu.name }}</p>
        <p><b>Type</b>: {{ menu.type }}</p>
        <p><b>Origin</b>: {{ menu.origin }}</p>
        <p><b>Price</b>: {{ menu.price }}</p>
        <p><b>Description</b>: {{ menu.description }}</p>

        <p>
          <button @click="navigateTo('/menu/' + menu.id)">ดูข้อมูลเมนู</button>
          <button v-if="isAdminLoggedIn" @click="navigateTo('/menu/edit/' + menu.id)">แก้ไขข้อมูล</button>
          <button v-if="isAdminLoggedIn" @click="deleteMenu(menu.id)">ลบข้อมูล</button>
        </p>
      </div>
    </div>

    <div v-else>
      <p>ไม่มีข้อมูลเมนู</p>
    </div>
  </div>
</template>

<script>
import MenusService from '../../services/MenusService'
import { useAuthenStore } from '../../stores/authen'

export default {
  data () {
    return { menus: [] }
  },
  async created () {
    // เรียกใช้ Service เพื่อดึงข้อมูลเมื่อ Component ถูกโหลด [6]
    this.blogs = (await BlogsService.index()).data
  },
  methods: {
    logout () {
      this.$store.dispatch('setToken', null)
      this.$store.dispatch('setBlog', null)
      this.$router.push({
        name: 'login'
      })
    },
  computed: {
    isAdminLoggedIn () {
      const authenStore = useAuthenStore()
      return authenStore.isAdminLoggedIn
    }
  },
  methods: {
    navigateTo (route) {
      this.$router.push(route)
    },
    async deleteMenu (menuId) {
      if (!confirm('Want to delete?')) return
      try {
        await MenusService.delete(menuId)
        await this.refreshData()
      } catch (err) {
        console.log(err)
        alert(err?.response?.data?.error || 'Delete failed')
      }
    },
    async refreshData () {
      const res = await MenusService.index()
      this.menus = res.data.data || res.data
    },
    onLogout () {
      const authenStore = useAuthenStore()
      authenStore.logout()
      this.$router.push({ name: 'login' })
    }
  },
  async created () {
    await this.refreshData()
  }
  
}
}
</script>

<style scoped>
/* ใช้ .card จาก global style.css */
</style>

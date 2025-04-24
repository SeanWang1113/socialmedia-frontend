<template>
  <div class="login-container">
    <h1 class="text-2xl font-bold mb-4 text-center">登入</h1>

    <form @submit.prevent="handleLogin" class="max-w-sm mx-auto">
      <input
        v-model="phoneNumber"
        type="text"
        placeholder="請輸入手機號碼"
        class="w-full border p-2 rounded mb-2"
      />
      <button
        type="submit"
        class="w-full bg-blue-600 text-white py-2 px-4 rounded hover:bg-blue-700"
      >
        登入
      </button>
    </form>

    <p v-if="errorMessage" class="text-red-500 text-center mt-2">
      {{ errorMessage }}
    </p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

const phoneNumber = ref('')
const errorMessage = ref('')
const router = useRouter()

const handleLogin = async () => {
  try {
    const res = await axios.post('http://localhost:8080/api/auth/login', null, {
      params: { phoneNumber: phoneNumber.value }
    })

    const token = res.data.replace('登入成功，token: ', '')
    localStorage.setItem('token', token)

    router.push('/')
  } catch (err) {
    errorMessage.value = '登入失敗，請確認手機號碼是否正確'
    console.error('登入錯誤：', err)
  }
}
</script>

<style scoped>
.login-container {
  padding-top: 100px;
}
</style>

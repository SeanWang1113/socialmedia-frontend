<template>
  <div class="max-w-2xl mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">貼文列表</h1>

    <!-- 發文表單 -->
    <form @submit.prevent="submitPost" class="mb-4">
      <textarea v-model="newContent" placeholder="寫點什麼吧..." class="w-full border p-2 rounded mb-2"></textarea>
      <button class="bg-blue-600 text-white px-4 py-2 rounded">發文</button>
    </form>

    <!-- 顯示貼文 -->
    <div v-if="posts.length === 0">目前沒有貼文</div>

    <div
      v-for="post in posts"
      :key="post.postId"
      class="border p-4 rounded mb-4 bg-white"
    >
      <p class="text-gray-800">{{ post.content }}</p>
      <p class="text-sm text-gray-500">
        使用者 ID：{{ post.userId }}｜時間：{{ formatDate(post.createdAt) }}
      </p>

      <!-- 留言表單 -->
      <div class="mt-2">
        <input
          v-model="newComments[post.postId]"
          placeholder="留言..."
          class="w-full border px-2 py-1 rounded mb-2"
        />
        <button
          @click="submitComment(post.postId)"
          class="bg-green-600 text-white px-3 py-1 rounded"
        >
          送出留言
        </button>
      </div>

      <!-- 留言列表 -->
      <div v-if="commentsMap[post.postId]">
        <div
          v-for="comment in commentsMap[post.postId]"
          :key="comment.commentId"
          class="text-sm text-gray-700 border-t pt-1 mt-1"
        >
          {{ comment.content }} - 使用者 {{ comment.userId }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const posts = ref([])
const newContent = ref('')
const commentsMap = ref({})
const newComments = ref({})

const fetchPosts = async () => {
  const res = await axios.get('http://localhost:8080/api/auth/post/all')
  posts.value = res.data
  posts.value.forEach(post => fetchComments(post.postId))
}

const fetchComments = async (postId) => {
  try {
    const res = await axios.get(`http://localhost:8080/api/auth/comment/by-post/${postId}`)
    commentsMap.value[postId] = res.data
  } catch (err) {
    console.error('載入留言失敗', err)
  }
}

const submitPost = async () => {
  try {
    const token = localStorage.getItem('token')
    await axios.post('http://localhost:8080/api/auth/post/create', null, {
      params: { content: newContent.value },
      headers: {
        Authorization: `Bearer ${encodeURIComponent(token)}`
      }
    })
    newContent.value = ''
    await fetchPosts()
  } catch (err) {
    console.error('發文失敗', err)
  }
}

const submitComment = async (postId) => {
  const token = localStorage.getItem('token')
  try {
    await axios.post('http://localhost:8080/api/auth/comment/create', null, {
      params: {
        postId,
        content: newComments.value[postId] || ''
      },
      headers: {
        Authorization: `Bearer ${encodeURIComponent(token)}`
      }
    })
    newComments.value[postId] = ''
    await fetchComments(postId)
  } catch (err) {
    console.error('留言失敗', err)
  }
}

const formatDate = (dateString) => {
  const d = new Date(dateString)
  return d.toLocaleString()
}

onMounted(fetchPosts)
</script>

<style scoped>
body {
  background-color: #f9f9f9;
}
</style>

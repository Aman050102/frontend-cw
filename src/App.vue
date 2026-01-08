<script>
  window.__ENV__ = {
    // ตรวจสอบว่าไม่มี / ปิดท้าย URL
    API_BASE: "https://backend-cw.aman02012548.workers.dev"
  }
</script>

<script setup>
import { ref, reactive, onMounted } from 'vue'

// จัดการ API_BASE ให้สะอาด
const RAW_BASE = window.__ENV__?.API_BASE || ''
const API_BASE = RAW_BASE.replace(/\/$/, '')

const items = ref([])
const showList = ref(true)
const loading = ref(false)
const message = ref('')

const searchId = ref('')
const searchResult = ref(null)
const newItem = reactive({ title: '', description: '' })
const updateId = ref('')
const updateField = ref('title')
const updateValue = ref('')
const deleteId = ref('')

// Helper สำหรับจัดการ Fetch เพื่อป้องกัน JSON Parse Error จาก HTML
const safeFetch = async (url, options = {}) => {
  const res = await fetch(url, options)
  if (!res.ok) {
    const errorData = await res.json().catch(() => ({}))
    throw new Error(errorData.error || `Server error: ${res.status}`)
  }
  if (res.status === 204) return null
  return res.json()
}

const fetchAll = async () => {
  loading.value = true
  message.value = ''
  try {
    items.value = await safeFetch(`${API_BASE}/items`)
  } catch (e) {
    message.value = 'Fetch error: ' + e.message
    items.value = []
  } finally {
    loading.value = false
  }
}

const fetchById = async () => {
  if (!searchId.value) return (message.value = 'Enter id')
  try {
    searchResult.value = await safeFetch(`${API_BASE}/items/${encodeURIComponent(searchId.value)}`)
    message.value = ''
  } catch (e) {
    searchResult.value = null
    message.value = e.message
  }
}

const createItem = async () => {
  if (!newItem.title) return (message.value = 'Title required')
  try {
    await safeFetch(`${API_BASE}/items`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newItem),
    })
    message.value = 'Created successfully'
    newItem.title = ''; newItem.description = ''
    await fetchAll()
  } catch (e) {
    message.value = 'Create error: ' + e.message
  }
}

const updateItem = async () => {
  if (!updateId.value || !updateValue.value) return (message.value = 'ID and Value required')
  try {
    const body = { [updateField.value]: updateValue.value }
    await safeFetch(`${API_BASE}/items/${encodeURIComponent(updateId.value)}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
    })
    message.value = `Updated ${updateId.value} successfully`
    updateValue.value = ''
    await fetchAll()
  } catch (e) {
    message.value = 'Update error: ' + e.message
  }
}

const deleteItem = async () => {
  if (!deleteId.value) return (message.value = 'Enter id')
  try {
    await safeFetch(`${API_BASE}/items/${encodeURIComponent(deleteId.value)}`, { method: 'DELETE' })
    message.value = 'Deleted successfully'
    deleteId.value = ''
    await fetchAll()
  } catch (e) {
    message.value = 'Delete error: ' + e.message
  }
}

onMounted(fetchAll)
</script>

<template>
  <div class="container">
    <h1>CRUD Playground for 67023086</h1>

    <section class="card">
      <h2 @click="showList = !showList" style="cursor:pointer">List items <small>click to toggle</small></h2>
      <div v-if="showList">
        <div class="controls">
          <button @click="fetchAll" :disabled="loading">Fetch</button>
          <span v-if="loading">Loading...</span>
        </div>
        <table class="items">
          <thead>
            <tr><th>id</th><th>title</th><th>description</th><th>created_at</th><th>updated_at</th></tr>
          </thead>
          <tbody>
            <tr v-for="it in items" :key="it.id">
              <td>{{ it.id }}</td>
              <td>{{ it.title }}</td>
              <td>{{ it.description }}</td>
              <td>{{ it.created_at }}</td>
              <td>{{ it.updated_at }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <section class="card">
      <h2>Search by ID</h2>
      <div class="row">
        <input v-model="searchId" placeholder="id" />
        <button @click="fetchById">Search</button>
      </div>
      <pre v-if="searchResult">{{ JSON.stringify(searchResult, null, 2) }}</pre>
    </section>

    <section class="card">
      <h2>Create item</h2>
      <div class="row">
        <input v-model="newItem.title" placeholder="title" />
        <input v-model="newItem.description" placeholder="description" />
        <button @click="createItem">Create</button>
      </div>
    </section>

    <section class="card">
      <h2>Update item</h2>
      <div class="row">
        <input v-model="updateId" placeholder="id" />
        <select v-model="updateField">
          <option value="title">title</option>
          <option value="description">description</option>
        </select>
        <input v-model="updateValue" placeholder="new value" />
        <button @click="updateItem">Update</button>
      </div>
    </section>

    <section class="card">
      <h2>Delete item</h2>
      <div class="row">
        <input v-model="deleteId" placeholder="id" />
        <button @click="deleteItem">Delete</button>
      </div>
    </section>

    <div class="status" v-if="message">{{ message }}</div>
  </div>
</template>

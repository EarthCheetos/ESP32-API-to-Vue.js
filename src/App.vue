<script setup>
import { ref, onUnmounted } from 'vue'

const moisture = ref(0)
const status = ref('Waiting...')
const loading = ref(false)
const error = ref(null)
const espIp = ref('') 
const isConnected = ref(false)
let timer = null

const fetchMoistureData = async () => {
  if (!espIp.value) return
  
  loading.value = true
  error.value = null

  try {
    const response = await fetch(`http://${espIp.value}/api/moisture`)
    
    if (!response.ok) throw new Error('Failed to connect')
    
    const data = await response.json()
    
    moisture.value = data.moisture
    status.value = data.status
    isConnected.value = true
    
  } catch (err) {
    console.error(err)
    error.value = "Cannot reach ESP32. Check IP or WiFi."
    isConnected.value = false
  } finally {
    loading.value = false
  }
}
const startMonitoring = () => {
  fetchMoistureData()
  if (timer) clearInterval(timer)
  timer = setInterval(fetchMoistureData, 3000)
}

const stopMonitoring = () => {
  if (timer) clearInterval(timer)
  isConnected.value = false
  status.value = "Stopped"
}

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div class="container">
    <div class="card">
      <h1>📚 Bookstore Archive</h1>
      
      <div class="controls">
        <input 
          v-model="espIp" 
          placeholder="Enter ESP32 IP (e.g., 192.168.1.15)" 
          class="ip-input"
        />
        <button v-if="!isConnected" @click="startMonitoring" class="btn start">
          Start Monitor
        </button>
        <button v-else @click="stopMonitoring" class="btn stop">
          Stop
        </button>
      </div>

      <div v-if="error" class="error-msg">
        {{ error }}
      </div>
      <div class="display-area">
        <div class="metric">
          <span class="label">Moisture Level</span>
          <span class="value">{{ moisture }}%</span>
        </div>
        
        <div class="metric">
          <span class="label">Status</span>
          <span :class="['status-badge', status === 'Optimal' ? 'good' : 'bad']">
            {{ status }}
          </span>
        </div>
      </div>
      
      <p class="loading-text" v-if="loading">Updating...</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f4f4f9;
}

.card {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  width: 100%;
  max-width: 400px;
  text-align: center;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.ip-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  color: white;
}

.start { background-color: #4CAF50; }
.stop { background-color: #f44336; }

.display-area {
  margin-top: 20px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.metric {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  font-size: 1.2rem;
}

.value {
  font-weight: bold;
  font-size: 1.5rem;
  color: #2c3e50;
}

.status-badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: bold;
}

.good { background-color: #e8f5e9; color: #2e7d32; }
.bad { background-color: #ffebee; color: #c62828; }
.error-msg { color: red; margin: 10px 0; font-size: 0.9rem; }
.loading-text { font-size: 0.8rem; color: #888; margin-top: 10px; }
</style>
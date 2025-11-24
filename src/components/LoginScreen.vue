<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['login'])
const password = ref('')
const currentTime = ref('')
const currentDate = ref('')
const isLangMenuOpen = ref(false)
const selectedLang = ref('ABC')
let timer = null

const updateTime = () => {
  const now = new Date()
  currentDate.value = now.toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric' })
  currentTime.value = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: false })
}

const handleLogin = () => {
  emit('login')
}

const toggleLangMenu = () => {
  isLangMenuOpen.value = !isLangMenuOpen.value
}

const selectLang = (lang) => {
  selectedLang.value = lang
  isLangMenuOpen.value = false
}

onMounted(() => {
  updateTime()
  timer = setInterval(updateTime, 1000)
  
  // Close menu when clicking outside
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.lang-container')) {
      isLangMenuOpen.value = false
    }
  })
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div class="login-screen">
    <!-- Top Bar -->
    <div class="top-bar">
      <div class="status-right">
        <div class="lang-container">
          <span class="lang-icon" @click="toggleLangMenu">{{ selectedLang }}</span>
          <div v-if="isLangMenuOpen" class="lang-dropdown">
            <div class="lang-option" @click="selectLang('ABC')">ABC</div>
            <div class="lang-option" @click="selectLang('English')">English</div>
            <div class="lang-option" @click="selectLang('Serbian')">Serbian</div>
          </div>
        </div>
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg>
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="7" width="16" height="10" rx="2" ry="2"></rect><line x1="22" y1="11" x2="22" y2="13"></line></svg>
        <span class="battery-text">53%</span>
      </div>
    </div>

    <!-- Clock Section -->
    <div class="clock-section">
      <div class="date">{{ currentDate }}</div>
      <div class="time">{{ currentTime }}</div>
    </div>

    <!-- Login Container -->
    <div class="login-container">
      <div class="avatar-placeholder">
        <span>NS</span>
      </div>
      <h2 class="user-name">Nemanja Samac</h2>
      
      <div class="input-group">
        <input 
          type="password" 
          v-model="password" 
          placeholder="Enter Password"
          @keyup.enter="handleLogin"
          class="password-input"
        />
        <button class="login-button" @click="handleLogin" v-if="password.length > 0">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="5" y1="12" x2="19" y2="12"></line>
            <polyline points="12 5 19 12 12 19"></polyline>
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: url('/wallpaper.jpg') no-repeat center center;
  background-size: cover;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  z-index: 100;
  color: white;
}

.top-bar {
  width: 100%;
  padding: 10px 20px;
  display: flex;
  justify-content: flex-end;
  box-sizing: border-box;
}

.status-right {
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 0.8rem;
  font-weight: 500;
}

.lang-container {
  position: relative;
}

.lang-icon {
  font-size: 0.75rem;
  border: 1px solid rgba(255,255,255,0.6);
  padding: 1px 3px;
  border-radius: 3px;
  cursor: pointer;
}

.lang-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 5px;
  background: rgba(30, 30, 30, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 6px;
  padding: 5px 0;
  min-width: 100px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  z-index: 200;
}

.lang-option {
  padding: 5px 15px;
  font-size: 0.8rem;
  color: white;
  cursor: pointer;
  transition: background 0.2s;
}

.lang-option:hover {
  background: rgba(255, 255, 255, 0.2);
}

.clock-section {
  margin-top: 4vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-shadow: 0 2px 10px rgba(0,0,0,0.3);
}

.date {
  font-size: 1.4rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  opacity: 0.9;
}

.time {
  font-size: 6rem;
  font-weight: 700;
  line-height: 1;
  letter-spacing: -2px;
}

.login-container {
  margin-top: auto;
  margin-bottom: 5vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.avatar-placeholder {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.5rem;
  font-weight: 500;
  color: white;
  backdrop-filter: blur(20px);
  margin-bottom: 1rem;
}

.user-name {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 1rem;
  text-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

.input-group {
  position: relative;
  width: 180px;
}

.password-input {
  width: 100%;
  padding: 6px 30px 6px 12px;
  border-radius: 20px;
  border: none;
  background: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  color: white;
  font-size: 0.8rem;
  outline: none;
  text-align: left;
  transition: background 0.3s;
  box-sizing: border-box;
}

.password-input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.password-input:focus {
  background: rgba(0, 0, 0, 0.4);
}

.login-button {
  position: absolute;
  right: 4px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}

.login-button:hover {
  color: white;
}

.footer-text {
  margin-top: 10px;
  font-size: 0.75rem;
  opacity: 0.6;
  font-weight: 500;
  display: none; /* Hidden as per request */
}

.footer-actions {
  position: absolute;
  bottom: 2rem;
  display: flex;
  gap: 2rem;
}

.action-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.action-btn:hover {
  opacity: 1;
}

.icon-circle {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(10px);
}

.action-btn span {
  font-size: 0.7rem;
  font-weight: 500;
}
</style>

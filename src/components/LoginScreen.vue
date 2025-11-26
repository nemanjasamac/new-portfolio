<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['login'])
const currentTime = ref('')
const currentDate = ref('')
const isLangMenuOpen = ref(false)
const selectedLang = ref('A')
const languages = [
  { id: 'A', label: 'ABC', icon: 'A' },
  { id: 'US', label: 'U.S.', icon: 'US' },
  { id: 'SR', label: 'Serbian (Latin)', icon: 'SR' },
  { id: 'CP', label: 'Serbian', icon: 'CP' },
]
let timer = null

const updateTime = () => {
  const now = new Date()
  const dateOptions = { weekday: 'short', month: 'short', day: 'numeric' }
  const timeOptions = { hour: '2-digit', minute: '2-digit', hour12: false }
  
  // Format: Mon Nov 24
  const dateStr = now.toLocaleDateString('en-US', dateOptions).replace(',', '')
  // Format: 22:51
  const timeStr = now.toLocaleTimeString('en-US', timeOptions)
  
  currentDate.value = dateStr
  currentTime.value = timeStr
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
            <div class="lang-option" v-for="lang in languages" :key="lang.id" @click="selectLang(lang.id)">
              <div class="check-col">
                <svg v-if="selectedLang === lang.id" width="10" height="10" viewBox="0 0 12 12" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 6L4.5 8.5L10 3"/></svg>
              </div>
              <div class="lang-icon-box">{{ lang.icon }}</div>
              <span class="lang-label">{{ lang.label }}</span>
            </div>
            
            <div class="menu-separator"></div>
            
            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x1="9" y1="3" x2="9" y2="21"></line></svg>
              </div>
              <span class="menu-label">Show Emoji & Symbols</span>
            </div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="4" width="20" height="16" rx="2"></rect><line x1="6" y1="8" x2="6.01" y2="8"></line><line x1="10" y1="8" x2="10.01" y2="8"></line><line x1="14" y1="8" x2="14.01" y2="8"></line><line x1="18" y1="8" x2="18.01" y2="8"></line><line x1="6" y1="12" x2="6.01" y2="12"></line><line x1="10" y1="12" x2="10.01" y2="12"></line><line x1="14" y1="12" x2="14.01" y2="12"></line><line x1="18" y1="12" x2="18.01" y2="12"></line><line x1="7" y1="16" x2="17" y2="16"></line></svg>
              </div>
              <span class="menu-label">Show Keyboard Viewer</span>
            </div>

            <div class="menu-separator"></div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon"></div>
              <span class="menu-label">Show Input Source Name</span>
            </div>

            <div class="menu-separator"></div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon"></div>
              <span class="menu-label">Open Keyboard Settings...</span>
            </div>
          </div>
        </div>
        
        <div class="battery-group">
          <span class="battery-text">100%</span>
          <svg class="icon" width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <rect x="3.25" y="7.25" width="16.5" height="9.5" rx="2.5" stroke="currentColor" stroke-width="1.5" stroke-opacity="0.4"/>
            <path d="M22 10.5V13.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-opacity="0.4"/>
            <rect x="5.5" y="9.5" width="12" height="5" rx="1" fill="currentColor"/>
          </svg>
        </div>
        <!-- Control center icon   <svg viewBox="0 0 29 29" width="16" height="16" xmlns="http://www.w3.org/2000/svg" fill="currentColor"><path d="M7.5,13h14a5.5,5.5,0,0,0,0-11H7.5a5.5,5.5,0,0,0,0,11Zm0-9h14a3.5,3.5,0,0,1,0,7H7.5a3.5,3.5,0,0,1,0-7Zm0,6A2.5,2.5,0,1,0,5,7.5,2.5,2.5,0,0,0,7.5,10Zm14,6H7.5a5.5,5.5,0,0,0,0,11h14a5.5,5.5,0,0,0,0-11Zm1.43439,8a2.5,2.5,0,1,1,2.5-2.5A2.5,2.5,0,0,1,22.93439,24Z"></path></svg> -->
        <svg class="icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg>
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
      
      <div class="login-actions">
        <button class="login-btn" @click="handleLogin">Log In</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-screen {
  position: fixed;
  user-select: none;
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
  gap: 18px;
  font-size: 13px;
  font-weight: 500;
  -webkit-font-smoothing: antialiased;
}

.battery-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.lang-container {
  position: relative;
}

.lang-icon {
  font-size: 11px;
  border: 1.5px solid rgba(255,255,255,0.9);
  padding: 0px 3px;
  border-radius: 4px;
  cursor: var(--mac-cursor);
  font-weight: 600;
  min-width: 16px;
  text-align: center;
  display: inline-block;
}

.lang-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 8px;
  background: rgba(30, 30, 30, 0.6);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 8px;
  padding: 5px;
  min-width: 240px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 15px 30px rgba(0,0,0,0.3);
  z-index: 200;
  display: flex;
  flex-direction: column;
}

.top-time {
  font-weight: 500;
  margin-left: 4px;
}

.icon {
  opacity: 0.9;
}

.lang-option, .menu-item {
  display: flex;
  align-items: center;
  padding: 3px 10px 3px 4px;
  font-size: 13px;
  color: white;
  cursor: var(--mac-cursor);
  border-radius: 4px;
  height: 24px;
}

.lang-option:hover, .menu-item:hover {
  background: #007AFF;
}

.check-col {
  width: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 4px;
}

.lang-icon-box {
  width: 16px;
  height: 16px;
  background: #f0f0f0;
  border-radius: 3px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 9px;
  font-weight: 700;
  color: #333;
  margin-right: 8px;
  box-shadow: 0 0.5px 1px rgba(0,0,0,0.2);
}

.menu-icon {
  width: 16px;
  height: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 8px;
  color: #fff;
}

.lang-label, .menu-label {
  flex: 1;
  white-space: nowrap;
  font-weight: 400;
}

.menu-separator {
  height: 1px;
  background: rgba(255, 255, 255, 0.15);
  margin: 4px 10px;
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

.login-actions {
  margin-top: 10px;
}

.login-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  padding: 6px 20px;
  border-radius: 20px;
  color: white;
  font-size: 13px;
  font-weight: 500;
  cursor: var(--mac-cursor);
  backdrop-filter: blur(10px);
  transition: background 0.2s;
}

.login-btn:hover {
  background: rgba(255, 255, 255, 0.3);
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
  cursor: var(--mac-cursor);
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

<script setup>
import { ref, computed } from 'vue'
import safariIcon from '../assets/Icons/safari.svg'
import folderIcon from '../assets/Icons/folder.png'
import finderIcon from '../assets/Icons/finder.png'
import calculatorIcon from '../assets/Icons/calculator.svg'
import mailIcon from '../assets/Icons/mail.svg'
import settingsIcon from '../assets/Icons/apple-settings.svg'
import notesIcon from '../assets/Icons/apple-notes.svg'
import photosIcon from '../assets/Icons/apple-photos.svg'
import newsIcon from '../assets/Icons/news.svg'
import mapsIcon from '../assets/Icons/apple-maps.webp'
import musicIcon from '../assets/Icons/apple-music.svg'
import appStoreIcon from '../assets/Icons/app-store.svg'

const props = defineProps({
  isOpen: Boolean
})

const emit = defineEmits(['close', 'open-app'])

const searchQuery = ref('')

const apps = [
  { id: 'finder-app', label: 'Finder', icon: 'finder-app' },
  { id: 'finder', label: 'About Me', icon: 'finder' },
  { id: 'resume', label: 'My Resume', icon: 'resume' },
  { id: 'projects', label: 'My Projects', icon: 'folder' },
  { id: 'terminal', label: 'Terminal', icon: 'terminal' },
  { id: 'safari', label: 'Safari', icon: 'safari' },
  { id: 'mail', label: 'Contact Me', icon: 'mail' },
  { id: 'settings', label: 'Settings', icon: 'settings' },
  { id: 'calculator', label: 'Calculator', icon: 'calculator' },
  { id: 'notes', label: 'Notes', icon: 'notes' },
  { id: 'photos', label: 'Photos', icon: 'photos' },
  { id: 'maps', label: 'Maps', icon: 'maps' },
  { id: 'news', label: 'News', icon: 'news' },
  { id: 'music', label: 'Music', icon: 'music' },
  { id: 'appstore', label: 'App Store', icon: 'appstore' },
]

const filteredApps = computed(() => {
  if (!searchQuery.value) return apps
  return apps.filter(app => app.label.toLowerCase().includes(searchQuery.value.toLowerCase()))
})

const handleAppClick = (appId) => {
  emit('open-app', appId)
  emit('close')
}
</script>

<template>
  <div class="launchpad-overlay" @click.self="emit('close')">
    <div class="search-bar-container">
      <div class="search-bar">
        <svg class="search-icon" viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="11" cy="11" r="8"></circle>
          <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
        </svg>
        <input 
          type="text" 
          placeholder="Search" 
          v-model="searchQuery"
          ref="searchInput"
        />
      </div>
    </div>

    <div class="apps-grid">
      <div 
        v-for="app in filteredApps" 
        :key="app.id" 
        class="app-item"
        @click="handleAppClick(app.id)"
      >
        <div class="app-icon-container">
            <!-- Finder App -->
            <img v-if="app.icon === 'finder-app'" :src="finderIcon" class="icon-img" alt="Finder" />

            <!-- Finder / About Me -->
            <svg v-if="app.icon === 'finder'" viewBox="0 0 100 100" class="icon-svg">
                <rect x="5" y="5" width="90" height="90" rx="22" fill="#ececec"/>
                <rect x="5" y="5" width="90" height="90" rx="22" fill="url(#finder-grad-lp)" fill-opacity="0.8"/>
                <path d="M50 94c24.3 0 44-19.7 44-44S74.3 6 50 6 6 25.7 6 50s19.7 44 44 44z" fill="none"/>
                <path d="M28 35c0-5 8-8 22-8s22 3 22 8v30c0 15-44 15-44 0V35z" fill="#007AFF" opacity="0.1"/>
                <path d="M33 32c0 0 5-4 17-4s17 4 17 4" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
                <path d="M33 45c0 0 5 5 17 5s17-5 17-5" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
                <line x1="50" y1="45" x2="50" y2="58" stroke="#333" stroke-width="3" stroke-linecap="round"/>
                <defs>
                <linearGradient id="finder-grad-lp" x1="0%" y1="0%" x2="0%" y2="100%">
                    <stop offset="0%" style="stop-color:#f9f9f9;stop-opacity:1" />
                    <stop offset="100%" style="stop-color:#c7c7c7;stop-opacity:1" />
                </linearGradient>
                </defs>
            </svg>

            <!-- Resume -->
            <svg v-if="app.icon === 'resume'" viewBox="0 0 100 100" class="icon-svg" >
                <rect x="15" y="5" width="70" height="90" rx="8" fill="#fff"/>
                <path d="M15 25h70" stroke="#ddd" stroke-width="1"/>
                <rect x="25" y="35" width="50" height="4" rx="2" fill="#e0e0e0"/>
                <rect x="25" y="45" width="50" height="4" rx="2" fill="#e0e0e0"/>
                <rect x="25" y="55" width="35" height="4" rx="2" fill="#e0e0e0"/>
                <rect x="25" y="70" width="50" height="4" rx="2" fill="#e0e0e0"/>
                <rect x="25" y="80" width="40" height="4" rx="2" fill="#e0e0e0"/>
                <circle cx="70" cy="20" r="8" fill="#FF9500"/>
            </svg>

            <!-- Projects / Folder -->
            <img v-if="app.icon === 'folder'" :src="folderIcon" class="icon-img" alt="Projects" />

            <!-- Terminal -->
            <svg v-if="app.icon === 'terminal'" viewBox="0 0 100 100" class="icon-svg">
                <rect x="10" y="10" width="80" height="80" rx="15" fill="#333"/>
                <rect x="10" y="10" width="80" height="80" rx="15" fill="url(#terminal-grad-lp)" fill-opacity="0.5"/>
                <text x="20" y="45" font-family="monospace" font-size="24" fill="#27c93f" font-weight="bold">&gt;_</text>
                <defs>
                <linearGradient id="terminal-grad-lp" x1="0%" y1="0%" x2="0%" y2="100%">
                    <stop offset="0%" style="stop-color:#444;stop-opacity:1" />
                    <stop offset="100%" style="stop-color:#222;stop-opacity:1" />
                </linearGradient>
                </defs>
            </svg>

            <!-- Safari -->
            <img v-if="app.icon === 'safari'" :src="safariIcon" class="icon-img" alt="Safari" />

            <!-- Mail -->
            <img v-if="app.icon === 'mail'" :src="mailIcon" class="icon-img" alt="Mail" />

            <!-- Calculator -->
            <img v-if="app.icon === 'calculator'" :src="calculatorIcon" class="icon-img" alt="Calculator" />

            <!-- Settings -->
            <img v-if="app.icon === 'settings'" :src="settingsIcon" class="icon-img" alt="Settings" />

            <!-- Notes -->
            <img v-if="app.icon === 'notes'" :src="notesIcon" class="icon-img" alt="Notes" />

            <!-- Photos -->
            <img v-if="app.icon === 'photos'" :src="photosIcon" class="icon-img" alt="Photos" />

            <!-- News -->
            <img v-if="app.icon === 'news'" :src="newsIcon" class="icon-img" alt="News" />

            <!-- Music -->
            <img v-if="app.icon === 'music'" :src="musicIcon" class="icon-img" alt="Music" />

            <!-- Maps -->
            <img v-if="app.icon === 'maps'" :src="mapsIcon" class="icon-img rounded-icon" alt="Maps" />

            <!-- App Store -->
            <img v-if="app.icon === 'appstore'" :src="appStoreIcon" class="icon-img" alt="App Store" />

            <!-- Generic App Icon for others -->
            <div v-if="!['finder-app', 'finder', 'resume', 'folder', 'terminal', 'safari', 'mail', 'settings', 'calculator', 'notes', 'maps', 'photos', 'news', 'music', 'appstore'].includes(app.icon)" class="generic-icon" :class="app.icon">
                <div class="icon-bg"></div>
                <span class="icon-symbol">{{ app.label[0] }}</span>
            </div>
        </div>
        <div class="app-label">{{ app.label }}</div>
      </div>
    </div>
    
    <div class="pagination-dots">
        <div class="dot active"></div>
        <div class="dot"></div>
    </div>
  </div>
</template>

<style scoped>
.launchpad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 80px;
  box-sizing: border-box;
  overflow: hidden;
}

.search-bar-container {
  margin-bottom: 40px;
  width: 100%;
  display: flex;
  justify-content: center;
}

.search-bar {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  padding: 6px 12px;
  width: 250px;
}

.search-icon {
  color: rgba(255, 255, 255, 0.7);
  margin-right: 8px;
}

.search-bar input {
  background: transparent;
  border: none;
  color: white;
  font-size: 14px;
  width: 100%;
  outline: none;
}

.search-bar input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.apps-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 40px 20px;
  max-width: 1000px;
  padding: 0 40px;
}

.app-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  transition: transform 0.2s;
}

.app-item:hover {
  transform: scale(1.05);
}

.app-item:active {
  transform: scale(0.95);
  filter: brightness(0.8);
}

.app-icon-container {
  width: 80px;
  height: 80px;
  margin-bottom: 10px;
}

.icon-img, .icon-svg {
  width: 100%;
  height: 100%;
  object-fit: contain;
  filter: drop-shadow(0 4px 8px rgba(0,0,0,0.3));
}

.rounded-icon {
  border-radius: 18px;
}

.app-label {
  color: white;
  font-size: 13px;
  text-align: center;
  text-shadow: 0 1px 2px rgba(0,0,0,0.8);
  font-weight: 500;
}

.generic-icon {
    width: 100%;
    height: 100%;
    border-radius: 18px;
    position: relative;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
}

.icon-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%, #fecfef 100%);
}

.calculator .icon-bg { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
.notes .icon-bg { background: linear-gradient(135deg, #f6d365 0%, #fda085 100%); }
.photos .icon-bg { background: linear-gradient(135deg, #84fab0 0%, #8fd3f4 100%); }
.maps .icon-bg { background: linear-gradient(135deg, #a18cd1 0%, #fbc2eb 100%); }
.news .icon-bg { background: linear-gradient(135deg, #fdcbf1 0%, #e6dee9 100%); }
.music .icon-bg { background: linear-gradient(135deg, #fa709a 0%, #fee140 100%); }
.appstore .icon-bg { background: linear-gradient(135deg, #00c6fb 0%, #005bea 100%); }

.icon-symbol {
    position: relative;
    color: white;
    font-size: 40px;
    font-weight: bold;
    text-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.pagination-dots {
    position: absolute;
    bottom: 100px;
    display: flex;
    gap: 8px;
}

.dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: rgba(255,255,255,0.3);
}

.dot.active {
    background: white;
}
</style>
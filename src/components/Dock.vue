<script setup>
import { ref } from 'vue'
import safariIcon from '../assets/Icons/safari.svg'
import folderIcon from '../assets/Icons/folder.png'
import finderIcon from '../assets/Icons/finder.png'
import launchpadIcon from '../assets/Icons/launchpad.png'
import mailIcon from '../assets/Icons/mail.svg'
import notesIcon from '../assets/Icons/apple-notes.svg'
import photosIcon from '../assets/Icons/apple-photos.svg'
import newsIcon from '../assets/Icons/news.svg'
import mapsIcon from '../assets/Icons/apple-maps.webp'
import musicIcon from '../assets/Icons/apple-music.svg'
import appStoreIcon from '../assets/Icons/app-store.svg'

const dockItems = ref([
  { id: 'finder-app', label: 'Finder', icon: 'finder-app' },
  { id: 'launchpad', label: 'Launchpad', icon: 'launchpad' },
  { id: 'finder', label: 'About Me', icon: 'finder' },
  { id: 'resume', label: 'My Resume', icon: 'resume' },
  { id: 'projects', label: 'My Projects', icon: 'folder' },
  { id: 'terminal', label: 'Terminal', icon: 'terminal' },
  { id: 'safari', label: 'Safari', icon: 'safari' },
  { id: 'mail', label: 'Contact Me', icon: 'mail' },
  { id: 'notes', label: 'Notes', icon: 'notes' },
  { id: 'photos', label: 'Photos', icon: 'photos' },
  { id: 'news', label: 'News', icon: 'news' },
  { id: 'music', label: 'Music', icon: 'music' },
  { id: 'maps', label: 'Maps', icon: 'maps' },
  { id: 'appstore', label: 'App Store', icon: 'appstore' },
  { id: 'trash', label: 'Trash', icon: 'trash' },
])

const emit = defineEmits(['open-app'])

const hoveredIndex = ref(null)
</script>

<template>
  <div class="dock-container">
    <div class="dock-wrapper">
      <div 
        v-for="(item, index) in dockItems" 
        :key="item.id"
        class="dock-item"
        :class="{ 'trash-drop-zone': item.id === 'trash' }"
        @mouseenter="hoveredIndex = index"
        @mouseleave="hoveredIndex = null"
        @click="emit('open-app', item.id)"
      >
        <div class="tooltip">{{ item.label }}</div>
        <div class="app-icon">
          <!-- Finder App -->
          <img v-if="item.icon === 'finder-app'" :src="finderIcon" class="icon-img" alt="Finder" />

          <!-- Launchpad -->
          <img v-if="item.icon === 'launchpad'" :src="launchpadIcon" class="icon-img" alt="Launchpad" />

          <!-- Finder / About Me -->
          <svg v-if="item.icon === 'finder'" viewBox="0 0 100 100" class="icon-svg">
            <rect x="5" y="5" width="90" height="90" rx="22" fill="#ececec"/>
            <rect x="5" y="5" width="90" height="90" rx="22" fill="url(#finder-grad)" fill-opacity="0.8"/>
            <path d="M50 94c24.3 0 44-19.7 44-44S74.3 6 50 6 6 25.7 6 50s19.7 44 44 44z" fill="none"/>
            <path d="M28 35c0-5 8-8 22-8s22 3 22 8v30c0 15-44 15-44 0V35z" fill="#007AFF" opacity="0.1"/>
            <path d="M33 32c0 0 5-4 17-4s17 4 17 4" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <path d="M33 45c0 0 5 5 17 5s17-5 17-5" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <line x1="50" y1="45" x2="50" y2="58" stroke="#333" stroke-width="3" stroke-linecap="round"/>
            <defs>
              <linearGradient id="finder-grad" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" style="stop-color:#f9f9f9;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#c7c7c7;stop-opacity:1" />
              </linearGradient>
            </defs>
          </svg>

          <!-- Resume / Doc -->
          <svg v-if="item.icon === 'resume'" viewBox="0 0 100 100" class="icon-svg" >
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
          <img v-if="item.icon === 'folder'" :src="folderIcon" class="icon-img" alt="Projects" />

          <!-- Terminal -->
          <svg v-if="item.icon === 'terminal'" viewBox="0 0 100 100" class="icon-svg">
            <rect x="10" y="10" width="80" height="80" rx="15" fill="#333"/>
            <rect x="10" y="10" width="80" height="80" rx="15" fill="url(#terminal-grad)" fill-opacity="0.5"/>
            <text x="20" y="45" font-family="monospace" font-size="24" fill="#27c93f" font-weight="bold">&gt;_</text>
            <defs>
              <linearGradient id="terminal-grad" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" style="stop-color:#444;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#222;stop-opacity:1" />
              </linearGradient>
            </defs>
          </svg>

          <!-- Safari -->
          <img v-if="item.icon === 'safari'" :src="safariIcon" class="icon-img" alt="Safari" />

          <!-- Mail -->
          <img v-if="item.icon === 'mail'" :src="mailIcon" class="icon-img" alt="Mail" />

          <!-- Notes -->
          <img v-if="item.icon === 'notes'" :src="notesIcon" class="icon-img" alt="Notes" />

          <!-- Photos -->
          <img v-if="item.icon === 'photos'" :src="photosIcon" class="icon-img" alt="Photos" />

          <!-- News -->
          <img v-if="item.icon === 'news'" :src="newsIcon" class="icon-img" alt="News" />

          <!-- Music -->
          <img v-if="item.icon === 'music'" :src="musicIcon" class="icon-img" alt="Music" />

          <!-- Maps -->
          <img v-if="item.icon === 'maps'" :src="mapsIcon" class="icon-img" alt="Maps" />

          <!-- App Store -->
          <img v-if="item.icon === 'appstore'" :src="appStoreIcon" class="icon-img" alt="App Store" />

          <!-- Trash -->
          <svg v-if="item.icon === 'trash'" viewBox="0 0 100 100" class="icon-svg trash-icon">
            <defs>
              <linearGradient id="trash-grad" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" style="stop-color:#e6e6e6;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#b0b0b0;stop-opacity:1" />
              </linearGradient>
            </defs>
            <path d="M20 25 L80 25 L75 90 L25 90 Z" fill="url(#trash-grad)" stroke="#999" stroke-width="1"/>
            <path d="M25 25 L28 15 L72 15 L75 25" fill="#ccc" stroke="#999" stroke-width="1"/>
            <line x1="35" y1="35" x2="35" y2="80" stroke="#999" stroke-width="2"/>
            <line x1="50" y1="35" x2="50" y2="80" stroke="#999" stroke-width="2"/>
            <line x1="65" y1="35" x2="65" y2="80" stroke="#999" stroke-width="2"/>
          </svg>
        </div>
        <div class="dot"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dock-container {
  position: fixed;
  bottom: 20px;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: center;
  z-index: 9000;
  pointer-events: none; /* Let clicks pass through container area */
}

.dock-wrapper {
  display: flex;
  align-items: flex-end;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  padding: 12px;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  gap: 12px;
  pointer-events: auto; /* Re-enable clicks on the dock itself */
  transition: all 0.3s ease;
}

.dock-item {
  position: relative;
  width: 50px;
  height: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  transition: all 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  cursor: var(--mac-cursor);
}

.dock-item:hover {
  transform: translateY(-10px) scale(1.2);
  margin: 0 10px;
}

.app-icon {
  width: 100%;
  height: 100%;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  background: white;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-svg {
  width: 100%;
  height: 100%;
}

.icon-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.tooltip {
  position: absolute;
  top: -40px;
  background: rgba(30, 30, 30, 0.8);
  backdrop-filter: blur(10px);
  color: white;
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 12px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
  white-space: nowrap;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.dock-item:hover .tooltip {
  opacity: 1;
}

.dot {
  width: 4px;
  height: 4px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  margin-top: 4px;
  position: absolute;
  bottom: -8px;
}
</style>
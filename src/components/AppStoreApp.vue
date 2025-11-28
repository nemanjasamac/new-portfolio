<script setup>
import { ref, computed } from 'vue'
import xcodeIcon from '../assets/Icons/xcode.png'
import finalCutIcon from '../assets/Icons/final-cut.png'
import logicProIcon from '../assets/Icons/logic-pro.png'
import minecraftIcon from '../assets/Icons/minecraft.svg'
import discordIcon from '../assets/Icons/discord.png'
import zoomIcon from '../assets/Icons/zoom.png'

const props = defineProps({
  isOpen: Boolean,
  zIndex: {
    type: Number,
    default: 100
  }
})

const emit = defineEmits(['close', 'focus'])

const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: 100, y: 50 })
const isMaximized = ref(false)
const preMaximizePosition = ref({ x: 0, y: 0 })
const selectedTab = ref('discover')
const searchQuery = ref('')
const installingApps = ref({})

const sidebarItems = [
  { id: 'discover', label: 'Discover', icon: 'star' },
  { id: 'arcade', label: 'Arcade', icon: 'game-controller' },
  { id: 'create', label: 'Create', icon: 'brush' },
  { id: 'work', label: 'Work', icon: 'briefcase' },
  { id: 'play', label: 'Play', icon: 'play' },
  { id: 'develop', label: 'Develop', icon: 'code' },
  { id: 'categories', label: 'Categories', icon: 'grid' },
  { id: 'updates', label: 'Updates', icon: 'download' },
]

const allApps = [
  { id: 1, title: 'VS Code', category: 'Developer Tools', image: 'https://upload.wikimedia.org/wikipedia/commons/9/9a/Visual_Studio_Code_1.35_icon.svg', description: 'Code editing. Redefined.', tags: ['develop', 'work'] },
  { id: 2, title: 'Slack', category: 'Business', image: 'https://upload.wikimedia.org/wikipedia/commons/d/d5/Slack_icon_2019.svg', description: 'Where work happens.', tags: ['work', 'play'] },
  { id: 3, title: 'Discord', category: 'Social Networking', image: discordIcon, description: 'Talk, chat, hang out.', tags: ['play'] },
  { id: 4, title: 'Xcode', category: 'Developer Tools', icon: xcodeIcon, tags: ['develop', 'work'] },
  { id: 5, title: 'Final Cut Pro', category: 'Video', icon: finalCutIcon, tags: ['create'] },
  { id: 6, title: 'Logic Pro', category: 'Music', icon: logicProIcon, tags: ['create'] },
  { id: 7, title: 'Sketch', category: 'Graphics', icon: 'https://upload.wikimedia.org/wikipedia/commons/5/59/Sketch_Logo.svg', tags: ['create'] },
  { id: 8, title: 'Figma', category: 'Graphics', icon: 'https://upload.wikimedia.org/wikipedia/commons/3/33/Figma-logo.svg', tags: ['create', 'work'] },
  { id: 9, title: 'Minecraft', category: 'Games', icon: minecraftIcon, tags: ['arcade', 'play'] },
  { id: 10, title: 'Among Us', category: 'Games', icon: 'https://upload.wikimedia.org/wikipedia/en/9/9a/Among_Us_cover_art.jpg', tags: ['arcade', 'play'] },
  { id: 11, title: 'Zoom', category: 'Business', icon: zoomIcon, tags: ['work'] },
  { id: 12, title: 'Spotify', category: 'Music', icon: 'https://upload.wikimedia.org/wikipedia/commons/1/19/Spotify_logo_without_text.svg', tags: ['play'] },
]

const featuredApps = computed(() => allApps.filter(app => [1, 2, 3].includes(app.id)))
const popularApps = computed(() => allApps.filter(app => [4, 5, 6, 7, 8].includes(app.id)))

const filteredApps = computed(() => {
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    return allApps.filter(app => 
      app.title.toLowerCase().includes(query) || 
      app.category.toLowerCase().includes(query)
    )
  }
  
  if (selectedTab.value === 'discover') return [] // Handled separately in template
  
  return allApps.filter(app => app.tags && app.tags.includes(selectedTab.value))
})

const handleInstall = (appId) => {
  if (installingApps.value[appId] === 'OPEN') return
  
  installingApps.value[appId] = 'LOADING'
  setTimeout(() => {
    installingApps.value[appId] = 'OPEN'
  }, 2000)
}

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.sidebar') || e.target.closest('.app-card-btn')) return
  
  isDragging.value = true
  dragOffset.value = {
    x: e.clientX - position.value.x,
    y: e.clientY - position.value.y
  }
  
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
}

const handleMouseMove = (e) => {
  if (!isDragging.value) return
  if (isMaximized.value) return
  position.value = {
    x: e.clientX - dragOffset.value.x,
    y: e.clientY - dragOffset.value.y
  }
}

const handleMouseUp = () => {
  isDragging.value = false
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('mouseup', handleMouseUp)
}

const toggleMaximize = () => {
  if (isMaximized.value) {
    position.value = { ...preMaximizePosition.value }
    isMaximized.value = false
  } else {
    preMaximizePosition.value = { ...position.value }
    position.value = { x: 0, y: 0 }
    isMaximized.value = true
  }
}
</script>

<template>
  <div 
    v-if="isOpen"
    class="app-store-window"
    :class="{ maximized: isMaximized }"
    :style="{ 
      top: position.y + 'px', 
      left: position.x + 'px',
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <div class="sidebar">
      <div class="window-controls">
        <button class="control-btn close" @click="$emit('close')"></button>
        <button class="control-btn minimize"></button>
        <button class="control-btn maximize" @click="toggleMaximize"></button>
      </div>
      
      <div class="search-container">
        <div class="search-input-wrapper">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
            <input type="text" placeholder="Search" v-model="searchQuery" />
        </div>
      </div>

      <div class="nav-items">
        <div 
          v-for="item in sidebarItems" 
          :key="item.id"
          class="nav-item"
          :class="{ active: selectedTab === item.id }"
          @click="selectedTab = item.id"
        >
          <span class="nav-icon">
            <!-- Simple SVG icons based on id -->
            <svg v-if="item.icon === 'star'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>
            <svg v-if="item.icon === 'game-controller'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="6" width="20" height="12" rx="2"/><path d="M6 12h4m-2-2v4m10-2h2m-2 0h2"/></svg>
            <svg v-if="item.icon === 'brush'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 19l7-7 3 3-7 7-3-3z"/><path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"/><path d="M2 2l7.586 7.586"/><circle cx="11" cy="11" r="2"/></svg>
            <svg v-if="item.icon === 'briefcase'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="7" width="20" height="14" rx="2" ry="2"/><path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"/></svg>
            <svg v-if="item.icon === 'play'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>
            <svg v-if="item.icon === 'code'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="16 18 22 12 16 6"/><polyline points="8 6 2 12 8 18"/></svg>
            <svg v-if="item.icon === 'grid'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
            <svg v-if="item.icon === 'download'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
          </span>
          {{ item.label }}
        </div>
      </div>
      
      <div class="user-profile">
        <div class="avatar">NS</div>
        <div class="user-name">Nemanja Samac</div>
      </div>
    </div>

    <div class="main-content">
      <div v-if="selectedTab === 'discover' && !searchQuery" class="content-scroll">
        <div class="hero-section">
            <div class="hero-card">
                <div class="hero-content">
                    <span class="hero-tag">FEATURED</span>
                    <h1>Design Your World</h1>
                    <p>Create stunning visuals with ease.</p>
                </div>
                <div class="hero-image"></div>
            </div>
        </div>

        <div class="section-header">
            <h2>Popular Apps</h2>
            <a href="#">See All</a>
        </div>

        <div class="apps-grid">
            <div v-for="app in popularApps" :key="app.id" class="app-item">
                <img :src="app.icon || app.image" class="app-icon" />
                <div class="app-details">
                    <div class="app-title">{{ app.title }}</div>
                    <div class="app-category">{{ app.category }}</div>
                </div>
                <button 
                  class="get-btn" 
                  :class="{ 'loading': installingApps[app.id] === 'LOADING', 'open': installingApps[app.id] === 'OPEN' }"
                  @click="handleInstall(app.id)"
                >
                  {{ installingApps[app.id] === 'LOADING' ? '' : (installingApps[app.id] === 'OPEN' ? 'OPEN' : 'GET') }}
                  <div v-if="installingApps[app.id] === 'LOADING'" class="spinner"></div>
                </button>
            </div>
        </div>

        <div class="section-header">
            <h2>Essential Tools</h2>
            <a href="#">See All</a>
        </div>
        
        <div class="featured-row">
             <div v-for="app in featuredApps" :key="app.id" class="featured-card">
                <div class="featured-top">
                    <span class="featured-category">{{ app.category }}</span>
                    <h3 class="featured-title">{{ app.title }}</h3>
                    <p class="featured-desc">{{ app.description }}</p>
                </div>
                <div class="featured-bottom">
                    <img :src="app.image || app.icon" class="featured-img" />
                </div>
             </div>
        </div>
      </div>
      
      <div v-else-if="filteredApps.length > 0" class="content-scroll">
        <div class="section-header">
            <h2>{{ searchQuery ? 'Search Results' : selectedTab.charAt(0).toUpperCase() + selectedTab.slice(1) }}</h2>
        </div>
        <div class="apps-grid">
            <div v-for="app in filteredApps" :key="app.id" class="app-item">
                <img :src="app.icon || app.image" class="app-icon" />
                <div class="app-details">
                    <div class="app-title">{{ app.title }}</div>
                    <div class="app-category">{{ app.category }}</div>
                </div>
                <button 
                  class="get-btn" 
                  :class="{ 'loading': installingApps[app.id] === 'LOADING', 'open': installingApps[app.id] === 'OPEN' }"
                  @click="handleInstall(app.id)"
                >
                  {{ installingApps[app.id] === 'LOADING' ? '' : (installingApps[app.id] === 'OPEN' ? 'OPEN' : 'GET') }}
                  <div v-if="installingApps[app.id] === 'LOADING'" class="spinner"></div>
                </button>
            </div>
        </div>
      </div>

      <div v-else class="empty-state">
        <div class="empty-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="#ddd" stroke-width="1" width="80" height="80">
                <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path>
            </svg>
        </div>
        <h2>{{ searchQuery ? 'No Results Found' : selectedTab.charAt(0).toUpperCase() + selectedTab.slice(1) }}</h2>
        <p>{{ searchQuery ? 'Try searching for something else.' : 'Coming soon to the App Store.' }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-store-window {
  position: absolute;
  width: 950px;
  height: 650px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
  display: flex;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.app-store-window.maximized {
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  border-radius: 0;
}

.sidebar {
  width: 240px;
  background: #f5f5f7;
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  padding-top: 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
  padding: 10px 15px 15px;
}

.control-btn {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: none;
  cursor: pointer;
  padding: 0;
}

.close { background: #ff5f56; border: 1px solid #e0443e; }
.minimize { background: #ffbd2e; border: 1px solid #dea123; }
.maximize { background: #27c93f; border: 1px solid #1aab29; }

.search-container {
    padding: 0 15px 20px;
}

.search-input-wrapper {
    background: #e5e5e5;
    border-radius: 6px;
    padding: 6px 10px;
    display: flex;
    align-items: center;
    gap: 8px;
    color: #888;
}

.search-input-wrapper input {
    background: transparent;
    border: none;
    outline: none;
    font-size: 13px;
    width: 100%;
}

.nav-items {
    flex: 1;
    padding: 0 10px;
}

.nav-item {
    display: flex;
    align-items: center;
    padding: 10px 12px;
    border-radius: 6px;
    cursor: pointer;
    color: #333;
    font-size: 14px;
    margin-bottom: 2px;
}

.nav-item:hover {
    background: rgba(0,0,0,0.05);
}

.nav-item.active {
    background: #e5e5e5;
    color: #007AFF;
    font-weight: 500;
}

.nav-icon {
    margin-right: 10px;
    display: flex;
    align-items: center;
}

.nav-icon svg {
    width: 18px;
    height: 18px;
}

.user-profile {
    padding: 20px;
    display: flex;
    align-items: center;
    gap: 10px;
    border-top: 1px solid #e5e5e5;
}

.avatar {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    background: #007AFF;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    font-weight: 600;
}

.user-name {
    font-size: 13px;
    font-weight: 500;
}

.main-content {
    flex: 1;
    background: #fff;
    overflow-y: auto;
}

.content-scroll {
    padding: 30px 40px;
}

.hero-section {
    margin-bottom: 40px;
}

.hero-card {
    height: 300px;
    border-radius: 12px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    position: relative;
    overflow: hidden;
    color: white;
    padding: 40px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.15);
}

.hero-content {
    position: relative;
    z-index: 2;
    max-width: 50%;
}

.hero-tag {
    font-size: 12px;
    font-weight: 600;
    opacity: 0.8;
    margin-bottom: 10px;
    display: block;
}

.hero-content h1 {
    font-size: 36px;
    font-weight: 700;
    margin: 0 0 10px 0;
    line-height: 1.1;
}

.hero-content p {
    font-size: 16px;
    opacity: 0.9;
}

.section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    padding-top: 20px;
    border-top: 1px solid #f0f0f0;
}

.section-header:first-of-type {
    border-top: none;
    padding-top: 0;
}

.section-header h2 {
    font-size: 20px;
    font-weight: 600;
    margin: 0;
}

.section-header a {
    color: #007AFF;
    text-decoration: none;
    font-size: 13px;
}

.apps-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
    margin-bottom: 40px;
}

.app-item {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 10px;
    border-radius: 8px;
    transition: background 0.2s;
}

.app-item:hover {
    background: #f9f9f9;
}

.app-icon {
    width: 50px;
    height: 50px;
    border-radius: 12px;
    object-fit: contain;
    background: #f0f0f0;
}

.app-details {
    flex: 1;
}

.app-title {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    margin-bottom: 2px;
}

.app-category {
    font-size: 12px;
    color: #888;
}

.get-btn {
    background: #f0f0f7;
    color: #007AFF;
    border: none;
    padding: 5px 15px;
    border-radius: 15px;
    font-weight: 600;
    font-size: 12px;
    cursor: pointer;
    transition: all 0.2s;
}

.get-btn:hover {
    background: #e0e0eb;
}

.get-btn.loading {
    width: 40px;
    height: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    pointer-events: none;
}

.get-btn.open {
    background: #007AFF;
    color: white;
}

.get-btn.open:hover {
    background: #0069d9;
}

.spinner {
    width: 14px;
    height: 14px;
    border: 2px solid #007AFF;
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    box-sizing: border-box;
    flex-shrink: 0;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.featured-row {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
    margin-bottom: 40px;
}

.featured-card {
    background: #f5f5f7;
    border-radius: 12px;
    padding: 20px;
    height: 200px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    transition: transform 0.2s;
    cursor: pointer;
}

.featured-card:hover {
    transform: scale(1.02);
}

.featured-category {
    font-size: 11px;
    color: #888;
    text-transform: uppercase;
    font-weight: 600;
    display: block;
    margin-bottom: 5px;
}

.featured-title {
    font-size: 18px;
    margin: 0 0 5px 0;
    color: #333;
}

.featured-desc {
    font-size: 13px;
    color: #666;
    margin: 0;
    line-height: 1.4;
}

.featured-bottom {
    display: flex;
    justify-content: flex-end;
}

.featured-img {
    width: 60px;
    height: 60px;
    object-fit: contain;
    border-radius: 12px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
}

.empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: #ccc;
}

.empty-state h2 {
    color: #333;
    margin: 20px 0 5px;
}

.empty-state p {
    color: #888;
}
</style>
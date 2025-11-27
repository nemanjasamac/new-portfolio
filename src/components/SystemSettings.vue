<script setup>
import { ref } from 'vue'

const props = defineProps({
  isOpen: Boolean,
  currentWallpaper: String,
  initialX: {
    type: Number,
    default: 300
  },
  initialY: {
    type: Number,
    default: 100
  },
  zIndex: {
    type: Number,
    default: 500
  }
})

const emit = defineEmits(['close', 'change-wallpaper', 'focus'])

const activeTab = ref('Wallpaper')
const searchQuery = ref('')
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: props.initialX, y: props.initialY })

const handleMouseDown = (e) => {
  emit('focus')
  // Allow dragging from sidebar or empty areas, but not controls or inputs
  if (e.target.closest('.window-controls') || e.target.closest('input') || e.target.closest('.menu-item') || e.target.closest('.wallpaper-item')) return
  
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

const wallpapers = [
  { id: 1, name: 'Tahoe Morning', url: 'https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1920&q=80' },
  { id: 2, name: 'Tahoe Evening', url: 'https://images.unsplash.com/photo-1472214103451-9374bd1c798e?auto=format&fit=crop&w=1920&q=80' },
  { id: 3, name: 'Tahoe Night', url: 'https://images.unsplash.com/photo-1470770841072-f978cf4d019e?auto=format&fit=crop&w=1920&q=80' },
  { id: 4, name: 'Sequoia Sunrise', url: 'https://images.unsplash.com/photo-1511497584788-876760111969?auto=format&fit=crop&w=1920&q=80' },
  { id: 5, name: 'Sequoia Morning', url: 'https://images.unsplash.com/photo-1542273917363-3b1817f69a2d?auto=format&fit=crop&w=1920&q=80' },
  { id: 6, name: 'Sequoia Night', url: 'https://images.unsplash.com/photo-1504608524841-42fe6f032b4b?auto=format&fit=crop&w=1920&q=80' },
  { id: 7, name: 'Sonoma Horizon', url: 'https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?auto=format&fit=crop&w=1920&q=80' },
  { id: 8, name: 'Sonoma Evening', url: 'https://images.unsplash.com/photo-1500530855697-b586d89ba3ee?auto=format&fit=crop&w=1920&q=80' },
  { id: 9, name: 'Sonoma from Above', url: 'https://images.unsplash.com/photo-1493246507139-91e8fad9978e?auto=format&fit=crop&w=1920&q=80' },
  { id: 10, name: 'Sonoma River', url: 'https://images.unsplash.com/photo-1476610182048-b716b8518aae?auto=format&fit=crop&w=1920&q=80' },
  { id: 11, name: 'Goa Beaches', url: 'https://images.unsplash.com/photo-1507525428034-b723cf961d3e?auto=format&fit=crop&w=1920&q=80' },
]

const selectWallpaper = (url) => {
  emit('change-wallpaper', url)
}

</script>

<template>
  <div 
    class="settings-window" 
    v-if="isOpen"
    :style="{ 
      left: `${position.x}px`, 
      top: `${position.y}px`,
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <!-- Sidebar -->
    <div class="sidebar">
      <div class="window-controls">
        <div class="control close" @click="emit('close')"></div>
        <div class="control minimize"></div>
        <div class="control maximize"></div>
      </div>
      
      <div class="search-bar">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
        <input type="text" placeholder="Search" v-model="searchQuery">
      </div>

      <div class="user-profile">
        <div class="avatar">NS</div>
        <div class="user-info">
          <div class="name">Nemanja Samac</div>
          <div class="account-type">samacOS Account</div>
        </div>
      </div>

      <div class="sidebar-menu">
        <div class="menu-item"><div class="icon-box blue">W</div>Wi-Fi</div>
        <div class="menu-item"><div class="icon-box blue">B</div>Bluetooth</div>
        <div class="menu-item"><div class="icon-box blue">N</div>Network</div>
        <div class="menu-item"><div class="icon-box green">B</div>Battery</div>
        <div class="menu-separator"></div>
        <div class="menu-item"><div class="icon-box gray">G</div>General</div>
        <div class="menu-item"><div class="icon-box blue">A</div>Accessibility</div>
        <div class="menu-item"><div class="icon-box gray">A</div>Appearance</div>
        <div class="menu-item active"><div class="icon-box cyan">W</div>Wallpaper</div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <div class="content-header">
        <div class="nav-arrows">
          <button>&lt;</button>
          <button>&gt;</button>
        </div>
        <div class="page-title">Wallpaper</div>
      </div>

      <div class="wallpaper-preview-section">
        <div class="current-preview" :style="{ backgroundImage: `url(${currentWallpaper})` }">
          <div class="preview-overlay"></div>
        </div>
        <div class="preview-info">
          <div class="wallpaper-name">Current Wallpaper</div>
          <div class="wallpaper-options">
            <select>
              <option>Fill Screen</option>
              <option>Fit to Screen</option>
              <option>Stretch to Fill</option>
              <option>Center</option>
            </select>
            <div class="toggle-row">
              <span>Show on all Spaces</span>
              <div class="toggle active"></div>
            </div>
          </div>
        </div>
      </div>

      <div class="wallpaper-grid-section">
        <div class="section-title">Landscape <span class="show-less">Show Less</span></div>
        <div class="wallpaper-grid">
          <div 
            v-for="wp in wallpapers" 
            :key="wp.id" 
            class="wallpaper-item"
            :class="{ active: currentWallpaper === wp.url }"
            @click="selectWallpaper(wp.url)"
          >
            <div class="wp-thumb" :style="{ backgroundImage: `url(${wp.url})` }">
                <div class="download-icon">↓</div>
                <div class="play-icon">▶</div>
            </div>
            <div class="wp-name">{{ wp.name }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.settings-window {
  position: absolute;
  width: 900px;
  height: 600px;
  background: #f5f5f5;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3);
  display: flex;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  border: 1px solid rgba(0,0,0,0.1);
}

/* Sidebar */
.sidebar {
  width: 260px;
  background: rgba(235, 235, 235, 0.9);
  backdrop-filter: blur(20px);
  padding: 20px 16px;
  display: flex;
  flex-direction: column;
  border-right: 1px solid rgba(0,0,0,0.1);
}

.window-controls {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
}

.control {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
}

.close { background: #ff5f56; border: 1px solid #e0443e; }
.minimize { background: #ffbd2e; border: 1px solid #dea123; }
.maximize { background: #27c93f; border: 1px solid #1aab29; }

.search-bar {
  background: rgba(0,0,0,0.05);
  border-radius: 6px;
  padding: 6px 10px;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 20px;
  border: 1px solid rgba(0,0,0,0.05);
}

.search-bar input {
  background: transparent;
  border: none;
  outline: none;
  font-size: 13px;
  width: 100%;
  color: #333;
}

.search-bar svg {
  color: #888;
}

.user-profile {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  margin-bottom: 10px;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #888;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 600;
}

.user-info .name {
  font-size: 13px;
  font-weight: 600;
  color: #333;
}

.user-info .account-type {
  font-size: 11px;
  color: #888;
}

.sidebar-menu {
  display: flex;
  flex-direction: column;
  gap: 2px;
  overflow-y: auto;
}

.menu-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 13px;
  color: #333;
  cursor: pointer;
}

.menu-item:hover {
  background: rgba(0,0,0,0.05);
}

.menu-item.active {
  background: rgba(0,0,0,0.1);
}

.icon-box {
  width: 22px;
  height: 22px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 12px;
  font-weight: 600;
}

.blue { background: #007AFF; }
.green { background: #34C759; }
.gray { background: #8E8E93; }
.cyan { background: #32ADE6; }

.menu-separator {
  height: 1px;
  background: rgba(0,0,0,0.1);
  margin: 8px 10px;
}

/* Main Content */
.main-content {
  flex: 1;
  padding: 20px 40px;
  overflow-y: auto;
}

.content-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 30px;
}

.nav-arrows {
  display: flex;
  gap: 10px;
}

.nav-arrows button {
  background: transparent;
  border: none;
  font-size: 18px;
  color: #888;
  cursor: pointer;
}

.page-title {
  font-size: 20px;
  font-weight: 600;
  color: #333;
}

.wallpaper-preview-section {
  display: flex;
  gap: 30px;
  margin-bottom: 40px;
}

.current-preview {
  width: 240px;
  height: 150px;
  border-radius: 12px;
  background-size: cover;
  background-position: center;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  position: relative;
}

.preview-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 15px;
}

.wallpaper-name {
  font-size: 16px;
  font-weight: 500;
  color: #333;
}

.wallpaper-options select {
  padding: 4px 12px;
  border-radius: 6px;
  border: 1px solid #ddd;
  background: white;
  font-size: 13px;
  margin-bottom: 10px;
  width: 150px;
}

.toggle-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13px;
  color: #333;
}

.toggle {
  width: 36px;
  height: 20px;
  background: #e9e9ea;
  border-radius: 10px;
  position: relative;
  cursor: pointer;
}

.toggle.active {
  background: #007AFF;
}

.toggle::after {
  content: '';
  position: absolute;
  top: 2px;
  left: 2px;
  width: 16px;
  height: 16px;
  background: white;
  border-radius: 50%;
  transition: transform 0.2s;
}

.toggle.active::after {
  transform: translateX(16px);
}

/* Grid */
.section-title {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 15px;
  display: flex;
  justify-content: space-between;
}

.show-less {
  color: #007AFF;
  font-size: 12px;
  cursor: pointer;
}

.wallpaper-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 20px;
}

.wallpaper-item {
  cursor: pointer;
}

.wp-thumb {
  width: 100%;
  aspect-ratio: 16/10;
  border-radius: 8px;
  background-size: cover;
  background-position: center;
  margin-bottom: 8px;
  position: relative;
  border: 2px solid transparent;
  transition: all 0.2s;
}

.wallpaper-item.active .wp-thumb {
  border-color: #007AFF;
  box-shadow: 0 0 0 2px rgba(0,122,255,0.3);
}

.wp-name {
  font-size: 12px;
  color: #333;
  text-align: center;
}

.download-icon, .play-icon {
    position: absolute;
    width: 20px;
    height: 20px;
    background: rgba(0,0,0,0.5);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 10px;
}

.download-icon {
    bottom: 6px;
    right: 6px;
    border: 1px solid rgba(255,255,255,0.5);
}

.play-icon {
    bottom: 6px;
    left: 6px;
}

</style>

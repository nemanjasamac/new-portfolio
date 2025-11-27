<script setup>
import { ref } from 'vue'

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
const position = ref({ x: 200, y: 100 })
const searchQuery = ref('')
const isStreetView = ref(false)
const currentCoords = ref({ lat: 37.3346, lon: -122.0090 })
const mapUrl = ref('https://www.openstreetmap.org/export/embed.html?bbox=-122.0105,37.3300,-122.0000,37.3380&layer=mapnik')

const favorites = [
  { id: 1, name: 'Home', icon: 'home', lat: 40.7128, lon: -74.0060 },
  { id: 2, name: 'Work', icon: 'briefcase', lat: 37.7749, lon: -122.4194 },
  { id: 3, name: 'School', icon: 'book', lat: 51.5074, lon: -0.1278 }
]

const recents = [
  { id: 1, name: 'Apple Park', location: 'Cupertino, CA', lat: 37.3346, lon: -122.0090 },
  { id: 2, name: 'San Francisco', location: 'California', lat: 37.7749, lon: -122.4194 },
  { id: 3, name: 'New York City', location: 'New York', lat: 40.7128, lon: -74.0060 }
]

const updateMapLocation = (lat, lon) => {
  currentCoords.value = { lat, lon }
  // Create a small bounding box around the point
  const delta = 0.005
  const bbox = `${lon - delta},${lat - delta},${lon + delta},${lat + delta}`
  mapUrl.value = `https://www.openstreetmap.org/export/embed.html?bbox=${bbox}&layer=mapnik`
}

const handleSearch = async () => {
  if (!searchQuery.value) return
  
  try {
    const response = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(searchQuery.value)}`)
    const data = await response.json()
    
    if (data && data.length > 0) {
      const { lat, lon } = data[0]
      updateMapLocation(parseFloat(lat), parseFloat(lon))
    }
  } catch (error) {
    console.error('Search failed:', error)
  }
}

const toggleStreetView = () => {
  isStreetView.value = !isStreetView.value
}

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.search-container') || e.target.closest('.sidebar-content')) return
  
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
</script>

<template>
  <div 
    v-if="isOpen"
    class="maps-window"
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
        <div class="control-btn close" @click.stop="emit('close')"></div>
        <div class="control-btn minimize"></div>
        <div class="control-btn maximize"></div>
      </div>
      
      <div class="search-container">
        <div class="search-input-wrapper">
          <svg class="search-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <input 
            type="text" 
            placeholder="Search Maps" 
            v-model="searchQuery"
            @keyup.enter="handleSearch"
          >
        </div>
      </div>

      <div class="sidebar-content">
        <div class="section">
          <div class="section-title">Favorites</div>
          <div class="favorites-grid">
            <div 
              v-for="fav in favorites" 
              :key="fav.id" 
              class="favorite-item"
              @click="updateMapLocation(fav.lat, fav.lon)"
            >
              <div class="fav-icon-circle">
                <svg v-if="fav.icon === 'home'" width="20" height="20" viewBox="0 0 24 24" fill="white" stroke="currentColor" stroke-width="2">
                  <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                  <polyline points="9 22 9 12 15 12 15 22"></polyline>
                </svg>
                <svg v-if="fav.icon === 'briefcase'" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2">
                  <rect x="2" y="7" width="20" height="14" rx="2" ry="2"></rect>
                  <path d="M16 21V5a2 2 0 0 0-2-2h-4a2 2 0 0 0-2 2v16"></path>
                </svg>
                <svg v-if="fav.icon === 'book'" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2">
                  <path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"></path>
                  <path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"></path>
                </svg>
              </div>
              <span class="fav-name">{{ fav.name }}</span>
            </div>
          </div>
        </div>

        <div class="section">
          <div class="section-title">Recents</div>
          <div class="recents-list">
            <div 
              v-for="recent in recents" 
              :key="recent.id" 
              class="recent-item"
              @click="updateMapLocation(recent.lat, recent.lon)"
            >
              <div class="recent-icon">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M12 22s-8-4.5-8-11.8A8 8 0 0 1 12 2a8 8 0 0 1 8 8.2c0 7.3-8 11.8-8 11.8z"></path>
                  <circle cx="12" cy="10" r="3"></circle>
                </svg>
              </div>
              <div class="recent-info">
                <div class="recent-name">{{ recent.name }}</div>
                <div class="recent-location">{{ recent.location }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Map View -->
    <div class="map-container">
      <iframe 
        v-if="!isStreetView"
        width="100%" 
        height="100%" 
        frameborder="0" 
        scrolling="no" 
        marginheight="0" 
        marginwidth="0" 
        :src="mapUrl" 
        style="border: 0; filter: saturate(1.2) contrast(1.1);">
      </iframe>
      
      <iframe 
        v-else
        width="100%" 
        height="100%" 
        frameborder="0" 
        style="border:0" 
        :src="`https://maps.google.com/maps?q=&layer=c&cbll=${currentCoords.lat},${currentCoords.lon}&cbp=11,0,0,0,0&output=svembed`" 
        allowfullscreen>
      </iframe>

      <div class="map-overlay-controls">
        <div 
          class="map-control-btn" 
          :class="{ active: isStreetView }"
          @click="toggleStreetView"
          title="Look Around"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M2 12s3-7 10-7 10 7 10 7-3 7-10 7-10-7-10-7Z"></path>
            <circle cx="12" cy="12" r="3"></circle>
          </svg>
        </div>
        <div class="map-control-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polygon points="3 11 22 2 13 21 11 13 3 11"></polygon>
          </svg>
        </div>
        <div class="map-control-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            <line x1="11" y1="8" x2="11" y2="14"></line>
            <line x1="8" y1="11" x2="14" y2="11"></line>
          </svg>
        </div>
        <div class="map-control-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="5" y1="11" x2="17" y2="11"></line>
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.maps-window {
  position: fixed;
  width: 900px;
  height: 600px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3), 0 0 0 1px rgba(0,0,0,0.1);
  display: flex;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.sidebar {
  width: 280px;
  background: rgba(245, 245, 245, 0.95);
  backdrop-filter: blur(20px);
  border-right: 1px solid #d1d1d1;
  display: flex;
  flex-direction: column;
  z-index: 2;
}

.window-controls {
  padding: 15px;
  display: flex;
  gap: 8px;
}

.control-btn {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
}

.close { background: #ff5f56; }
.minimize { background: #ffbd2e; }
.maximize { background: #27c93f; }

.search-container {
  padding: 0 15px 15px 15px;
}

.search-input-wrapper {
  background: #e3e3e3;
  border-radius: 8px;
  padding: 6px 10px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.search-icon {
  color: #888;
}

.search-input-wrapper input {
  border: none;
  background: transparent;
  width: 100%;
  outline: none;
  font-size: 13px;
  color: #333;
}

.sidebar-content {
  flex: 1;
  overflow-y: auto;
  padding: 0 15px;
}

.section {
  margin-bottom: 25px;
}

.section-title {
  font-size: 17px;
  font-weight: 700;
  color: #333;
  margin-bottom: 15px;
}

.favorites-grid {
  display: flex;
  gap: 15px;
}

.favorite-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.fav-icon-circle {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #007AFF;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.fav-name {
  font-size: 11px;
  color: #444;
}

.recents-list {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.recent-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
}

.recent-item:hover {
  background: rgba(0,0,0,0.05);
}

.recent-icon {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #999;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.recent-info {
  display: flex;
  flex-direction: column;
}

.recent-name {
  font-size: 14px;
  font-weight: 500;
  color: #333;
}

.recent-location {
  font-size: 12px;
  color: #888;
}

.map-container {
  flex: 1;
  position: relative;
  background: #fbfbfb;
}

.map-overlay-controls {
  position: absolute;
  bottom: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 1px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  overflow: hidden;
}

.map-control-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #333;
}

.map-control-btn:hover {
  background: #f5f5f5;
}

.map-control-btn:not(:last-child) {
  border-bottom: 1px solid #eee;
}

.map-control-btn.active {
  background: #e3e3e3;
  color: #007AFF;
}
</style>

<script setup>
import { ref, computed } from 'vue'

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

const sidebarItems = [
  { id: 'library', label: 'Library', icon: 'library' },
  { id: 'memories', label: 'Memories', icon: 'memories' },
  { id: 'people', label: 'People', icon: 'people' },
  { id: 'places', label: 'Places', icon: 'places' },
  { id: 'recents', label: 'Recents', icon: 'recents' },
  { id: 'imports', label: 'Imports', icon: 'imports' },
]

const albums = [
  { id: 'favorites', label: 'Favorites', count: 12 },
  { id: 'projects', label: 'My Projects', count: 3 },
  { id: 'wallpapers', label: 'Wallpapers', count: 5 },
]

const selectedItem = ref('library')

const photos = ref([
  { id: 1, url: 'https://images.unsplash.com/photo-1498050108023-c5249f4df085?auto=format&fit=crop&w=800&q=80', title: 'Coding Setup', date: 'Nov 20, 2023' },
  { id: 2, url: 'https://images.unsplash.com/photo-1504639725590-34d0984388bd?auto=format&fit=crop&w=800&q=80', title: 'Tech Stack', date: 'Nov 18, 2023' },
  { id: 3, url: 'https://images.unsplash.com/photo-1557821552-17105176677c?auto=format&fit=crop&w=800&q=80', title: 'E-Commerce', date: 'Oct 15, 2023' },
  { id: 4, url: 'https://images.unsplash.com/photo-1556742049-0cfed4f7a07d?auto=format&fit=crop&w=800&q=80', title: 'Shopping', date: 'Oct 12, 2023' },
  { id: 5, url: 'https://images.unsplash.com/photo-1484480974693-6ca0a78fb36b?auto=format&fit=crop&w=800&q=80', title: 'Task Manager', date: 'Sep 05, 2023' },
  { id: 6, url: 'https://images.unsplash.com/photo-1540350394557-8d14678e7f91?auto=format&fit=crop&w=800&q=80', title: 'Planning', date: 'Sep 01, 2023' },
  { id: 7, url: 'https://images.unsplash.com/photo-1472214103451-9374bd1c798e?auto=format&fit=crop&w=800&q=80', title: 'Nature', date: 'Aug 20, 2023' },
  { id: 8, url: 'https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?auto=format&fit=crop&w=800&q=80', title: 'Foggy Mountains', date: 'Aug 15, 2023' },
])

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.sidebar') || e.target.closest('.photo-grid')) return
  
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
    class="photos-window"
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
      
      <div class="sidebar-section">
        <div class="section-title">Photos</div>
        <div 
          v-for="item in sidebarItems" 
          :key="item.id"
          class="sidebar-item"
          :class="{ active: selectedItem === item.id }"
          @click="selectedItem = item.id"
        >
          <span class="item-icon">
            <svg v-if="item.id === 'library'" viewBox="0 0 24 24" fill="currentColor"><path d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 14H4V6h16v12zM4 18l4-5 3 4 5-6 4 5"/></svg>
            <svg v-if="item.id === 'memories'" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-6h2v6zm0-8h-2V7h2v2z"/></svg>
            <svg v-if="item.id === 'people'" viewBox="0 0 24 24" fill="currentColor"><path d="M16 11c1.66 0 2.99-1.34 2.99-3S17.66 5 16 5c-1.66 0-3 1.34-3 3s1.34 3 3 3zm-8 0c1.66 0 2.99-1.34 2.99-3S9.66 5 8 5C6.34 5 5 6.34 5 8s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V19h14v-2.5c0-2.33-4.67-3.5-7-3.5zm8 0c-.29 0-.62.02-.97.05 1.16.84 1.97 1.97 1.97 3.45V19h6v-2.5c0-2.33-4.67-3.5-7-3.5z"/></svg>
            <svg v-if="item.id === 'places'" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"/></svg>
            <svg v-if="item.id === 'recents'" viewBox="0 0 24 24" fill="currentColor"><path d="M11.99 2C6.47 2 2 6.48 2 12s4.47 10 9.99 10C17.52 22 22 17.52 22 12S17.52 2 11.99 2zM12 20c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8zm.5-13H11v6l5.25 3.15.75-1.23-4.5-2.67z"/></svg>
            <svg v-if="item.id === 'imports'" viewBox="0 0 24 24" fill="currentColor"><path d="M19 9h-4V3H9v6H5l7 7 7-7zM5 18v2h14v-2H5z"/></svg>
          </span>
          {{ item.label }}
        </div>
      </div>

      <div class="sidebar-section">
        <div class="section-title">Albums</div>
        <div 
          v-for="album in albums" 
          :key="album.id"
          class="sidebar-item"
          :class="{ active: selectedItem === album.id }"
          @click="selectedItem = album.id"
        >
          <span class="item-icon">
            <svg viewBox="0 0 24 24" fill="currentColor"><path d="M18 2H6c-1.1 0-2 .9-2 2v16c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zm0 18H6V4h2v8l2.5-1.5L13 12V4h5v16z"/></svg>
          </span>
          {{ album.label }}
          <span class="count">{{ album.count }}</span>
        </div>
      </div>
    </div>

    <div class="main-content">
      <div class="toolbar">
        <div class="toolbar-title">{{ sidebarItems.find(i => i.id === selectedItem)?.label || albums.find(a => a.id === selectedItem)?.label }}</div>
        <div class="toolbar-actions">
          <button class="action-btn">
            <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/></svg>
          </button>
          <button class="action-btn">
            <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
          </button>
        </div>
      </div>

      <div class="photo-grid">
        <div v-for="photo in photos" :key="photo.id" class="photo-item">
          <div class="photo-wrapper">
            <img :src="photo.url" :alt="photo.title" loading="lazy" />
          </div>
          <div class="photo-info">
            <span class="photo-title">{{ photo.title }}</span>
            <span class="photo-date">{{ photo.date }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.photos-window {
  position: absolute;
  width: 900px;
  height: 600px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
  display: flex;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.photos-window.maximized {
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  border-radius: 0;
}

.sidebar {
  width: 220px;
  background: rgba(240, 240, 240, 0.8);
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  padding-top: 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
  padding: 10px 15px 20px;
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

.sidebar-section {
  margin-bottom: 20px;
}

.section-title {
  padding: 0 15px 5px;
  font-size: 11px;
  font-weight: 600;
  color: #888;
  text-transform: uppercase;
}

.sidebar-item {
  display: flex;
  align-items: center;
  padding: 6px 15px;
  font-size: 13px;
  color: #333;
  cursor: pointer;
  transition: background 0.2s;
}

.sidebar-item:hover {
  background: rgba(0, 0, 0, 0.05);
}

.sidebar-item.active {
  background: rgba(0, 122, 255, 0.15);
  color: #007aff;
  font-weight: 500;
}

.item-icon {
  margin-right: 10px;
  display: flex;
  align-items: center;
  color: inherit;
}

.item-icon svg {
  width: 18px;
  height: 18px;
}

.count {
  margin-left: auto;
  font-size: 12px;
  color: #888;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #fff;
}

.toolbar {
  height: 50px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
}

.toolbar-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.toolbar-actions {
  display: flex;
  gap: 10px;
}

.action-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: #666;
  padding: 5px;
  border-radius: 4px;
}

.action-btn:hover {
  background: rgba(0, 0, 0, 0.05);
}

.photo-grid {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 20px;
  align-content: start;
}

.photo-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.photo-wrapper {
  aspect-ratio: 1;
  border-radius: 4px;
  overflow: hidden;
  background: #eee;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  transition: transform 0.2s;
}

.photo-wrapper:hover {
  transform: scale(1.02);
}

.photo-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.photo-info {
  display: flex;
  flex-direction: column;
}

.photo-title {
  font-size: 13px;
  font-weight: 500;
  color: #333;
}

.photo-date {
  font-size: 11px;
  color: #888;
}
</style>

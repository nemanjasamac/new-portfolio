<script setup>
import { ref } from 'vue'
import folderIcon from '../assets/Icons/folder.png'

const props = defineProps({
  initialX: {
    type: Number,
    default: 150
  },
  initialY: {
    type: Number,
    default: 100
  },
  zIndex: {
    type: Number,
    default: 100
  },
  initialSection: {
    type: String,
    default: 'about'
  }
})

const emit = defineEmits(['close', 'focus', 'open-project'])

const windowRef = ref(null)
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: props.initialX, y: props.initialY })
const isMaximized = ref(false)
const preMaximizeState = ref({ x: 0, y: 0 })
const activeSection = ref(props.initialSection)

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.sidebar-item')) return
  
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
  isMaximized.value = !isMaximized.value
  if (isMaximized.value) {
    preMaximizeState.value = { ...position.value }
    position.value = { x: 0, y: 24 } // Below menu bar
  } else {
    position.value = { x: preMaximizeState.value.x, y: preMaximizeState.value.y }
  }
}

const closeWindow = () => {
  emit('close')
}
</script>

<template>
  <div 
    ref="windowRef"
    class="finder-window"
    :class="{ maximized: isMaximized }"
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
        <div class="control close" @click.stop="closeWindow"></div>
        <div class="control minimize"></div>
        <div class="control maximize" @click.stop="toggleMaximize"></div>
      </div>
      
      <div class="sidebar-group">
        <div class="group-title">Favorites</div>
        <div 
          class="sidebar-item" 
          :class="{ active: activeSection === 'about' }"
          @click="activeSection = 'about'"
        >
          <span class="icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
          </span>
          About Me
        </div>
        <div 
          class="sidebar-item"
          :class="{ active: activeSection === 'projects' }"
          @click="activeSection = 'projects'"
        >
          <span class="icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path></svg>
          </span>
          Projects
        </div>
        <div 
          class="sidebar-item"
          :class="{ active: activeSection === 'downloads' }"
          @click="activeSection = 'downloads'"
        >
          <span class="icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>
          </span>
          Downloads
        </div>
      </div>
      
      <div class="sidebar-group">
        <div class="group-title">iCloud</div>
        <div 
          class="sidebar-item"
          :class="{ active: activeSection === 'icloud' }"
          @click="activeSection = 'icloud'"
        >
          <span class="icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"></path></svg>
          </span>
          iCloud Drive
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <div class="toolbar">
        <div class="nav-controls">
          <button class="nav-btn">&lt;</button>
          <button class="nav-btn">&gt;</button>
        </div>
        <div class="path-title">
          {{ 
            activeSection === 'about' ? 'About Me' : 
            activeSection === 'projects' ? 'Projects' : 
            activeSection === 'downloads' ? 'Downloads' : 'iCloud Drive' 
          }}
        </div>
        <div class="view-controls">
          <button class="view-btn">≣</button>
          <button class="view-btn">::</button>
        </div>
      </div>
      
      <div class="content-area">
        <!-- About Me Section -->
        <div v-if="activeSection === 'about'">
          <div class="about-header">
            <div class="avatar-placeholder">
              <span>NS</span>
            </div>
            <h1>Nemanja Samac</h1>
            <p class="subtitle">Software Engineer</p>
          </div>
          
          <div class="about-body">
            <p>I’m Nemanja, a software engineer obsessed with building clean, fast, and thoughtful digital experiences.
I design and develop full-stack applications with a focus on performance, simplicity, and smooth user experience.<br></br><br></br>

From backend architecture and APIs to modern front-end interfaces and 3D/interactive visuals, I enjoy creating products that feel polished, reliable, and intuitive. I’m currently working on several SaaS systems, experimenting with AI integration, and constantly pushing myself to learn and build more.</p>
            <!-- Add more content here later -->
          </div>
        </div>

        <!-- Projects Section -->
        <div v-else-if="activeSection === 'projects'" class="grid-view">
          <div class="grid-item" @dblclick="emit('open-project', 'portfolio')">
            <div class="folder-icon">
                <img :src="folderIcon" alt="Folder" width="64" height="64" />
            </div>
            <span class="item-label">Portfolio V1</span>
          </div>
          <div class="grid-item" @dblclick="emit('open-project', 'ecommerce')">
            <div class="folder-icon">
                <img :src="folderIcon" alt="Folder" width="64" height="64" />
            </div>
            <span class="item-label">E-Commerce App</span>
          </div>
          <div class="grid-item" @dblclick="emit('open-project', 'taskmanager')">
            <div class="folder-icon">
                <img :src="folderIcon" alt="Folder" width="64" height="64" />
            </div>
            <span class="item-label">Task Manager</span>
          </div>
        </div>

        <!-- Downloads Section -->
        <div v-else-if="activeSection === 'downloads'" class="list-view">
          <div class="list-item">
            <span class="file-icon">📄</span>
            <span class="file-name">Nemanja Samac CV.pdf</span>
            <span class="file-date">Today at 9:41 AM</span>
            <span class="file-size">1.2 MB</span>
          </div>
          <div class="list-item">
            <span class="file-icon">🖼️</span>
            <span class="file-name">profile-pic.jpg</span>
            <span class="file-date">Yesterday at 2:30 PM</span>
            <span class="file-size">2.4 MB</span>
          </div>
        </div>

        <!-- iCloud Section -->
        <div v-else-if="activeSection === 'icloud'" class="empty-state">
          <div class="cloud-icon">☁️</div>
          <p>iCloud Drive is empty</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.finder-window {
  position: absolute;
  width: 800px;
  height: 500px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3);
  display: flex;
  overflow: hidden;
  backdrop-filter: blur(20px);
  border: 1px solid rgba(0,0,0,0.1);
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}

.finder-window.maximized {
  width: 100% !important;
  height: calc(100vh - 24px) !important; /* Subtract menu bar height */
  border-radius: 0;
}

/* Sidebar */
.sidebar {
  width: 200px;
  background: rgba(240, 240, 240, 0.9);
  border-right: 1px solid rgba(0,0,0,0.1);
  padding: 10px;
  display: flex;
  flex-direction: column;
}

.window-controls {
  display: flex;
  gap: 8px;
  padding: 10px 8px;
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

.sidebar-group {
  margin-bottom: 20px;
}

.group-title {
  font-size: 11px;
  color: #888;
  font-weight: 600;
  margin-bottom: 5px;
  padding-left: 10px;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 13px;
  color: #333;
  cursor: pointer;
}

.sidebar-item:hover {
  background: rgba(0,0,0,0.05);
}

.sidebar-item.active {
  background: rgba(0,0,0,0.1);
}

.icon {
  font-size: 16px;
}

/* Main Content */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #fff;
}

.toolbar {
  height: 50px;
  border-bottom: 1px solid rgba(0,0,0,0.1);
  display: flex;
  align-items: center;
  padding: 0 20px;
  justify-content: space-between;
  background: rgba(255,255,255,0.8);
}

.nav-controls, .view-controls {
  display: flex;
  gap: 10px;
}

.nav-btn, .view-btn {
  border: none;
  background: none;
  color: #555;
  font-size: 16px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
}

.nav-btn:hover, .view-btn:hover {
  background: rgba(0,0,0,0.05);
}

.path-title {
  font-weight: 600;
  color: #333;
}

.content-area {
  flex: 1;
  padding: 40px;
  overflow-y: auto;
}

.about-header {
  text-align: center;
  margin-bottom: 40px;
}

.avatar-placeholder {
  width: 100px;
  height: 100px;
  background: linear-gradient(135deg, #007AFF, #5856D6);
  border-radius: 50%;
  margin: 0 auto 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 32px;
  font-weight: bold;
}

h1 {
  font-size: 28px;
  margin: 0 0 10px;
  color: #333;
}

.subtitle {
  color: #666;
  font-size: 18px;
  margin: 0;
}

.about-body {
  max-width: 600px;
  margin: 0 auto;
  line-height: 1.6;
  color: #444;
}

/* Grid View */
.grid-view {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 20px;
  padding: 20px;
}

.grid-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}

.grid-item:hover {
  background: rgba(0,0,0,0.05);
}

.folder-icon {
  font-size: 48px;
}

.item-label {
  font-size: 13px;
  text-align: center;
  color: #333;
}

/* List View */
.list-view {
  display: flex;
  flex-direction: column;
}

.list-item {
  display: flex;
  align-items: center;
  padding: 8px 20px;
  gap: 15px;
  border-bottom: 1px solid #eee;
  font-size: 13px;
  color: #333;
  cursor: default;
}

.list-item:nth-child(even) {
  background: #f9f9f9;
}

.list-item:hover {
  background: #f0f0f0;
}

.file-icon {
  font-size: 20px;
}

.file-name {
  flex: 1;
  font-weight: 500;
}

.file-date {
  color: #888;
  width: 150px;
}

.file-size {
  color: #888;
  width: 60px;
  text-align: right;
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #ccc;
}

.cloud-icon {
  font-size: 64px;
  margin-bottom: 10px;
}
</style>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  project: {
    type: Object,
    required: true
  },
  initialX: {
    type: Number,
    default: 200
  },
  initialY: {
    type: Number,
    default: 100
  },
  zIndex: {
    type: Number,
    default: 100
  }
})

const emit = defineEmits(['close', 'focus'])

const windowRef = ref(null)
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: props.initialX, y: props.initialY })
const isMaximized = ref(false)
const preMaximizeState = ref({ x: 0, y: 0 })

const activeImageIndex = ref(0)

const nextImage = () => {
  if (props.project.images && props.project.images.length) {
    activeImageIndex.value = (activeImageIndex.value + 1) % props.project.images.length
  }
}

const prevImage = () => {
  if (props.project.images && props.project.images.length) {
    activeImageIndex.value = (activeImageIndex.value - 1 + props.project.images.length) % props.project.images.length
  }
}

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.action-btn') || e.target.closest('.image-nav-btn')) return
  
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
    class="project-window"
    :class="{ maximized: isMaximized }"
    :style="{ 
      left: `${position.x}px`, 
      top: `${position.y}px`,
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <!-- Title Bar -->
    <div class="title-bar">
      <div class="window-controls">
        <div class="control close" @click.stop="closeWindow"></div>
        <div class="control minimize"></div>
        <div class="control maximize" @click.stop="toggleMaximize"></div>
      </div>
      <div class="window-title">{{ project.title }}</div>
      <div class="placeholder"></div>
    </div>

    <!-- Content -->
    <div class="content-area">
      <!-- Image Gallery -->
      <div class="image-gallery" v-if="project.images && project.images.length">
        <div class="main-image-container">
          <img :src="project.images[activeImageIndex]" class="main-image" />
          
          <button class="image-nav-btn prev" @click="prevImage" v-if="project.images.length > 1">
            &lt;
          </button>
          <button class="image-nav-btn next" @click="nextImage" v-if="project.images.length > 1">
            &gt;
          </button>
          
          <div class="image-dots" v-if="project.images.length > 1">
            <span 
              v-for="(_, index) in project.images" 
              :key="index" 
              class="dot"
              :class="{ active: index === activeImageIndex }"
              @click="activeImageIndex = index"
            ></span>
          </div>
        </div>
      </div>

      <div class="project-header">
        <div class="project-icon">
          <span>📂</span>
        </div>
        <div class="header-text">
          <h1>{{ project.title }}</h1>
          <p class="subtitle">{{ project.subtitle || 'Project Details' }}</p>
        </div>
      </div>

      <div class="project-body">
        <div class="description-section">
          <h3>About</h3>
          <p>{{ project.description }}</p>
        </div>

        <div class="tech-section" v-if="project.technologies && project.technologies.length">
          <h3>Technologies</h3>
          <div class="tech-tags">
            <span v-for="tech in project.technologies" :key="tech" class="tech-tag">
              {{ tech }}
            </span>
          </div>
        </div>

        <div class="actions-section">
          <a v-if="project.link" :href="project.link" target="_blank" class="action-btn primary">
            Visit Project
          </a>
          <a v-if="project.github" :href="project.github" target="_blank" class="action-btn secondary">
            View Code
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.project-window {
  position: absolute;
  width: 600px;
  height: 450px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  backdrop-filter: blur(20px);
  border: 1px solid rgba(0,0,0,0.1);
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}

.project-window.maximized {
  width: 100% !important;
  height: calc(100vh - 24px) !important;
  border-radius: 0;
}

.title-bar {
  height: 38px;
  background: linear-gradient(to bottom, #e6e6e6, #dcdcdc);
  border-bottom: 1px solid #ccc;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
  width: 60px;
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

.window-title {
  font-size: 13px;
  font-weight: 600;
  color: #444;
}

.placeholder {
  width: 60px;
}

.content-area {
  flex: 1;
  padding: 30px;
  overflow-y: auto;
  background: #fff;
}

.project-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #eee;
}

.project-icon {
  width: 64px;
  height: 64px;
  background: #f0f0f0;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
}

.header-text h1 {
  margin: 0;
  font-size: 24px;
  color: #333;
}

.subtitle {
  margin: 5px 0 0;
  color: #888;
  font-size: 14px;
}

.project-body {
  color: #444;
  line-height: 1.6;
}

h3 {
  font-size: 14px;
  color: #888;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin: 20px 0 10px;
}

.tech-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tech-tag {
  background: #f0f0f0;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  color: #555;
}

.actions-section {
  margin-top: 40px;
  display: flex;
  gap: 15px;
}

.action-btn {
  padding: 8px 16px;
  border-radius: 6px;
  text-decoration: none;
  font-size: 13px;
  font-weight: 500;
  transition: all 0.2s;
}

.action-btn.primary {
  background: #007AFF;
  color: white;
}

.action-btn.primary:hover {
  background: #0063cc;
}

.action-btn.secondary {
  background: #f0f0f0;
  color: #333;
}

.action-btn.secondary:hover {
  background: #e0e0e0;
}

.image-gallery {
  margin-bottom: 20px;
  border-radius: 8px;
  overflow: hidden;
  background: #000;
}

.main-image-container {
  position: relative;
  width: 100%;
  height: 250px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.image-nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  transition: background 0.2s;
}

.image-nav-btn:hover {
  background: rgba(255, 255, 255, 0.4);
}

.image-nav-btn.prev {
  left: 10px;
}

.image-nav-btn.next {
  right: 10px;
}

.image-dots {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 6px;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.4);
  cursor: pointer;
}

.dot.active {
  background: white;
}
</style>

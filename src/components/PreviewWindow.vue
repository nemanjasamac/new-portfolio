<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  title: {
    type: String,
    default: 'Preview'
  },
  fileUrl: {
    type: String,
    required: true
  },
  initialX: {
    type: Number,
    default: 100
  },
  initialY: {
    type: Number,
    default: 50
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
const preMaximizeState = ref({ x: 0, y: 0, width: 0, height: 0 })

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.toolbar-btn')) return
  
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
  document.body.style.cursor = ''
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
    class="preview-window"
    :class="{ maximized: isMaximized }"
    :style="{ 
      left: `${position.x}px`, 
      top: `${position.y}px`,
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <!-- Title Bar / Toolbar -->
    <div class="title-bar">
      <div class="window-controls">
        <div class="control close" @click.stop="closeWindow"></div>
        <div class="control minimize"></div>
        <div class="control maximize" @click.stop="toggleMaximize"></div>
      </div>
      
      <div class="file-info">
        <span class="file-icon">📄</span>
        <span class="file-name">{{ title }}</span>
      </div>
      
      <div class="toolbar-spacer"></div>
    </div>

    <!-- Toolbar Tools -->
    <div class="toolbar">
      <div class="tool-group">
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x="9" y="3" x2="9" y2="21"></line></svg>
          <span class="btn-label">View</span>
        </button>
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line><line x1="11" y1="8" x2="11" y2="14"></line><line x1="8" y1="11" x2="14" y2="11"></line></svg>
          <span class="btn-label">Zoom</span>
        </button>
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 19l7-7 3 3-7 7-3-3z"></path><path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"></path><path d="M2 2l7.586 7.586"></path><circle cx="11" cy="11" r="2"></circle></svg>
          <span class="btn-label">Highlight</span>
        </button>
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M2.5 2v6h6M2.66 15.57a10 10 0 1 0 .57-8.38"/></svg>
          <span class="btn-label">Rotate</span>
        </button>
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 20h9"></path><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"></path></svg>
          <span class="btn-label">Markup</span>
        </button>
        <button class="toolbar-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
          <span class="btn-label">Search</span>
        </button>
      </div>
    </div>

    <!-- Content -->
    <div class="window-content">
      <iframe :src="`${fileUrl}#toolbar=0`" class="pdf-frame" frameborder="0"></iframe>
    </div>
  </div>
</template>

<style scoped>
.preview-window {
  position: absolute;
  width: 800px;
  height: 600px;
  background: #f5f5f5;
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3), 0 0 0 1px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}

.preview-window.maximized {
  width: 100vw !important;
  height: calc(100vh - 24px) !important;
  border-radius: 0;
}

.title-bar {
  height: 38px;
  background: #e8e8e8;
  display: flex;
  align-items: center;
  padding: 0 12px;
  border-bottom: 1px solid #d1d1d1;
  user-select: none;
}

.window-controls {
  display: flex;
  gap: 8px;
  margin-right: 16px;
}

.control {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: var(--mac-pointer);
}

.control.close { background: #ff5f56; border: 1px solid #e0443e; }
.control.minimize { background: #ffbd2e; border: 1px solid #dea123; }
.control.maximize { background: #27c93f; border: 1px solid #1aab29; }

.file-info {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  font-weight: 600;
  color: #333;
  flex: 1;
  justify-content: center;
  margin-right: 60px; /* Balance the controls */
}

.file-icon {
  font-size: 14px;
}

.toolbar {
  height: 44px;
  background: #f6f6f6;
  border-bottom: 1px solid #d1d1d1;
  display: flex;
  align-items: center;
  padding: 0 12px;
  justify-content: center;
}

.tool-group {
  display: flex;
  gap: 16px;
}

.toolbar-btn {
  background: none;
  border: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  cursor: var(--mac-pointer);
  color: #555;
  padding: 4px 8px;
  border-radius: 4px;
}

.toolbar-btn:hover {
  background: rgba(0,0,0,0.05);
}

.toolbar-btn svg {
  opacity: 0.8;
}

.btn-label {
  font-size: 9px;
  font-weight: 500;
}

.window-content {
  flex: 1;
  background: #525659; /* PDF viewer background color */
  position: relative;
}

.pdf-frame {
  width: 100%;
  height: 100%;
  display: block;
}
</style>

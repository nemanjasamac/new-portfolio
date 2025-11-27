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
const position = ref({ x: 150, y: 80 })

const notes = ref([
  {
    id: 1,
    title: 'Project Ideas',
    date: 'Today',
    content: '1. Portfolio Website\n2. E-commerce App\n3. Task Manager\n4. Weather App\n\nNeed to start working on the portfolio design. Maybe use Vue.js?',
    deleted: false
  },
  {
    id: 2,
    title: 'Shopping List',
    date: 'Yesterday',
    content: '- Milk\n- Eggs\n- Bread\n- Coffee\n- Apples',
    deleted: false
  },
  {
    id: 3,
    title: 'Meeting Notes',
    date: 'Nov 24',
    content: 'Team meeting at 10 AM.\n\nDiscussed:\n- Q4 Goals\n- New features\n- Bug fixes',
    deleted: false
  }
])

const selectedNoteId = ref(1)
const searchQuery = ref('')
const currentFolder = ref('notes') // 'notes' or 'deleted'

const selectedNote = computed(() => notes.value.find(n => n.id === selectedNoteId.value))
const filteredNotes = computed(() => {
  let filtered = notes.value.filter(n => {
    if (currentFolder.value === 'notes') return !n.deleted
    if (currentFolder.value === 'deleted') return n.deleted
    return !n.deleted
  })

  if (searchQuery.value) {
    filtered = filtered.filter(n => 
      n.title.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
      n.content.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  }
  return filtered
})

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.note-item') || e.target.closest('.editor-content') || e.target.closest('input')) return
  
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

const createNewNote = () => {
  if (currentFolder.value === 'deleted') return // Cannot create new notes in trash

  const newId = Math.max(...notes.value.map(n => n.id)) + 1
  const newNote = {
    id: newId,
    title: 'New Note',
    date: 'Just now',
    content: '',
    deleted: false
  }
  notes.value.unshift(newNote)
  selectedNoteId.value = newId
}

const deleteNote = () => {
  if (!selectedNoteId.value) return
  
  const noteIndex = notes.value.findIndex(n => n.id === selectedNoteId.value)
  if (noteIndex === -1) return

  if (currentFolder.value === 'notes') {
    // Move to trash
    notes.value[noteIndex].deleted = true
  } else {
    // Permanently delete
    notes.value = notes.value.filter(n => n.id !== selectedNoteId.value)
  }

  // Select next available note in current view
  const remainingNotes = filteredNotes.value
  if (remainingNotes.length > 0) {
    selectedNoteId.value = remainingNotes[0].id
  } else {
    selectedNoteId.value = null
  }
}

const restoreNote = () => {
  if (!selectedNoteId.value || currentFolder.value !== 'deleted') return
  const note = notes.value.find(n => n.id === selectedNoteId.value)
  if (note) {
    note.deleted = false
    // Select next available note in trash
    const remainingNotes = filteredNotes.value
    if (remainingNotes.length > 0) {
      selectedNoteId.value = remainingNotes[0].id
    } else {
      selectedNoteId.value = null
    }
  }
}

const switchFolder = (folder) => {
  currentFolder.value = folder
  selectedNoteId.value = null
  // Auto-select first note in folder if available
  const availableNotes = notes.value.filter(n => folder === 'notes' ? !n.deleted : n.deleted)
  if (availableNotes.length > 0) {
    selectedNoteId.value = availableNotes[0].id
  }
}
</script>

<template>
  <div 
    v-if="isOpen"
    class="notes-window"
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
      
      <div class="sidebar-content">
        <div class="folder-group">
          <div class="group-title">iCloud</div>
          <div 
            class="folder-item" 
            :class="{ active: currentFolder === 'notes' }"
            @click="switchFolder('notes')"
          >
            <svg class="folder-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"></path>
            </svg>
            <span class="folder-name">Notes</span>
          </div>
          <div 
            class="folder-item"
            :class="{ active: currentFolder === 'deleted' }"
            @click="switchFolder('deleted')"
          >
            <svg class="folder-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
            </svg>
            <span class="folder-name">Recently Deleted</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Note List -->
    <div class="note-list-col">
      <div class="list-header">
        <div class="view-mode">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="3" width="7" height="7"></rect>
            <rect x="14" y="3" width="7" height="7"></rect>
            <rect x="14" y="14" width="7" height="7"></rect>
            <rect x="3" y="14" width="7" height="7"></rect>
          </svg>
        </div>
        <div class="trash-icon" @click="deleteNote" :title="currentFolder === 'notes' ? 'Delete' : 'Delete Permanently'">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
          </svg>
        </div>
      </div>
      
      <div class="notes-list">
        <div 
          v-for="note in filteredNotes" 
          :key="note.id" 
          class="note-item"
          :class="{ active: selectedNoteId === note.id }"
          @click="selectedNoteId = note.id"
        >
          <div class="note-title">{{ note.title }}</div>
          <div class="note-preview">
            <span class="note-date">{{ note.date }}</span>
            <span class="note-excerpt">{{ note.content.substring(0, 30).replace(/\n/g, ' ') }}...</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Editor -->
    <div class="editor-col">
      <div class="editor-toolbar">
        <div class="search-box">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <input type="text" placeholder="Search" v-model="searchQuery">
        </div>
        <div class="toolbar-actions">
          <div v-if="currentFolder === 'deleted'" class="restore-btn" @click="restoreNote">
            Recover
          </div>
          <div class="compose-btn" @click="createNewNote" :class="{ disabled: currentFolder === 'deleted' }">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
            </svg>
          </div>
        </div>
      </div>
      
      <div class="editor-content" v-if="selectedNote">
        <div class="note-meta">{{ selectedNote.date }}</div>
        <textarea 
          class="note-textarea" 
          v-model="selectedNote.content"
          @input="selectedNote.title = selectedNote.content.split('\n')[0] || 'New Note'"
          :disabled="currentFolder === 'deleted'"
        ></textarea>
      </div>
      <div class="no-selection" v-else>
        No Note Selected
      </div>
    </div>
  </div>
</template>

<style scoped>
.notes-window {
  position: fixed;
  width: 800px;
  height: 500px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3), 0 0 0 1px rgba(0,0,0,0.1);
  display: flex;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

/* Sidebar */
.sidebar {
  width: 200px;
  background: rgba(245, 245, 245, 0.9);
  backdrop-filter: blur(20px);
  border-right: 1px solid #d1d1d1;
  display: flex;
  flex-direction: column;
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

.sidebar-content {
  padding: 10px;
}

.group-title {
  font-size: 11px;
  font-weight: 600;
  color: #888;
  margin-bottom: 5px;
  padding-left: 10px;
}

.folder-item {
  display: flex;
  align-items: center;
  padding: 6px 10px;
  border-radius: 5px;
  cursor: pointer;
  color: #444;
  font-size: 13px;
}

.folder-item.active {
  background: #dcdcdc;
}

.folder-icon {
  width: 16px;
  height: 16px;
  margin-right: 8px;
  color: #f5b041;
}

/* Note List */
.note-list-col {
  width: 250px;
  background: white;
  border-right: 1px solid #e0e0e0;
  display: flex;
  flex-direction: column;
}

.list-header {
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 15px;
  border-bottom: 1px solid #e0e0e0;
}

.view-mode, .trash-icon {
  color: #666;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
}

.view-mode:hover, .trash-icon:hover {
  background: #f0f0f0;
}

.notes-list {
  flex: 1;
  overflow-y: auto;
}

.note-item {
  padding: 10px 20px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}

.note-item.active {
  background: #f5d668;
}

.note-item.active .note-title,
.note-item.active .note-date,
.note-item.active .note-excerpt {
  color: #5c4d1f;
}

.note-title {
  font-weight: 700;
  font-size: 13px;
  margin-bottom: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.note-preview {
  font-size: 12px;
  color: #888;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.note-date {
  margin-right: 5px;
  color: #444;
}

/* Editor */
.editor-col {
  flex: 1;
  background: white;
  display: flex;
  flex-direction: column;
}

.editor-toolbar {
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 15px;
  border-bottom: 1px solid #e0e0e0;
}

.search-box {
  display: flex;
  align-items: center;
  background: #f0f0f0;
  border-radius: 5px;
  padding: 4px 8px;
  width: 150px;
}

.search-box input {
  border: none;
  background: transparent;
  margin-left: 5px;
  font-size: 12px;
  width: 100%;
  outline: none;
}

.toolbar-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.restore-btn {
  font-size: 12px;
  color: #007AFF;
  cursor: pointer;
  font-weight: 500;
}

.restore-btn:hover {
  text-decoration: underline;
}

.compose-btn {
  color: #666;
  cursor: pointer;
}

.compose-btn:hover {
  color: #333;
}

.compose-btn.disabled {
  opacity: 0.3;
  cursor: default;
  pointer-events: none;
}

.editor-content {
  flex: 1;
  padding: 20px 30px;
  display: flex;
  flex-direction: column;
}

.note-meta {
  text-align: center;
  color: #aaa;
  font-size: 12px;
  margin-bottom: 15px;
}

.note-textarea {
  flex: 1;
  border: none;
  resize: none;
  outline: none;
  font-size: 15px;
  line-height: 1.5;
  color: #333;
  font-family: inherit;
}

.no-selection {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ccc;
}
</style>
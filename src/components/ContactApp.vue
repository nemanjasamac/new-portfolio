<script setup>
import { ref } from 'vue'
import emailjs from '@emailjs/browser'

const props = defineProps({
  isOpen: Boolean,
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

const form = ref({
  to: 'nemanja@samac.dev',
  cc: '',
  subject: '',
  message: ''
})

const isSending = ref(false)

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('input') || e.target.closest('textarea') || e.target.closest('.toolbar-btn')) return
  
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

const sendEmail = async () => {
  if (!form.value.subject || !form.value.message) {
    alert('Please fill in the subject and message.')
    return
  }

  isSending.value = true
  
  try {
    await emailjs.send(
      import.meta.env.VITE_EMAIL_SERVICE_ID,
      import.meta.env.VITE_EMAIL_TEMPLATE_ID,
      {
        to_name: 'Nemanja',
        from_name: 'Visitor',
        message: form.value.message,
        subject: form.value.subject,
        reply_to: form.value.cc
      },
      import.meta.env.VITE_EMAIL_PUBLIC_KEY
    )
    
    alert('Email sent successfully!')
    emit('close')
    form.value.subject = ''
    form.value.message = ''
  } catch (error) {
    console.error('Email error:', error)
    alert('Failed to send email. Please try again.')
  } finally {
    isSending.value = false
  }
}

</script>

<template>
  <div 
    class="mail-window" 
    v-if="isOpen"
    :style="{ 
      left: `${position.x}px`, 
      top: `${position.y}px`,
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <div class="title-bar">
      <div class="window-controls">
        <div class="control close" @click="emit('close')"></div>
        <div class="control minimize"></div>
        <div class="control maximize"></div>
      </div>
      <div class="window-title">New Message</div>
      <div class="placeholder"></div>
    </div>

    <div class="toolbar">
      <button class="toolbar-btn" @click="sendEmail" :disabled="isSending">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
      </button>
      <button class="toolbar-btn">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21.44 11.05l-9.19 9.19a6 6 0 0 1-8.49-8.49l9.19-9.19a4 4 0 0 1 5.66 5.66l-9.2 9.19a2 2 0 0 1-2.83-2.83l8.49-8.48"></path></svg>
      </button>
      <button class="toolbar-btn">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg>
      </button>
      <button class="toolbar-btn">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
      </button>
    </div>

    <div class="compose-area">
      <div class="field-row">
        <label>To:</label>
        <input type="text" v-model="form.to" readonly class="readonly-input">
      </div>
      <div class="field-row">
        <label>Cc:</label>
        <input type="text" v-model="form.cc">
      </div>
      <div class="field-row">
        <label>Subject:</label>
        <input type="text" v-model="form.subject" placeholder="Project Inquiry">
      </div>
      <div class="message-body">
        <textarea v-model="form.message" placeholder="Hi Nemanja, I'd like to discuss..."></textarea>
      </div>
    </div>
  </div>
</template>

<style scoped>
.mail-window {
  position: absolute;
  width: 600px;
  height: 500px;
  background: #f5f5f5;
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.3);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  border: 1px solid rgba(0,0,0,0.1);
}

.title-bar {
  height: 38px;
  background: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 12px;
  border-bottom: 1px solid #ddd;
}

.window-controls {
  display: flex;
  gap: 8px;
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
  color: #666;
  font-weight: 500;
}

.placeholder {
  width: 52px; /* Balance controls */
}

.toolbar {
  height: 44px;
  background: #f5f5f5;
  display: flex;
  align-items: center;
  padding: 0 16px;
  gap: 20px;
  border-bottom: 1px solid #ddd;
}

.toolbar-btn {
  background: none;
  border: none;
  color: #555;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.toolbar-btn:hover {
  background: rgba(0,0,0,0.05);
  color: #000;
}

.toolbar-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.compose-area {
  flex: 1;
  background: white;
  display: flex;
  flex-direction: column;
}

.field-row {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  border-bottom: 1px solid #eee;
}

.field-row label {
  width: 60px;
  font-size: 13px;
  color: #888;
  text-align: right;
  margin-right: 10px;
}

.field-row input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 13px;
  color: #333;
}

.readonly-input {
  color: #007AFF !important;
  background: transparent;
}

.message-body {
  flex: 1;
  padding: 16px;
}

.message-body textarea {
  width: 100%;
  height: 100%;
  border: none;
  outline: none;
  resize: none;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 14px;
  line-height: 1.5;
  color: #333;
}
</style>

<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'

const props = defineProps({
  initialX: {
    type: Number,
    default: 100
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

const emit = defineEmits(['close', 'focus', 'open-app'])

const windowRef = ref(null)
const inputRef = ref(null)
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: props.initialX, y: props.initialY })
const isMaximized = ref(false)
const preMaximizeState = ref({ x: 0, y: 0 })

const history = ref([
  { type: 'output', content: 'Last login: ' + new Date().toString().split('GMT')[0] + ' on ttys000' },
  { type: 'output', content: 'Welcome to NemanjaOS Terminal. Type "help" for a list of commands.' }
])
const currentInput = ref('')
const commandHistory = ref([])
const historyIndex = ref(-1)

const fileSystem = {
  '~': {
    type: 'dir',
    children: {
      'projects': { type: 'dir' },
      'about.txt': { type: 'file', content: 'Hi! I am Nemanja Samac, a Full Stack Developer.' },
      'contact.txt': { type: 'file', content: 'Email: nemanja@example.com\nGitHub: github.com/nemanjasamac' },
      'skills.txt': { type: 'file', content: '- Vue.js\n- React\n- Node.js\n- TypeScript' }
    }
  }
}

const currentPath = ref('~')

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls')) return
  
  // Focus input when clicking anywhere in the terminal
  if (inputRef.value) {
    inputRef.value.focus()
  }

  if (e.target.closest('.terminal-content')) return // Don't drag if clicking content

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
    position.value = { x: 0, y: 24 }
  } else {
    position.value = { x: preMaximizeState.value.x, y: preMaximizeState.value.y }
  }
}

const closeWindow = () => {
  emit('close')
}

const executeCommand = () => {
  const cmd = currentInput.value.trim()
  
  if (!cmd) {
    history.value.push({ type: 'input', content: '' })
    currentInput.value = ''
    return
  }

  history.value.push({ type: 'input', content: cmd })
  commandHistory.value.push(cmd)
  historyIndex.value = commandHistory.value.length

  const parts = cmd.split(' ')
  const command = parts[0].toLowerCase()
  const args = parts.slice(1)

  switch (command) {
    case 'help':
      history.value.push({ 
        type: 'output', 
        content: `Available commands:
  help     - Show this help message
  ls       - List directory contents
  cat      - Print file content
  clear    - Clear the terminal screen
  whoami   - Print current user
  date     - Print current date
  open     - Open an application (e.g., open resume)` 
      })
      break
      
    case 'ls':
      const files = Object.keys(fileSystem['~'].children).map(name => {
        const isDir = fileSystem['~'].children[name].type === 'dir'
        return isDir ? name + '/' : name
      })
      history.value.push({ type: 'output', content: files.join('  ') })
      break
      
    case 'cat':
      if (args.length === 0) {
        history.value.push({ type: 'output', content: 'usage: cat [file]' })
      } else {
        const fileName = args[0]
        const file = fileSystem['~'].children[fileName]
        if (file && file.type === 'file') {
          history.value.push({ type: 'output', content: file.content })
        } else if (file && file.type === 'dir') {
          history.value.push({ type: 'output', content: `cat: ${fileName}: Is a directory` })
        } else {
          history.value.push({ type: 'output', content: `cat: ${fileName}: No such file or directory` })
        }
      }
      break
      
    case 'clear':
      history.value = []
      break
      
    case 'whoami':
      history.value.push({ type: 'output', content: 'guest' })
      break
      
    case 'date':
      history.value.push({ type: 'output', content: new Date().toString() })
      break

    case 'open':
      if (args.length === 0) {
        history.value.push({ type: 'output', content: 'usage: open [app_name]' })
      } else {
        const appName = args[0].toLowerCase()
        if (appName === 'resume' || appName === 'cv') {
           emit('open-app', 'resume')
           history.value.push({ type: 'output', content: 'Opening Resume...' })
        } else if (appName === 'finder' || appName === 'about') {
           emit('open-app', 'about-me')
           history.value.push({ type: 'output', content: 'Opening About Me...' })
        } else {
           history.value.push({ type: 'output', content: `Application '${appName}' not found` })
        }
      }
      break
      
    default:
      history.value.push({ type: 'output', content: `zsh: command not found: ${command}` })
  }

  currentInput.value = ''
  scrollToBottom()
}

const handleKeyDown = (e) => {
  if (e.key === 'ArrowUp') {
    e.preventDefault()
    if (historyIndex.value > 0) {
      historyIndex.value--
      currentInput.value = commandHistory.value[historyIndex.value]
    }
  } else if (e.key === 'ArrowDown') {
    e.preventDefault()
    if (historyIndex.value < commandHistory.value.length - 1) {
      historyIndex.value++
      currentInput.value = commandHistory.value[historyIndex.value]
    } else {
      historyIndex.value = commandHistory.value.length
      currentInput.value = ''
    }
  }
}

const scrollToBottom = () => {
  nextTick(() => {
    if (windowRef.value) {
      const content = windowRef.value.querySelector('.terminal-content')
      if (content) content.scrollTop = content.scrollHeight
    }
  })
}

onMounted(() => {
  if (inputRef.value) inputRef.value.focus()
})
</script>

<template>
  <div 
    ref="windowRef"
    class="terminal-window"
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
      <div class="window-title">Nemanja Samac — -zsh — 80x24</div>
      <div class="placeholder"></div>
    </div>

    <!-- Content -->
    <div class="terminal-content" @click="inputRef?.focus()">
      <div v-for="(item, index) in history" :key="index" class="history-item">
        <div v-if="item.type === 'input'" class="input-line">
          <span class="prompt">nemanja@samac ~ %</span>
          <span class="command">{{ item.content }}</span>
        </div>
        <div v-else class="output-line">
          <pre>{{ item.content }}</pre>
        </div>
      </div>
      
      <div class="input-line current">
        <span class="prompt">guest@samacos ~ %</span>
        <input 
          ref="inputRef"
          v-model="currentInput"
          @keydown.enter="executeCommand"
          @keydown="handleKeyDown"
          type="text" 
          class="terminal-input"
          spellcheck="false"
          autocomplete="off"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.terminal-window {
  position: absolute;
  width: 600px;
  height: 400px;
  background: rgba(30, 30, 30, 0.95);
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.5);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255,255,255,0.1);
  font-family: 'Menlo', 'Monaco', 'Courier New', monospace;
}

.terminal-window.maximized {
  width: 100% !important;
  height: calc(100vh - 24px) !important;
  border-radius: 0;
}

.title-bar {
  height: 28px;
  background: #2d2d2d;
  border-bottom: 1px solid #333;
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

.close { background: #ff5f56; }
.minimize { background: #ffbd2e; }
.maximize { background: #27c93f; }

.window-title {
  font-size: 13px;
  color: #999;
  font-weight: 500;
}

.placeholder {
  width: 60px;
}

.terminal-content {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
  background: #1e1e1e;
  color: #fff;
  font-size: 13px;
  line-height: 1.4;
  cursor: text;
}

.input-line {
  display: flex;
  align-items: center;
}

.prompt {
  color: #27c93f;
  margin-right: 8px;
  white-space: nowrap;
}

.terminal-input {
  background: transparent;
  border: none;
  color: #fff;
  font-family: inherit;
  font-size: inherit;
  flex: 1;
  outline: none;
  padding: 0;
  margin: 0;
}

.output-line pre {
  margin: 0;
  font-family: inherit;
  white-space: pre-wrap;
  color: #e0e0e0;
}

.history-item {
  margin-bottom: 2px;
}
</style>

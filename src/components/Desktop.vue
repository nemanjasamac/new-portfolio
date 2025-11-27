<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import Dock from './Dock.vue'
import PreviewWindow from './PreviewWindow.vue'
import AboutMeApp from './AboutMeApp.vue'
import ProjectWindow from './ProjectWindow.vue'
import TerminalApp from './TerminalApp.vue'
import ControlCenter from './ControlCenter.vue'
import SystemSettings from './SystemSettings.vue'
import ContactApp from './ContactApp.vue'
import Launchpad from './Launchpad.vue'
import CalculatorApp from './CalculatorApp.vue'
import NotesApp from './NotesApp.vue'
import MapsApp from './MapsApp.vue'
import folderIcon from '../assets/Icons/folder.png'
import safariIcon from '../assets/Icons/safari.svg'
import mailIcon from '../assets/Icons/mail.svg'

const emit = defineEmits(['reboot'])

const isPreviewOpen = ref(false)
const isAboutMeOpen = ref(false)
const isTerminalOpen = ref(false)
const isControlCenterOpen = ref(false)
const isSystemSettingsOpen = ref(false)
const isContactOpen = ref(false)
const isLaunchpadOpen = ref(false)
const isCalculatorOpen = ref(false)
const isNotesOpen = ref(false)
const isMapsOpen = ref(false)
const currentWallpaper = ref('/wallpaper.jpg')
const aboutMeInitialSection = ref('about')
const openProjects = ref([])
const previewFile = ref({ title: '', url: '' })

const projectsData = {
  'portfolio': {
    id: 'portfolio',
    title: 'Portfolio V1',
    subtitle: 'Personal Website',
    description: 'My first portfolio website built with Vue.js and MacOS inspired design. It features a fully functional desktop environment, window management, and a dock.',
    technologies: ['Vue.js', 'CSS3', 'Vite'],
    link: 'https://github.com/nemanjasamac/new-portfolio',
    github: 'https://github.com/nemanjasamac/new-portfolio',
    images: [
      'https://images.unsplash.com/photo-1498050108023-c5249f4df085?auto=format&fit=crop&w=800&q=80',
      'https://images.unsplash.com/photo-1504639725590-34d0984388bd?auto=format&fit=crop&w=800&q=80'
    ]
  },
  'ecommerce': {
    id: 'ecommerce',
    title: 'E-Commerce App',
    subtitle: 'Online Store',
    description: 'A full-featured e-commerce application with product catalog, shopping cart, and checkout process.',
    technologies: ['React', 'Node.js', 'MongoDB'],
    link: '#',
    github: '#',
    images: [
      'https://images.unsplash.com/photo-1557821552-17105176677c?auto=format&fit=crop&w=800&q=80',
      'https://images.unsplash.com/photo-1556742049-0cfed4f7a07d?auto=format&fit=crop&w=800&q=80'
    ]
  },
  'taskmanager': {
    id: 'taskmanager',
    title: 'Task Manager',
    subtitle: 'Productivity Tool',
    description: 'A task management application to help you stay organized. Features include drag-and-drop tasks, categories, and due dates.',
    technologies: ['Vue.js', 'Firebase', 'Tailwind'],
    link: '#',
    github: '#',
    images: [
      'https://images.unsplash.com/photo-1484480974693-6ca0a78fb36b?auto=format&fit=crop&w=800&q=80',
      'https://images.unsplash.com/photo-1540350394557-8d14678e7f91?auto=format&fit=crop&w=800&q=80'
    ]
  }
}

const currentTime = ref('')
const currentDate = ref('')
const isLangMenuOpen = ref(false)
const isFinderMenuOpen = ref(false)
const isAppleMenuOpen = ref(false)
const isContextMenuOpen = ref(false)
const contextMenuX = ref(0)
const contextMenuY = ref(0)
const isSelecting = ref(false)
const selectionStart = ref({ x: 0, y: 0 })
const selectionCurrent = ref({ x: 0, y: 0 })
const selectedLang = ref('A')
const languages = [
  { id: 'A', label: 'ABC', icon: 'A' },
  { id: 'US', label: 'U.S.', icon: 'US' },
  { id: 'SR', label: 'Serbian (Latin)', icon: 'SR' },
  { id: 'CP', label: 'Serbian', icon: 'CP' },
]
const desktopItems = ref([])
const isDragging = ref(false)
const dragOffsets = ref({})
let timer = null

// Easter Egg State
const showBSOD = ref(false)
const konamiCode = ['ArrowUp', 'ArrowUp', 'ArrowDown', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'ArrowLeft', 'ArrowRight', 'b', 'a']
const inputSequence = ref([])

const handleKeydown = (e) => {
  inputSequence.value.push(e.key)
  if (inputSequence.value.length > konamiCode.length) {
    inputSequence.value.shift()
  }
  
  if (JSON.stringify(inputSequence.value) === JSON.stringify(konamiCode)) {
    triggerBSOD()
    inputSequence.value = []
  }
}

const triggerBSOD = () => {
  showBSOD.value = true
  setTimeout(() => {
    showBSOD.value = false
  }, 5000)
}

const createNewFolder = () => {
  const id = Date.now()
  desktopItems.value.push({
    id,
    type: 'folder',
    name: 'New Folder',
    x: contextMenuX.value,
    y: contextMenuY.value,
    selected: false,
    deletable: true
  })
  isContextMenuOpen.value = false
}

const openProject = (projectId) => {
  const project = projectsData[projectId]
  if (project && !openProjects.value.find(p => p.id === projectId)) {
    openProjects.value.push({
      ...project,
      zIndex: 100 + openProjects.value.length + 1,
      x: 150 + (openProjects.value.length * 30),
      y: 100 + (openProjects.value.length * 30)
    })
  }
}

const closeProject = (projectId) => {
  openProjects.value = openProjects.value.filter(p => p.id !== projectId)
}

const focusProject = (projectId) => {
  const project = openProjects.value.find(p => p.id === projectId)
  if (project) {
    // Bring to front logic could be improved, but simple z-index bump works for now
    project.zIndex = Math.max(...openProjects.value.map(p => p.zIndex), 100) + 1
  }
}

const openItem = (item) => {
  if (item.type === 'resume') {
    previewFile.value = {
      title: 'Nemanja Samac CV.pdf',
      url: '/Nemanja Samac CV.pdf'
    }
    isPreviewOpen.value = true
  } else if (item.type === 'about-me') {
    aboutMeInitialSection.value = 'about'
    isAboutMeOpen.value = true
  } else if (item.type === 'project-folder') {
    openProject(item.projectId)
  } else if (item.type === 'projects-folder') {
    aboutMeInitialSection.value = 'projects'
    isAboutMeOpen.value = true
  } else if (item.type === 'contact-me') {
    isContactOpen.value = true
  }
}

const handleOpenApp = (appId) => {
  handleDockClick(appId)
}

const handleDockClick = (id) => {
  if (id === 'resume') {
    previewFile.value = {
      title: 'Nemanja Samac CV.pdf',
      url: '/Nemanja Samac CV.pdf'
    }
    isPreviewOpen.value = true
  } else if (id === 'finder' || id === 'finder-app') {
    aboutMeInitialSection.value = 'about'
    isAboutMeOpen.value = true
  } else if (id === 'projects') {
    aboutMeInitialSection.value = 'projects'
    isAboutMeOpen.value = true
  } else if (id === 'terminal') {
    isTerminalOpen.value = !isTerminalOpen.value
  } else if (id === 'launchpad') {
    isLaunchpadOpen.value = !isLaunchpadOpen.value
  } else if (id === 'mail') {
    isContactOpen.value = true
  } else if (id === 'settings') {
    isSystemSettingsOpen.value = true
  } else if (id === 'calculator') {
    isCalculatorOpen.value = true
  } else if (id === 'notes') {
    isNotesOpen.value = true
  } else if (id === 'maps') {
    isMapsOpen.value = true
  }
}

const handleItemMouseDown = (e, item) => {
  // Handle selection logic
  if (e.shiftKey || e.metaKey || e.ctrlKey) {
    item.selected = !item.selected
  } else {
    if (!item.selected) {
      desktopItems.value.forEach(i => i.selected = false)
      item.selected = true
    }
  }
  
  // Start dragging if selected
  if (item.selected) {
    isDragging.value = true
    dragOffsets.value = {}
    
    // Calculate offsets for all selected items
    desktopItems.value.filter(i => i.selected).forEach(i => {
      dragOffsets.value[i.id] = {
        x: e.clientX - i.x,
        y: e.clientY - i.y
      }
    })
  }
}

const updateTime = () => {
  const now = new Date()
  const dateOptions = { weekday: 'short', month: 'short', day: 'numeric' }
  const timeOptions = { hour: '2-digit', minute: '2-digit', hour12: false }
  
  // Format: Mon Nov 24
  const dateStr = now.toLocaleDateString('en-US', dateOptions).replace(',', '')
  // Format: 22:51
  const timeStr = now.toLocaleTimeString('en-US', timeOptions)
  
  currentDate.value = dateStr
  currentTime.value = timeStr
}

const toggleLangMenu = () => {
  isLangMenuOpen.value = !isLangMenuOpen.value
  if (isLangMenuOpen.value) {
    isFinderMenuOpen.value = false
    isAppleMenuOpen.value = false
    isContextMenuOpen.value = false
  }
}

const toggleFinderMenu = () => {
  isFinderMenuOpen.value = !isFinderMenuOpen.value
  if (isFinderMenuOpen.value) {
    isLangMenuOpen.value = false
    isAppleMenuOpen.value = false
    isContextMenuOpen.value = false
  }
}

const toggleAppleMenu = () => {
  isAppleMenuOpen.value = !isAppleMenuOpen.value
  if (isAppleMenuOpen.value) {
    isLangMenuOpen.value = false
    isFinderMenuOpen.value = false
    isContextMenuOpen.value = false
  }
}

const handleContextMenu = (e) => {
  if (e.target.closest('.menu-bar')) return
  e.preventDefault()
  isContextMenuOpen.value = true
  isLangMenuOpen.value = false
  isFinderMenuOpen.value = false
  isAppleMenuOpen.value = false
  
  // Prevent overflow
  const menuWidth = 220
  const menuHeight = 320
  
  let x = e.clientX
  let y = e.clientY
  
  if (x + menuWidth > window.innerWidth) {
    x = window.innerWidth - menuWidth - 10
  }
  
  if (y + menuHeight > window.innerHeight) {
    y = window.innerHeight - menuHeight - 10
  }
  
  contextMenuX.value = x
  contextMenuY.value = y
}

const closeContextMenu = () => {
  isContextMenuOpen.value = false
}

const handleMouseDown = (e) => {
  // Only start selection if clicking directly on desktop container or content area
  if (!e.target.classList.contains('desktop-container') && 
      !e.target.classList.contains('desktop-content')) return
      
  // Deselect all items
  desktopItems.value.forEach(i => i.selected = false)

  isSelecting.value = true
  selectionStart.value = { x: e.clientX, y: e.clientY }
  selectionCurrent.value = { x: e.clientX, y: e.clientY }
  
  // Close other menus
  isContextMenuOpen.value = false
  isLangMenuOpen.value = false
  isFinderMenuOpen.value = false
  isAppleMenuOpen.value = false
}

const handleMouseMove = (e) => {
  if (isDragging.value) {
    desktopItems.value.filter(i => i.selected).forEach(i => {
      const offset = dragOffsets.value[i.id]
      if (offset) {
        i.x = e.clientX - offset.x
        i.y = e.clientY - offset.y
      }
    })
    return
  }

  if (isSelecting.value) {
    selectionCurrent.value = { x: e.clientX, y: e.clientY }
    updateSelection()
  }
}

const updateSelection = () => {
  const left = Math.min(selectionStart.value.x, selectionCurrent.value.x)
  const top = Math.min(selectionStart.value.y, selectionCurrent.value.y)
  const right = Math.max(selectionStart.value.x, selectionCurrent.value.x)
  const bottom = Math.max(selectionStart.value.y, selectionCurrent.value.y)

  desktopItems.value.forEach(item => {
    // Item bounds (approximate)
    const itemLeft = item.x
    const itemRight = item.x + 80
    const itemTop = item.y
    const itemBottom = item.y + 90

    const isIntersecting = !(itemLeft > right || 
                           itemRight < left || 
                           itemTop > bottom || 
                           itemBottom < top)
    
    item.selected = isIntersecting
  })
}

const handleMouseUp = (e) => {
  if (isDragging.value) {
    // Check if dropped on trash
    const trashEl = document.querySelector('.trash-drop-zone')
    if (trashEl) {
      const rect = trashEl.getBoundingClientRect()
      // Check if mouse is within trash bounds
      if (e.clientX >= rect.left && e.clientX <= rect.right &&
          e.clientY >= rect.top && e.clientY <= rect.bottom) {
        
        // Remove deletable items
        desktopItems.value = desktopItems.value.filter(item => {
          // If selected and deletable, remove it (return false)
          if (item.selected && item.deletable) {
            return false
          }
          return true
        })
      }
    }
  }

  isSelecting.value = false
  isDragging.value = false
  dragOffsets.value = {}
}

const selectionBoxStyle = computed(() => {
  const left = Math.min(selectionStart.value.x, selectionCurrent.value.x)
  const top = Math.min(selectionStart.value.y, selectionCurrent.value.y)
  const width = Math.abs(selectionCurrent.value.x - selectionStart.value.x)
  const height = Math.abs(selectionCurrent.value.y - selectionStart.value.y)
  
  return {
    left: `${left}px`,
    top: `${top}px`,
    width: `${width}px`,
    height: `${height}px`
  }
})

const selectLang = (lang) => {
  selectedLang.value = lang
  isLangMenuOpen.value = false
}

const reloadPage = () => {
    emit('reboot')
}

const shutDown = () => {
    
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  console.log('%c Looking for secrets? Try the Konami Code! ', 'background: #222; color: #bada55; font-size: 14px; padding: 4px;')
  
  updateTime()
  timer = setInterval(updateTime, 1000)
  
  // Close menu when clicking outside
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.lang-container')) {
      isLangMenuOpen.value = false
    }
    if (!e.target.closest('.finder-container')) {
      isFinderMenuOpen.value = false
    }
    if (!e.target.closest('.apple-container')) {
      isAppleMenuOpen.value = false
    }
    if (!e.target.closest('.context-menu')) {
      isContextMenuOpen.value = false
    }
  })

  // Add default items
  const leftX = 20
  
  const defaultItems = [
    {
      id: 'about-me',
      type: 'about-me',
      name: 'About Me',
      x: leftX,
      y: 20,
      selected: false,
      deletable: false
    },
    {
      id: 'projects-folder',
      type: 'projects-folder',
      name: 'My Projects',
      x: leftX,
      y: 110,
      selected: false,
      deletable: false
    },
    {
      id: 'safari',
      type: 'safari',
      name: 'Safari',
      x: leftX,
      y: 200,
      selected: false,
      deletable: false
    },
    {
      id: 'resume',
      type: 'resume',
      name: 'My Resume',
      x: leftX,
      y: 290,
      selected: false,
      deletable: false
    },
    {
      id: 'contact-me',
      type: 'contact-me',
      name: 'Contact Me',
      x: leftX,
      y: 380,
      selected: false,
      deletable: false
    }
  ]

  defaultItems.forEach(item => {
    if (!desktopItems.value.find(i => i.id === item.id)) {
      desktopItems.value.push(item)
    }
  })
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  if (timer) clearInterval(timer)
})
</script>

<template>
  <div 
    class="desktop-container" 
    :style="{ backgroundImage: `url('${currentWallpaper}')` }"
    @contextmenu="handleContextMenu" 
    @click="closeContextMenu"
    @mousedown="handleMouseDown"
    @mousemove="handleMouseMove"
    @mouseup="handleMouseUp"
  >
    <!-- BSOD Easter Egg -->
    <div v-if="showBSOD" class="bsod-overlay">
        <div class="bsod-content">
            <div class="sad-face">:(</div>
            <div class="bsod-text">
                Your PC ran into a problem and needs to restart. We're just collecting some error info, and then we'll restart for you.
            </div>
            <div class="bsod-progress">20% complete</div>
            <div class="bsod-details">
                <div class="qr-code">
                    <svg viewBox="0 0 100 100" width="100" height="100" fill="white">
                        <rect x="10" y="10" width="30" height="30"/>
                        <rect x="60" y="10" width="30" height="30"/>
                        <rect x="10" y="60" width="30" height="30"/>
                        <rect x="50" y="50" width="10" height="10"/>
                        <rect x="70" y="70" width="10" height="10"/>
                        <rect x="50" y="70" width="10" height="10"/>
                        <rect x="70" y="50" width="10" height="10"/>
                    </svg>
                </div>
                <div class="stop-code">
                    For more information about this issue and possible fixes, visit https://www.windows.com/stopcode
                    <br><br>
                    If you call a support person, give them this info:
                    <br>
                    Stop code: CRITICAL_PROCESS_DIED
                </div>
            </div>
        </div>
    </div>

    <!-- Selection Box -->
    <div 
      v-if="isSelecting" 
      class="selection-box"
      :style="selectionBoxStyle"
    ></div>

    <!-- Context Menu -->
    <div 
      v-if="isContextMenuOpen" 
      class="context-menu"
      :style="{ top: `${contextMenuY}px`, left: `${contextMenuX}px` }"
    >
      <div class="menu-item-row" @click="createNewFolder">
        <div class="menu-label">New Folder</div>
      </div>
      
      <div class="menu-separator"></div>
      
      <div class="menu-item-row">
        <div class="menu-label">Get Info</div>
      </div>
      <div class="menu-item-row" @click="isSystemSettingsOpen = true; isContextMenuOpen = false">
        <div class="menu-label">Change Wallpaper...</div>
      </div>
      <div class="menu-item-row disabled">
        <div class="menu-label">Edit Widgets...</div>
      </div>
      
      <div class="menu-separator"></div>
      
      <div class="menu-item-row disabled">
        <div class="menu-label">Use Stacks</div>
      </div>
      <div class="menu-item-row disabled">
        <div class="menu-label">Sort By</div>
        <div class="submenu-arrow">›</div>
        <div class="submenu">
          <div class="menu-item-row">
            <div class="menu-label">Name</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Kind</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Date Last Opened</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Date Added</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Date Modified</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Date Created</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Size</div>
          </div>
          <div class="menu-item-row">
            <div class="menu-label">Tags</div>
          </div>
        </div>
      </div>
      <div class="menu-item-row disabled">
        <div class="menu-label">Clean Up</div>
      </div>
      <div class="menu-item-row disabled">
        <div class="menu-label">Clean Up By</div>
      </div>
    </div>

    <!-- Top Menu Bar -->
    <div class="menu-bar">
      <div class="left-menu">
        <div class="apple-container relative-container">
          <div class="apple-icon" :class="{ active: isAppleMenuOpen }" @click="toggleAppleMenu">
            <svg viewBox="0 0 170 170" width="20" height="20" xmlns="http://www.w3.org/2000/svg">
              <path fill="white" d="M150.37 130.25c-2.45 5.66-5.35 10.87-8.71 15.66-4.93 7.03-10.03 14.05-17.79 14.05-7.79 0-10.03-4.63-18.71-4.63-8.68 0-11.38 4.63-18.71 4.63-7.76 0-12.86-7.03-17.79-14.05-9.67-13.76-17.06-38.89-7.13-56.13 4.93-8.56 13.76-13.99 23.38-13.99 7.37 0 14.32 4.96 18.77 4.96 4.45 0 12.8-4.96 21.58-4.96 3.52 0 15.66 0.28 23.05 11.08-0.59 0.34-13.82 8.06-13.82 23.17 0 18.45 16.17 24.61 16.34 24.7-0.12 0.37-2.55 8.74-8.46 15.51zM110.33 52.99c4.02-4.87 6.73-11.65 5.99-18.39-5.78 0.23-12.78 3.85-16.93 8.71-3.7 4.28-6.94 11.11-6.08 17.66 6.46 0.5 13.01-3.11 17.02-7.98z"/>
            </svg>
          </div>

          <div v-if="isAppleMenuOpen" class="dropdown-menu apple-dropdown">
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect><line x="8" y="21" x2="16" y2="21"></line><line x="12" y1="17" x2="12" y2="21"></line></svg>
              </span>
              <span class="menu-label">About samacOS</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row" @click="isSystemSettingsOpen = true; isAppleMenuOpen = false">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>
              </span>
              <span class="menu-label">System Settings...</span>
              <span class="badge">1 update</span>
            </div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20.9l-9.9-15.6h19.8L12 20.9z"></path><line x1="7" y1="13" x2="17" y2="13"></line></svg>
              </span>
              <span class="menu-label">App Store...</span>
              <span class="badge">6 updates</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row has-submenu">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></svg>
              </span>
              <span class="menu-label">Recent Items</span>
              <svg class="submenu-arrow" width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"></polyline></svg>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg>
              </span>
              <span class="menu-label">Force Quit...</span>
              <span class="shortcut">⌥⌘⎋</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="12" x2="12" y2="18"></line></svg>
              </span>
              <span class="menu-label">Sleep</span>
            </div>
            <div class="menu-item-row" @click="reloadPage()">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="currentColor" stroke="none"><polygon points="18 20 8 12 18 4 18 20"></polygon></svg>
              </span>
              <span class="menu-label">Restart...</span>
            </div>
            <div class="menu-item-row">
              <span class="menu-icon-small" @click="shutDown()">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18.36 6.64a9 9 0 1 1-12.73 0"></path><line x1="12" y1="2" x2="12" y2="12"></line></svg>
              </span>
              <span class="menu-label">Shut Down...</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
              </span>
              <span class="menu-label">Lock Screen</span>
              <span class="shortcut">^⌘Q</span>
            </div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
              </span>
              <span class="menu-label">Log Out Nemanja Samac...</span>
              <span class="shortcut">⇧⌘Q</span>
            </div>
          </div>
        </div>
        
        <div class="finder-container relative-container">
          <span class="menu-item bold finder-trigger" :class="{ active: isFinderMenuOpen }" @click="toggleFinderMenu">Finder</span>
          
          <div v-if="isFinderMenuOpen" class="dropdown-menu finder-dropdown">
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x="12" y1="16" x2="12" y2="12"></line><line x="12" y1="8" x2="12.01" y2="8"></line></svg>
              </span>
              <span class="menu-label">About Me</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"></circle><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path></svg>
              </span>
              <span class="menu-label">Settings...</span>
              <span class="shortcut">⌘,</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
              </span>
              <span class="menu-label">Empty Trash...</span>
              <span class="shortcut">⇧⌘⌫</span>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row has-submenu">
              <span class="menu-icon-small"></span>
              <span class="menu-label">Services</span>
              <svg class="submenu-arrow" width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"></polyline></svg>
              <div class="submenu">
                <div class="submenu-header">Development</div>
                <div class="menu-item-row">
                  <span class="menu-icon-small">⏱️</span>
                  <span class="menu-label">Activity Monitor</span>
                </div>
                <div class="menu-item-row">
                  <span class="menu-icon-small">📊</span>
                  <span class="menu-label">Allocations & Leaks</span>
                </div>
                <div class="menu-item-row">
                  <span class="menu-icon-small">📁</span>
                  <span class="menu-label">File Activity</span>
                </div>
                <div class="menu-item-row">
                  <span class="menu-icon-small">🔍</span>
                  <span class="menu-label">System Trace</span>
                </div>
                <div class="menu-separator"></div>
                <div class="menu-item-row">
                  <span class="menu-icon-small">⚙️</span>
                  <span class="menu-label">Services Settings...</span>
                </div>
              </div>
            </div>
            <div class="menu-separator"></div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg>
              </span>
              <span class="menu-label">Hide Finder</span>
              <span class="shortcut">⌘H</span>
            </div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg>
              </span>
              <span class="menu-label">Hide Others</span>
              <span class="shortcut">⌥⌘H</span>
            </div>
            <div class="menu-item-row">
              <span class="menu-icon-small">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
              </span>
              <span class="menu-label">Show All</span>
            </div>
          </div>
        </div>

        <span class="menu-item">File</span>
        <span class="menu-item">Edit</span>
        <span class="menu-item">View</span>
        <span class="menu-item">Go</span>
        <span class="menu-item">Window</span>
        <span class="menu-item">Help</span>
      </div>
      
      <div class="right-menu">
        <!-- Language Switcher -->
        <div class="status-item lang-container">
          <span class="lang-icon" @click="toggleLangMenu">{{ selectedLang }}</span>
          <div v-if="isLangMenuOpen" class="lang-dropdown">
            <div class="lang-option" v-for="lang in languages" :key="lang.id" @click="selectLang(lang.id)">
              <div class="check-col">
                <svg v-if="selectedLang === lang.id" width="10" height="10" viewBox="0 0 12 12" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 6L4.5 8.5L10 3"/></svg>
              </div>
              <div class="lang-icon-box">{{ lang.icon }}</div>
              <span class="lang-label">{{ lang.label }}</span>
            </div>
            
            <div class="menu-separator"></div>
            
            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><line x="9" y="3" x2="9" y2="21"></line></svg>
              </div>
              <span class="menu-label">Show Emoji & Symbols</span>
            </div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="2" y="4" width="20" height="16" rx="2"></rect><line x="6" y1="8" x2="6.01" y2="8"></line><line x="10" y1="8" x2="10.01" y2="8"></line><line x="14" y1="8" x2="14.01" y2="8"></line><line x="18" y1="8" x2="18.01" y2="8"></line><line x="6" y1="12" x2="6.01" y2="12"></line><line x="10" y1="12" x2="10.01" y2="12"></line><line x="14" y1="12" x2="14.01" y2="12"></line><line x="18" y1="12" x2="18.01" y2="12"></line><line x="7" y1="16" x2="17" y2="16"></line></svg>
              </div>
              <span class="menu-label">Show Keyboard Viewer</span>
            </div>

            <div class="menu-separator"></div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon"></div>
              <span class="menu-label">Show Input Source Name</span>
            </div>

            <div class="menu-separator"></div>

            <div class="menu-item">
              <div class="check-col"></div>
              <div class="menu-icon"></div>
              <span class="menu-label">Open Keyboard Settings...</span>
            </div>
          </div>
        </div>

        <!-- Battery -->
        <div class="status-item">
          <div class="battery-group">
          <span class="battery-text">100%</span>
          <svg class="icon" width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <rect x="3.25" y="7.25" width="16.5" height="9.5" rx="2.5" stroke="currentColor" stroke-width="1.5" stroke-opacity="0.4"/>
            <path d="M22 10.5V13.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-opacity="0.4"/>
            <rect x="5.5" y="9.5" width="12" height="5" rx="1" fill="currentColor"/>
          </svg>
        </div>
        </div>

        <!-- Wifi -->
        <div class="status-item">
          <svg class="icon" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg>
        </div>

        <!-- Search -->
        <div class="status-item">
          <svg class="icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
        </div>

        <!-- Control Center -->
        <div class="status-item" @click="isControlCenterOpen = !isControlCenterOpen">
          <svg class="icon" width="18" height="18" viewBox="0 0 29 29" xmlns="http://www.w3.org/2000/svg" fill="currentColor">
            <path d="M7.5,13h14a5.5,5.5,0,0,0,0-11H7.5a5.5,5.5,0,0,0,0,11Zm0-9h14a3.5,3.5,0,0,1,0,7H7.5a3.5,3.5,0,0,1,0-7Zm0,6A2.5,2.5,0,1,0,5,7.5,2.5,2.5,0,0,0,7.5,10Zm14,6H7.5a5.5,5.5,0,0,0,0,11h14a5.5,5.5,0,0,0,0-11Zm1.43439,8a2.5,2.5,0,1,1,2.5-2.5A2.5,2.5,0,0,1,22.93439,24Z"></path>
          </svg>
        </div>

        <!-- Date Time -->
        <div class="status-item date-time">
          {{ currentDate }} &nbsp; {{ currentTime }}
        </div>
      </div>
    </div>
    
    <ControlCenter :is-open="isControlCenterOpen" @close="isControlCenterOpen = false" />

    <!-- Desktop Content Area -->
    <div class="desktop-content" @mousedown="handleMouseDown" @mousemove="handleMouseMove" @mouseup="handleMouseUp">
      <!-- Windows -->
      <PreviewWindow 
        v-if="isPreviewOpen"
        :title="previewFile.title"
        :file-url="previewFile.url"
        @close="isPreviewOpen = false"
      />

      <AboutMeApp 
        v-if="isAboutMeOpen"
        :initial-section="aboutMeInitialSection"
        @close="isAboutMeOpen = false"
        @open-project="openProject"
      />

      <TerminalApp 
        v-if="isTerminalOpen"
        @close="isTerminalOpen = false"
        @open-app="handleOpenApp"
      />

      <SystemSettings 
        :is-open="isSystemSettingsOpen"
        :current-wallpaper="currentWallpaper"
        @close="isSystemSettingsOpen = false"
        @change-wallpaper="(url) => currentWallpaper = url"
      />

      <ContactApp
        :is-open="isContactOpen"
        @close="isContactOpen = false"
      />

      <ProjectWindow
        v-for="project in openProjects"
        :key="project.id"
        :project="project"
        :initial-x="project.x"
        :initial-y="project.y"
        :z-index="project.zIndex"
        @close="closeProject(project.id)"
        @focus="focusProject(project.id)"
      />

      <!-- Windows and icons will go here -->
      <div 
        v-for="item in desktopItems" 
        :key="item.id"
        class="desktop-item"
        :class="{ selected: item.selected }"
        :style="{ left: `${item.x}px`, top: `${item.y}px` }"
        @mousedown.stop="handleItemMouseDown($event, item)"
        @dblclick="openItem(item)"
      >
        <div class="item-icon">
          <img v-if="item.type === 'folder' || item.type === 'project-folder' || item.type === 'projects-folder'" :src="folderIcon" width="50" height="50" draggable="false" />
          
          <!-- About Me -->
          <svg v-else-if="item.type === 'about-me'" viewBox="0 0 100 100" width="50" height="50">
            <rect x="5" y="5" width="90" height="90" rx="22" fill="#ececec"/>
            <rect x="5" y="5" width="90" height="90" rx="22" fill="url(#finder-grad)" fill-opacity="0.8"/>
            <path d="M50 94c24.3 0 44-19.7 44-44S74.3 6 50 6 6 25.7 6 50s19.7 44 44 44z" fill="none"/>
            <path d="M28 35c0-5 8-8 22-8s22 3 22 8v30c0 15-44 15-44 0V35z" fill="#007AFF" opacity="0.1"/>
            <path d="M33 32c0 0 5-4 17-4s17 4 17 4" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <path d="M33 45c0 0 5 5 17 5s17-5 17-5" stroke="#333" stroke-width="3" stroke-linecap="round" fill="none"/>
            <line x1="50" y1="45" x2="50" y2="58" stroke="#333" stroke-width="3" stroke-linecap="round"/>
            <defs>
              <linearGradient id="finder-grad" x1="0%" y1="0%" x2="0%" y2="100%">
                <stop offset="0%" style="stop-color:#f9f9f9;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#c7c7c7;stop-opacity:1" />
              </linearGradient>
            </defs>
          </svg>

          <!-- Safari -->
          <img v-else-if="item.type === 'safari'" :src="safariIcon" width="50" height="50" draggable="false" />

          <!-- Resume -->
          <svg v-else-if="item.type === 'resume'" viewBox="0 0 100 100" width="50" height="50">
            <rect x="15" y="5" width="70" height="90" rx="8" fill="#fff"/>
            <path d="M15 25h70" stroke="#ddd" stroke-width="1"/>
            <rect x="25" y="35" width="50" height="4" rx="2" fill="#e0e0e0"/>
            <rect x="25" y="45" width="50" height="4" rx="2" fill="#e0e0e0"/>
            <rect x="25" y="55" width="35" height="4" rx="2" fill="#e0e0e0"/>
            <rect x="25" y="70" width="50" height="4" rx="2" fill="#e0e0e0"/>
            <rect x="25" y="80" width="40" height="4" rx="2" fill="#e0e0e0"/>
            <circle cx="70" cy="20" r="8" fill="#FF9500"/>
          </svg>

          <!-- Contact Me -->
          <img v-else-if="item.type === 'contact-me'" :src="mailIcon" width="50" height="50" draggable="false" />

          <svg v-else-if="item.type === 'drive'" viewBox="0 0 100 100" width="50" height="50">
            <rect x="15" y="35" width="70" height="40" rx="5" fill="#ccc" stroke="#999" stroke-width="2"/>
            <rect x="20" y="60" width="60" height="5" fill="#999"/>
            <circle cx="80" cy="45" r="3" fill="#0f0"/>
          </svg>
          <svg v-else-if="item.type === 'file'" viewBox="0 0 100 100" width="50" height="50">
            <path d="M25 10 L60 10 L75 25 L75 90 L25 90 Z" fill="white" stroke="#ccc" stroke-width="2"/>
            <path d="M60 10 L60 25 L75 25" fill="#eee" stroke="#ccc" stroke-width="2"/>
            <line x1="35" y1="40" x2="65" y2="40" stroke="#ccc" stroke-width="2"/>
            <line x1="35" y1="50" x2="65" y2="50" stroke="#ccc" stroke-width="2"/>
            <line x1="35" y1="60" x2="65" y2="60" stroke="#ccc" stroke-width="2"/>
          </svg>
        </div>
        <div class="item-name">{{ item.name }}</div>
      </div>

      <!-- Selection Box -->
      <div v-if="isSelecting" class="selection-box" :style="selectionBoxStyle"></div>
    </div>

    <!-- Launchpad -->
    <Transition name="launchpad">
      <Launchpad v-if="isLaunchpadOpen" :is-open="isLaunchpadOpen" @close="isLaunchpadOpen = false" @open-app="handleDockClick" />
    </Transition>

    <!-- Calculator App -->
    <CalculatorApp 
      :is-open="isCalculatorOpen" 
      :z-index="200"
      @close="isCalculatorOpen = false"
      @focus="() => {}"
    />

    <!-- Notes App -->
    <NotesApp 
      :is-open="isNotesOpen" 
      :z-index="200"
      @close="isNotesOpen = false"
      @focus="() => {}"
    />

    <!-- Maps App -->
    <MapsApp 
      :is-open="isMapsOpen" 
      :z-index="200"
      @close="isMapsOpen = false"
      @focus="() => {}"
    />

    <!-- Dock Component -->
    <Dock @open-app="handleDockClick" />
  </div>
</template>

<style scoped>
*{
  user-select: none;
}

.desktop-container {
  width: 100vw;
  height: 100vh;
  background-repeat: no-repeat;
  background-position: center center;
  background-size: cover;
  overflow: hidden;
  position: relative;
  transition: background-image 0.5s ease-in-out;
}

.menu-bar {
  width: 100%;
  height: 24px;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 12px;
  box-sizing: border-box;
  color: white;
  font-size: 13px;
  user-select: none;
  z-index: 1000;
  position: relative;
}

.left-menu, .right-menu {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.left-menu {
  gap: 18px;
}

.right-menu {
  gap: 16px;
}

.apple-icon {
  display: flex;
  align-items: center;
  opacity: 0.9;
  padding: 4px 10px;
  border-radius: 4px;
  margin-left: -10px;
}

.apple-icon.active {
  background: rgba(255,255,255,0.2);
}

.apple-dropdown {
  left: -5px;
  min-width: 260px;
}

.badge {
  background: rgba(255, 255, 255, 0.2);
  padding: 1px 8px;
  border-radius: 10px;
  font-size: 11px;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 500;
}

.menu-item-row:hover .badge {
  color: white;
  background: rgba(255, 255, 255, 0.3);
}

.menu-item {
  font-weight: 500;
  opacity: 0.9;
  cursor: var(--mac-cursor);
  padding: 4px 10px;
}

.menu-item.bold {
  font-weight: 700;
}

.status-item {
  display: flex;
  align-items: center;
  opacity: 0.9;
  cursor: var(--mac-cursor);
}

.relative-container {
  position: relative;
  height: 100%;
  display: flex;
  align-items: center;
}

.finder-trigger {
  padding: 4px 10px;
  border-radius: 4px;
  margin-left: -10px;
}

.finder-trigger.active {
  background: rgba(255,255,255,0.2);
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  margin-top: 4px;
  background: rgba(30, 30, 30, 0.6);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 10px;
  padding: 5px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 15px 30px rgba(0,0,0,0.4);
  z-index: 11000;
  display: flex;
  flex-direction: column;
}

.finder-dropdown {
  left: -10px;
  min-width: 240px;
}

.menu-item-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 4px 10px;
  font-size: 13px;
  color: white;
  cursor: var(--mac-cursor);
  border-radius: 5px;
  height: 20px;
  white-space: nowrap;
}

.menu-item-row:hover {
  background: #007AFF;
}

.shortcut {
  color: rgba(255,255,255,0.5);
  font-size: 12px;
  margin-left: 15px;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}

.menu-item-row:hover .shortcut {
  color: rgba(255,255,255,0.9);
}

.has-submenu {
  justify-content: space-between;
  position: relative;
}

.submenu-arrow {
  opacity: 0.7;
}

.submenu {
  position: absolute;
  left: 100%;
  top: -6px;
  display: none;
  background: rgba(30, 30, 30, 0.6);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 10px;
  padding: 5px;
  min-width: 220px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 15px 30px rgba(0,0,0,0.4);
}

.submenu-header {
  padding: 4px 10px;
  font-size: 12px;
  color: rgba(255,255,255,0.5);
  font-weight: 600;
  cursor: default;
}

.menu-icon-small {
  margin-right: 8px;
  font-size: 14px;
  width: 16px;
  text-align: center;
}

.has-submenu:hover .submenu {
  display: flex;
  flex-direction: column;
}

.menu-item.active {
  background: rgba(255,255,255,0.3);
  border-radius: 50px;
  padding: 4px 10px;
}

.lang-container {
  position: relative;
}

.lang-icon {
  font-size: 11px;
  border: 1.5px solid rgba(255,255,255,0.9);
  padding: 0px 3px;
  border-radius: 4px;
  font-weight: 600;
  min-width: 16px;
  text-align: center;
  display: inline-block;
  cursor: var(--mac-cursor);
}

.lang-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 8px;
  background: rgba(30, 30, 30, 0.6);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 8px;
  padding: 5px;
  min-width: 240px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 15px 30px rgba(0,0,0,0.3);
  z-index: 200;
  display: flex;
  flex-direction: column;
}

.lang-option, .menu-item {
  display: flex;
  align-items: center;
  padding: 3px 10px 3px 4px;
  font-size: 13px;
  color: white;
  cursor: var(--mac-cursor);
  border-radius: 4px;
  height: 24px;
}

.menu-item {
  display: flex;
  align-items: center;
  padding: 4px 10px;
  font-size: 13px;
  color: white;
  cursor: var(--mac-cursor);
  border-radius: 4px;
  height: 24px;
}

.lang-option:hover {
  background: #007AFF;
}

.check-col {
  width: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 4px;
}

.lang-icon-box {
  width: 16px;
  height: 16px;
  background: #f0f0f0;
  border-radius: 3px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 9px;
  font-weight: 700;
  color: #333;
  margin-right: 8px;
  box-shadow: 0 0.5px 1px rgba(0,0,0,0.2);
}

.menu-icon {
  width: 16px;
  height: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-right: 8px;
  color: #fff;
}

.lang-label, .menu-label {
  flex: 1;
  white-space: nowrap;
  font-weight: 400;
}

.menu-separator {
  height: 1px;
  background: rgba(255, 255, 255, 0.15);
  margin: 4px 10px;
}

.date-time {
  font-weight: 500;
  margin-left: 4px;
}

.battery-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.context-menu {
  position: fixed;
  background: rgba(30, 30, 30, 0.6);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 10px;
  padding: 5px;
  min-width: 220px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 15px 30px rgba(0,0,0,0.4);
  z-index: 9999;
  display: flex;
  flex-direction: column;
}

/* BSOD Styles */
.bsod-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: #0078D7;
    z-index: 999999;
    color: white;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    display: flex;
    align-items: center;
    padding-left: 15%;
    cursor: none;
    overflow: hidden;
    box-sizing: border-box;
}

.bsod-content {
    max-width: 800px;
}

.sad-face {
    font-size: 120px;
    margin-bottom: 40px;
}

.bsod-text {
    font-size: 24px;
    line-height: 1.4;
    margin-bottom: 30px;
}

.bsod-progress {
    font-size: 24px;
    margin-bottom: 40px;
}

.bsod-details {
    display: flex;
    align-items: center;
    gap: 20px;
}

.stop-code {
    font-size: 14px;
    line-height: 1.5;
}

.menu-item-row.disabled {
  color: rgba(255, 255, 255, 0.3);
}

.menu-item-row.disabled:hover {
  background: transparent;
}

.selection-box {
  position: fixed;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 4px;
  pointer-events: none;
  z-index: 100;
}

.desktop-item {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 80px;
  padding: 5px;
  border-radius: 5px;
  cursor: var(--mac-cursor);
  border: 1px solid transparent;
}

.desktop-item.selected {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.desktop-item.selected .item-icon {
  background: transparent;
  border: none;
}

.item-icon {
  margin-bottom: 4px;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.3));
}

.item-name {
  color: white;
  font-size: 12px;
  text-align: center;
  text-shadow: 0 1px 2px rgba(0,0,0,0.8);
  word-break: break-word;
  line-height: 1.2;
  padding: 2px 4px;
  border-radius: 3px;
}

.desktop-item.selected .item-name {
  background: #0058d0;
  color: white;
}

/* Launchpad Transition */
.launchpad-enter-active,
.launchpad-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.launchpad-enter-from,
.launchpad-leave-to {
  opacity: 0;
  transform: scale(1.1);
}
</style>

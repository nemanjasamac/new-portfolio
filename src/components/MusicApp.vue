<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  isOpen: Boolean,
  zIndex: {
    type: Number,
    default: 100
  },
  currentSong: Object,
  isPlaying: Boolean,
  songs: Array
})

const emit = defineEmits(['close', 'focus', 'toggle-play', 'next-song', 'prev-song', 'play-song'])

const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: 150, y: 80 })
const isMaximized = ref(false)
const preMaximizePosition = ref({ x: 0, y: 0 })

const sidebarItems = [
  { id: 'listen_now', label: 'Listen Now', icon: 'play-circle' },
  { id: 'browse', label: 'Browse', icon: 'grid' },
  { id: 'radio', label: 'Radio', icon: 'radio' },
]

const libraryItems = [
  { id: 'recently_added', label: 'Recently Added', icon: 'clock' },
  { id: 'artists', label: 'Artists', icon: 'mic' },
  { id: 'albums', label: 'Albums', icon: 'disc' },
  { id: 'songs', label: 'Songs', icon: 'music' },
]

const playlists = [
  { id: 'favorites', label: 'Favorites' },
  { id: 'chill', label: 'Chill Mix' },
  { id: 'workout', label: 'Workout' },
]

const selectedItem = ref('listen_now')

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.sidebar') || e.target.closest('.player-controls') || e.target.closest('.song-list')) return
  
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

const formatTime = (seconds) => {
  if (!seconds) return '0:00'
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
}
</script>

<template>
  <div 
    v-if="isOpen"
    class="music-window"
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
      
      <div class="search-container">
        <div class="search-input-wrapper">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>
            <input type="text" placeholder="Search" />
        </div>
      </div>

      <div class="sidebar-section">
        <div 
          v-for="item in sidebarItems" 
          :key="item.id"
          class="sidebar-item"
          :class="{ active: selectedItem === item.id }"
          @click="selectedItem = item.id"
        >
          <span class="item-icon">
            <svg v-if="item.icon === 'play-circle'" viewBox="0 0 24 24" fill="currentColor"><circle cx="12" cy="12" r="10"/><polygon points="10 8 16 12 10 16 10 8" fill="white"/></svg>
            <svg v-if="item.icon === 'grid'" viewBox="0 0 24 24" fill="currentColor"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
            <svg v-if="item.icon === 'radio'" viewBox="0 0 24 24" fill="currentColor"><circle cx="12" cy="12" r="2"/><path d="M16.24 7.76a6 6 0 0 1 0 8.49m-8.48-.01a6 6 0 0 1 0-8.49m11.31-2.82a10 10 0 0 1 0 14.14m-14.14 0a10 10 0 0 1 0-14.14"/></svg>
          </span>
          {{ item.label }}
        </div>
      </div>

      <div class="sidebar-section">
        <div class="section-title">Library</div>
        <div 
          v-for="item in libraryItems" 
          :key="item.id"
          class="sidebar-item"
          :class="{ active: selectedItem === item.id }"
          @click="selectedItem = item.id"
        >
          <span class="item-icon">
            <svg v-if="item.icon === 'clock'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
            <svg v-if="item.icon === 'mic'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" y1="19" x2="12" y2="23"/><line x1="8" y1="23" x2="16" y2="23"/></svg>
            <svg v-if="item.icon === 'disc'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="3"/></svg>
            <svg v-if="item.icon === 'music'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 18V5l12-2v13"/><circle cx="6" cy="18" r="3"/><circle cx="18" cy="16" r="3"/></svg>
          </span>
          {{ item.label }}
        </div>
      </div>

      <div class="sidebar-section">
        <div class="section-title">Playlists</div>
        <div 
          v-for="playlist in playlists" 
          :key="playlist.id"
          class="sidebar-item"
          :class="{ active: selectedItem === playlist.id }"
          @click="selectedItem = playlist.id"
        >
          <span class="item-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 18V5l12-2v13"/><circle cx="6" cy="18" r="3"/><circle cx="18" cy="16" r="3"/></svg>
          </span>
          {{ playlist.label }}
        </div>
      </div>
    </div>

    <div class="main-content">
      <div class="player-bar">
        <div class="player-controls">
            <button class="player-btn" @click="$emit('prev-song')">
                <svg viewBox="0 0 24 24" fill="currentColor"><polygon points="19 20 9 12 19 4 19 20"/><line x1="5" y1="19" x2="5" y2="5" stroke="currentColor" stroke-width="2"/></svg>
            </button>
            <button class="player-btn play-pause" @click="$emit('toggle-play')">
                <svg v-if="!isPlaying" viewBox="0 0 24 24" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"/></svg>
                <svg v-else viewBox="0 0 24 24" fill="currentColor"><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>
            </button>
            <button class="player-btn" @click="$emit('next-song')">
                <svg viewBox="0 0 24 24" fill="currentColor"><polygon points="5 4 15 12 5 20 5 4"/><line x1="19" y1="5" x2="19" y2="19" stroke="currentColor" stroke-width="2"/></svg>
            </button>
        </div>
        
        <div class="now-playing-info">
            <div class="apple-logo-container">
                <svg viewBox="0 0 170 170" width="16" height="16" fill="#333">
                    <path d="M150.37 130.25c-2.45 5.66-5.35 10.87-8.71 15.66-4.93 7.03-10.03 14.05-17.79 14.05-7.79 0-10.03-4.63-18.71-4.63-8.68 0-11.38 4.63-18.71 4.63-7.76 0-12.86-7.03-17.79-14.05-9.67-13.76-17.06-38.89-7.13-56.13 4.93-8.56 13.76-13.99 23.38-13.99 7.37 0 14.32 4.96 18.77 4.96 4.45 0 12.8-4.96 21.58-4.96 3.52 0 15.66 0.28 23.05 11.08-0.59 0.34-13.82 8.06-13.82 23.17 0 18.45 16.17 24.61 16.34 24.7-0.12 0.37-2.55 8.74-8.46 15.51zM110.33 52.99c4.02-4.87 6.73-11.65 5.99-18.39-5.78 0.23-12.78 3.85-16.93 8.71-3.7 4.28-6.94 11.11-6.08 17.66 6.46 0.5 13.01-3.11 17.02-7.98z"/>
                </svg>
            </div>
            <div class="song-display" v-if="currentSong">
                <span class="song-title">{{ currentSong.title }}</span>
                <span class="song-artist">{{ currentSong.artist }}</span>
            </div>
            <div class="song-display" v-else>
                <span class="song-title">samacOS Music</span>
            </div>
        </div>

        <div class="volume-control">
            <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/><path d="M19.07 4.93L17.66 6.34C18.78 7.46 19.5 9 19.5 10.71c0 1.71-.72 3.25-1.84 4.37l1.41 1.41C20.56 14.99 21.5 12.97 21.5 10.71c0-2.26-.94-4.28-2.43-5.78zM15.54 8.46l-1.41 1.41C14.64 10.38 15 11.01 15 11.71c0 .7-.36 1.33-.87 1.84l1.41 1.41c.92-.92 1.46-2.09 1.46-3.25 0-1.16-.54-2.33-1.46-3.25z"/></svg>
            <input type="range" min="0" max="100" value="75" class="volume-slider" />
        </div>
      </div>

      <div class="content-area">
        <!-- Listen Now -->
        <div v-if="selectedItem === 'listen_now'">
            <div class="hero-banner">
                <h1>Listen Now</h1>
                <h3>Top Picks for You</h3>
            </div>

            <div class="song-grid">
                <div 
                    v-for="(song, index) in songs" 
                    :key="index" 
                    class="song-card"
                    :class="{ active: currentSong && currentSong.title === song.title }"
                    @click="$emit('play-song', index)"
                >
                    <div class="song-cover">
                        <img :src="song.cover" :alt="song.title" />
                        <div class="play-overlay">
                            <svg viewBox="0 0 24 24" fill="white" width="30" height="30"><polygon points="5 3 19 12 5 21 5 3"/></svg>
                        </div>
                        <div class="playing-indicator" v-if="currentSong && currentSong.title === song.title && isPlaying">
                            <div class="bar"></div>
                            <div class="bar"></div>
                            <div class="bar"></div>
                        </div>
                    </div>
                    <div class="song-info">
                        <div class="card-title">{{ song.title }}</div>
                        <div class="card-artist">{{ song.artist }}</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Browse -->
        <div v-else-if="selectedItem === 'browse'">
            <div class="hero-banner">
                <h1>Browse</h1>
                <h3>New Music Daily</h3>
            </div>
            <div class="browse-grid">
                <div class="browse-card large">
                    <div class="browse-content" style="background: linear-gradient(45deg, #ff0055, #ff00aa);">
                        <span>New Music</span>
                        <h2>Daily Top 100</h2>
                    </div>
                </div>
                <div class="browse-card large">
                    <div class="browse-content" style="background: linear-gradient(45deg, #0055ff, #00aaff);">
                        <span>Featured</span>
                        <h2>Rap Life</h2>
                    </div>
                </div>
                <div class="browse-card">
                    <div class="browse-content" style="background: linear-gradient(45deg, #ffaa00, #ffff00);">
                        <h2>Hits</h2>
                    </div>
                </div>
                <div class="browse-card">
                    <div class="browse-content" style="background: linear-gradient(45deg, #00ff55, #00ffaa);">
                        <h2>Pop</h2>
                    </div>
                </div>
            </div>
        </div>

        <!-- Radio -->
        <div v-else-if="selectedItem === 'radio'">
            <div class="hero-banner">
                <h1>Radio</h1>
                <h3>Broadcasts</h3>
            </div>
            <div class="radio-list">
                <div class="radio-station">
                    <div class="station-logo" style="background: #fa2d48;">
                        <svg viewBox="0 0 24 24" fill="white" width="24" height="24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm-1-13h2v6h-2zm0 8h2v2h-2z"/></svg>
                    </div>
                    <div class="station-info">
                        <div class="station-name">samacOS Music 1</div>
                        <div class="station-desc">The new music that matters.</div>
                    </div>
                </div>
                <div class="radio-station">
                    <div class="station-logo" style="background: #a3aaae;">
                        <svg viewBox="0 0 24 24" fill="white" width="24" height="24"><circle cx="12" cy="12" r="10"/></svg>
                    </div>
                    <div class="station-info">
                        <div class="station-name">samacOS Music Hits</div>
                        <div class="station-desc">Songs you know and love.</div>
                    </div>
                </div>
                <div class="radio-station">
                    <div class="station-logo" style="background: #ff9500;">
                        <svg viewBox="0 0 24 24" fill="white" width="24" height="24"><path d="M12 2L2 22h20L12 2zm0 3.5L18.5 19H5.5L12 5.5z"/></svg>
                    </div>
                    <div class="station-info">
                        <div class="station-name">samacOS Music Country</div>
                        <div class="station-desc">Where country sounds like.</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Library: Songs (Reuse song grid) -->
        <div v-else-if="selectedItem === 'songs' || selectedItem === 'recently_added'">
            <div class="hero-banner">
                <h1>{{ selectedItem === 'songs' ? 'Songs' : 'Recently Added' }}</h1>
                <h3>{{ songs.length }} Songs</h3>
            </div>
            <div class="song-list-view">
                <div class="list-header">
                    <span>Title</span>
                    <span>Artist</span>
                    <span>Time</span>
                </div>
                <div 
                    v-for="(song, index) in songs" 
                    :key="index" 
                    class="list-row"
                    :class="{ active: currentSong && currentSong.title === song.title }"
                    @click="$emit('play-song', index)"
                >
                    <div class="list-col title">
                        <img :src="song.cover" class="list-cover" />
                        <span>{{ song.title }}</span>
                        <div class="playing-icon" v-if="currentSong && currentSong.title === song.title && isPlaying">
                            <svg viewBox="0 0 24 24" fill="#fa2d48" width="14" height="14"><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>
                        </div>
                    </div>
                    <div class="list-col artist">{{ song.artist }}</div>
                    <div class="list-col time">3:45</div>
                </div>
            </div>
        </div>

        <!-- Placeholder for others -->
        <div v-else class="empty-state">
            <div class="empty-icon">
                <svg viewBox="0 0 24 24" fill="none" stroke="#ddd" stroke-width="1" width="80" height="80">
                    <path d="M9 18V5l12-2v13" />
                    <circle cx="6" cy="18" r="3" />
                    <circle cx="18" cy="16" r="3" />
                </svg>
            </div>
            <h2>{{ selectedItem.replace('_', ' ').replace(/\b\w/g, l => l.toUpperCase()) }}</h2>
            <p>This section is empty.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.music-window {
  position: absolute;
  width: 900px;
  height: 600px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
  display: flex;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.music-window.maximized {
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  border-radius: 0;
}

.sidebar {
  width: 240px;
  background: #f9f9f9;
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  padding-top: 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
  padding: 10px 15px 15px;
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

.search-container {
    padding: 0 15px 20px;
}

.search-input-wrapper {
    background: #e5e5e5;
    border-radius: 6px;
    padding: 6px 10px;
    display: flex;
    align-items: center;
    gap: 8px;
    color: #888;
}

.search-input-wrapper input {
    background: transparent;
    border: none;
    outline: none;
    font-size: 13px;
    width: 100%;
}

.sidebar-section {
  margin-bottom: 25px;
}

.section-title {
  padding: 0 20px 5px;
  font-size: 12px;
  font-weight: 600;
  color: #888;
  text-transform: uppercase;
}

.sidebar-item {
  display: flex;
  align-items: center;
  padding: 8px 20px;
  font-size: 14px;
  color: #333;
  cursor: pointer;
  transition: background 0.2s;
}

.sidebar-item:hover {
  background: rgba(0, 0, 0, 0.05);
}

.sidebar-item.active {
  background: #e5e5e5;
  color: #fa2d48;
  font-weight: 500;
}

.sidebar-item.active .item-icon {
    color: #fa2d48;
}

.item-icon {
  margin-right: 12px;
  display: flex;
  align-items: center;
  color: #fa2d48;
}

.item-icon svg {
  width: 18px;
  height: 18px;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #fff;
}

.player-bar {
    height: 60px;
    border-bottom: 1px solid rgba(0,0,0,0.1);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    background: rgba(255,255,255,0.95);
    backdrop-filter: blur(20px);
}

.player-controls {
    display: flex;
    align-items: center;
    gap: 20px;
    flex: 1;
}

.player-btn {
    background: none;
    border: none;
    cursor: pointer;
    color: #333;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
}

.player-btn svg {
    width: 20px;
    height: 20px;
}

.player-btn.play-pause svg {
    width: 28px;
    height: 28px;
}

.now-playing-info {
    flex: 2;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border: 1px solid #e5e5e5;
    border-radius: 6px;
    padding: 4px 20px;
    background: #f9f9f9;
    height: 44px;
    position: relative;
}

.apple-logo-container {
    position: absolute;
    left: 10px;
    opacity: 0.3;
}

.song-display {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.song-title {
    font-size: 13px;
    font-weight: 600;
    color: #333;
}

.song-artist {
    font-size: 11px;
    color: #888;
}

.volume-control {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 10px;
    color: #888;
}

.volume-slider {
    width: 80px;
    height: 3px;
    -webkit-appearance: none;
    appearance: none;
    background: #ddd;
    border-radius: 2px;
    outline: none;
}

.volume-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background: #888;
    cursor: pointer;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

.content-area {
    flex: 1;
    padding: 30px 40px;
    overflow-y: auto;
}

.hero-banner {
    margin-bottom: 30px;
    border-bottom: 1px solid #eee;
    padding-bottom: 15px;
}

.hero-banner h1 {
    font-size: 34px;
    font-weight: 700;
    margin: 0 0 5px 0;
}

.hero-banner h3 {
    font-size: 18px;
    font-weight: 500;
    color: #fa2d48;
    margin: 0;
}

.song-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 25px;
}

.song-card {
    cursor: pointer;
    transition: transform 0.2s;
}

.song-card:hover {
    transform: scale(1.02);
}

.song-cover {
    width: 100%;
    aspect-ratio: 1;
    border-radius: 8px;
    overflow: hidden;
    margin-bottom: 10px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    position: relative;
}

.song-cover img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.play-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.2s;
}

.song-card:hover .play-overlay {
    opacity: 1;
}

.playing-indicator {
    position: absolute;
    bottom: 10px;
    right: 10px;
    display: flex;
    gap: 3px;
    align-items: flex-end;
    height: 15px;
}

.bar {
    width: 3px;
    background: #fa2d48;
    animation: bounce 1s infinite ease-in-out;
}

.bar:nth-child(1) { animation-delay: 0s; height: 60%; }
.bar:nth-child(2) { animation-delay: 0.2s; height: 100%; }
.bar:nth-child(3) { animation-delay: 0.4s; height: 80%; }

@keyframes bounce {
    0%, 100% { transform: scaleY(0.5); }
    50% { transform: scaleY(1); }
}

.song-info {
    padding: 0 2px;
}

.card-title {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    margin-bottom: 2px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.card-artist {
    font-size: 13px;
    color: #888;
}

.song-card.active .card-title {
    color: #fa2d48;
}

/* Browse Styles */
.browse-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 20px;
}

.browse-card {
    aspect-ratio: 16/9;
    border-radius: 8px;
    overflow: hidden;
    cursor: pointer;
    transition: transform 0.2s;
}

.browse-card.large {
    grid-column: span 2;
}

.browse-card:hover {
    transform: scale(1.02);
}

.browse-content {
    width: 100%;
    height: 100%;
    padding: 20px;
    color: white;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
}

.browse-content span {
    font-size: 12px;
    text-transform: uppercase;
    font-weight: 600;
    opacity: 0.9;
}

.browse-content h2 {
    margin: 5px 0 0;
    font-size: 24px;
}

/* Radio Styles */
.radio-list {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.radio-station {
    display: flex;
    align-items: center;
    padding: 10px;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.2s;
}

.radio-station:hover {
    background: #f5f5f5;
}

.station-logo {
    width: 50px;
    height: 50px;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 15px;
}

.station-info {
    flex: 1;
}

.station-name {
    font-weight: 600;
    color: #333;
    margin-bottom: 2px;
}

.station-desc {
    font-size: 13px;
    color: #888;
}

/* List View Styles */
.song-list-view {
    display: flex;
    flex-direction: column;
}

.list-header {
    display: flex;
    padding: 0 10px 10px;
    border-bottom: 1px solid #eee;
    color: #888;
    font-size: 12px;
    text-transform: uppercase;
}

.list-row {
    display: flex;
    align-items: center;
    padding: 8px 10px;
    border-radius: 6px;
    cursor: pointer;
}

.list-row:hover {
    background: #f5f5f5;
}

.list-row.active {
    background: #f0f0f0;
}

.list-row.active .list-col.title span {
    color: #fa2d48;
}

.list-col {
    flex: 1;
    font-size: 14px;
    color: #333;
}

.list-col.title {
    flex: 2;
    display: flex;
    align-items: center;
    gap: 10px;
    font-weight: 500;
}

.list-cover {
    width: 30px;
    height: 30px;
    border-radius: 4px;
}

.list-col.artist, .list-col.time {
    color: #888;
}

.list-col.time {
    text-align: right;
    flex: 0.5;
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

.empty-state h2 {
    color: #333;
    margin: 20px 0 5px;
}

.empty-state p {
    color: #888;
}
</style>

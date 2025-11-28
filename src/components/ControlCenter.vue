<script setup>
import { ref, computed, onUnmounted, watch } from 'vue'
import bluetoothOnIcon from '../assets/Icons/bluetooth-on.svg'
import bluetoothOffIcon from '../assets/Icons/bluetooth-off.svg'
import airdropIcon from '../assets/Icons/airdrop.svg'

const props = defineProps({
  isOpen: Boolean,
  currentSong: Object,
  isPlaying: Boolean
})

const emit = defineEmits(['close', 'toggle-play', 'next-song', 'prev-song'])

// State for toggles
const wifiOn = ref(true)
const bluetoothOn = ref(true)
const airdropOn = ref(true)
const brightness = ref(100)
const volume = ref(75)
const isDarkMode = ref(false)
const isStageManager = ref(false)

// Watchers for sliders
watch(volume, (newVal) => {
  // Volume control would need to be lifted up too if we want it synced perfectly, 
  // but for now let's just emit or handle it locally if we had access to the audio object.
  // Since audio object is in parent, we can't set volume directly here easily without another prop/emit.
  // For this specific request (persistence), we can skip volume sync or add it later.
})

watch(brightness, (newVal) => {
  document.getElementById('app').style.filter = `brightness(${newVal}%)`
  document.getElementById('app').style.transition = 'filter 0.1s ease'
}, { immediate: true })

const togglePlay = () => {
  emit('toggle-play')
}

const nextSong = () => {
  emit('next-song')
}

const prevSong = () => {
  emit('prev-song')
}


</script>

<template>
  <div class="control-center-overlay" v-if="isOpen" @click="emit('close')"></div>
  <Transition name="fade-scale">
    <div class="control-center" v-if="isOpen">
      
      <!-- Top Section -->
      <div class="row top-row">
        <!-- Connectivity -->
        <div class="block connectivity">
          <!-- Wi-Fi -->
          <div class="conn-row" :class="{ active: wifiOn }" @click="wifiOn = !wifiOn">
            <div class="conn-icon-circle wifi">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12.55a11 11 0 0 1 14.08 0"></path><path d="M1.42 9a16 16 0 0 1 21.16 0"></path><path d="M8.53 16.11a6 6 0 0 1 6.95 0"></path><line x1="12" y1="20" x2="12.01" y2="20"></line></svg>
            </div>
            <div class="conn-text">
              <div class="conn-title">Wi-Fi</div>
              <div class="conn-status">{{ wifiOn ? 'samacOS Network' : 'Off' }}</div>
            </div>
          </div>

          <!-- Bluetooth -->
          <div class="conn-row" :class="{ active: bluetoothOn }" @click="bluetoothOn = !bluetoothOn">
            <div class="conn-icon-circle bluetooth">
               <img :src="bluetoothOn ? bluetoothOnIcon : bluetoothOffIcon" alt="Bluetooth" class="icon-img" />
            </div>
            <div class="conn-text">
              <div class="conn-title">Bluetooth</div>
              <div class="conn-status">{{ bluetoothOn ? 'On' : 'Off' }}</div>
            </div>
          </div>

          <!-- AirDrop -->
          <div class="conn-row" :class="{ active: airdropOn }" @click="airdropOn = !airdropOn">
            <div class="conn-icon-circle airdrop">
               <img :src="airdropIcon" alt="AirDrop" class="icon-img" />
            </div>
            <div class="conn-text">
              <div class="conn-title">AirDrop</div>
              <div class="conn-status">{{ airdropOn ? 'Contacts Only' : 'Off' }}</div>
            </div>
          </div>
        </div>

        <!-- Right Column -->
        <div class="right-column">
          <!-- Media -->
          <div class="block media">
            <div class="media-content">
                <div class="media-icon-placeholder">
                    <img v-if="currentSong.cover" :src="currentSong.cover" class="album-art" />
                    <svg v-else width="24" height="24" viewBox="0 0 24 24" fill="currentColor" color="#ccc"><path d="M9 18V5l12 7-12 6z"/></svg>
                </div>
                <div class="media-info">
                    <div class="media-title">{{ currentSong.title }}</div>
                    <div class="media-artist">{{ currentSong.artist }}</div>
                </div>
                <div class="media-controls">
                    <svg @click="prevSong" class="control-icon" width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M11 18V6l-8.5 6 8.5 6zm.5-6l8.5 6V6l-8.5 6z"/></svg>
                    
                    <svg v-if="!isPlaying" @click="togglePlay" class="control-icon play-pause" width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M8 5v14l11-7z"/></svg>
                    <svg v-else @click="togglePlay" class="control-icon play-pause" width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><rect x="6" y="4" width="4" height="16"></rect><rect x="14" y="4" width="4" height="16"></rect></svg>
                    
                    <svg @click="nextSong" class="control-icon" width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M4 18l8.5-6L4 6v12zm9-12v12l8.5-6L13 6z"/></svg>
                </div>
            </div>
          </div>
          
          <!-- Small Buttons -->
          
        </div>
      </div>

      <!-- Middle Section -->
      <div class="row middle-row">
         <div class="block small-btn display-toggle">
            <div class="circle-icon white-bg">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="black" stroke-width="2"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="3"/></svg>
            </div>
         </div>
         <div class="small-buttons-row">
             <div class="block small-btn" :class="{ active: isDarkMode }" @click="isDarkMode = !isDarkMode">
                <div class="circle-icon">
                    <svg v-if="!isDarkMode" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
                    <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
                </div>
             </div>
             <div class="block small-btn" :class="{ active: isStageManager }" @click="isStageManager = !isStageManager">
                <div class="circle-icon">
                    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2" ry="2"/><line x="8" y="21" x2="16" y2="21"/><line x="12" y1="17" x2="12" y2="21"/></svg>
                </div>
             </div>
          </div>
      </div>

      <!-- Sliders Section -->
      <div class="sliders-section">
        <div class="block slider-block">
            <div class="slider-label">Display</div>
            <div class="slider-wrapper">
                <div class="slider-icon-container">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#666" stroke-width="2"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
                </div>
                <input type="range" v-model="brightness" min="10" max="100" class="slider display-slider" :style="{ '--val': brightness + '%' }">
            </div>
        </div>

        <div class="block slider-block">
            <div class="slider-label">Sound</div>
            <div class="slider-wrapper">
                <div class="slider-icon-container">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#666" stroke-width="2"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"></polygon><path d="M19.07 4.93a10 10 0 0 1 0 14.14M15.54 8.46a5 5 0 0 1 0 7.07"></path></svg>
                </div>
                <input type="range" v-model="volume" min="0" max="100" class="slider sound-slider" :style="{ '--val': volume + '%' }">
            </div>
        </div>
      </div>

    </div>
  </Transition>
</template>

<style scoped>
.control-center-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 9998;
}

.control-center {
  position: absolute;
  top: 34px;
  right: 10px;
  width: 340px;
  background: rgba(30, 30, 30, 0.65);
  backdrop-filter: blur(25px);
  -webkit-backdrop-filter: blur(25px);
  border-radius: 18px;
  padding: 14px;
  box-shadow: 0 0 0 1px rgba(255,255,255,0.1), 0 20px 40px rgba(0,0,0,0.4);
  z-index: 9999;
  color: white;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.row {
  display: flex;
  gap: 14px;
}

.block {
  background: rgba(40, 40, 40, 0.5);
  border-radius: 16px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  transition: background 0.2s;
}

.block:hover {
  background: rgba(60, 60, 60, 0.6);
}

/* Top Section */
.top-row {
  height: 150px;
}

.connectivity {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 14px;
}

.conn-row {
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
}

.conn-icon-circle {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: rgba(80, 80, 80, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  transition: all 0.3s;
}

.icon-img {
  width: 18px;
  height: 18px;
  filter: invert(1); /* Make black icons white */
}

.conn-row.active .conn-icon-circle {
  background: #007AFF;
}

.conn-text {
  display: flex;
  flex-direction: column;
}

.conn-title {
  font-size: 13px;
  font-weight: 600;
}

.conn-status {
  font-size: 11px;
  color: #aaa;
}

/* Right Column */
.right-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.media {
  flex: 2;
  padding: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.media-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
}

.media-icon-placeholder {
    margin-bottom: 5px;
}

.media-info {
    text-align: center;
    margin-bottom: 8px;
}

.media-title {
    font-size: 13px;
    font-weight: 600;
}

.media-artist {
    font-size: 11px;
    color: #aaa;
}

.media-controls {
    display: flex;
    gap: 16px;
    color: #ddd;
}

.small-buttons-row {
  flex: 1;
  display: flex;
  gap: 14px;
}

.small-btn {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.circle-icon {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: rgba(80,80,80,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.small-btn.active .circle-icon {
  background: white;
  color: #007AFF;
}

/* Middle Section */
.middle-row {
    height: 60px;
}

.middle-row .small-btn {
    flex: 1;
}

.white-bg {
    background: white !important;
    color: black !important;
}

.blue-bg {
    background: #007AFF !important;
    color: white !important;
}

.focus-pill {
    flex: 1;
    display: flex;
    align-items: center;
    padding: 0 16px;
    gap: 12px;
    cursor: pointer;
}

.focus-icon {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    background: rgba(80,80,80,0.5);
    display: flex;
    align-items: center;
    justify-content: center;
}

.focus-text {
    font-size: 13px;
    font-weight: 600;
}

/* Sliders */
.sliders-section {
    display: flex;
    flex-direction: column;
    gap: 14px;
}

.slider-block {
    padding: 12px;
    background: rgba(40, 40, 40, 0.5);
}

.slider-label {
    font-size: 12px;
    font-weight: 600;
    margin-bottom: 8px;
    padding-left: 4px;
    color: #ddd;
}

.slider-wrapper {
    position: relative;
    height: 26px;
    background: rgba(0,0,0,0.2);
    border-radius: 13px;
    overflow: hidden;
}

.slider-icon-container {
    position: absolute;
    left: 8px;
    top: 50%;
    transform: translateY(-50%);
    z-index: 2;
    pointer-events: none;
}

.slider {
    -webkit-appearance: none;
    appearance: none;
    width: 100%;
    height: 100%;
    background: transparent;
    outline: none;
    cursor: pointer;
    position: relative;
    z-index: 1;
}

.slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 0;
    height: 0;
    box-shadow: -100vw 0 0 100vw white;
}

.album-art {
  width: 40px;
  height: 40px;
  border-radius: 6px;
  object-fit: cover;
  margin-bottom: 5px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.3);
}

.control-icon {
  cursor: pointer;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.control-icon:hover {
  opacity: 1;
}

.play-pause {
  opacity: 1;
}

/* Transitions */
.fade-scale-enter-active,
.fade-scale-leave-active {
  transition: all 0.2s ease;
}

.fade-scale-enter-from,
.fade-scale-leave-to {
  opacity: 0;
  transform: scale(0.95) translate(10px, -10px);
}
</style>

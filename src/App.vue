<script setup>
import { ref } from 'vue'
import MacBookIntro from './components/MacBookIntro.vue'
import BootLoader from './components/BootLoader.vue'
import LoginScreen from './components/LoginScreen.vue'
import Desktop from './components/Desktop.vue'

const showIntro = ref(true)
const showBoot = ref(false)
const showLogin = ref(false)
const showDesktop = ref(false)

const handleBoot = () => {
  showIntro.value = false
  showBoot.value = true
  
  // Simulate boot time
  setTimeout(() => {
    showBoot.value = false
    showLogin.value = true
  }, 4000)
}

const handleLogin = () => {
  showLogin.value = false
  showDesktop.value = true
}
</script>

<template>
  <MacBookIntro v-if="showIntro" @boot="handleBoot" />
  
  <Transition name="fade">
    <BootLoader v-if="showBoot" />
  </Transition>
  
  <Transition name="fade">
    <LoginScreen v-if="showLogin" @login="handleLogin" />
  </Transition>
  
  <Transition name="fade">
    <Desktop v-if="showDesktop" />
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

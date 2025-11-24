<script setup>
import { ref, onMounted } from 'vue'
import BootLoader from './components/BootLoader.vue'
import LoginScreen from './components/LoginScreen.vue'

const isLoading = ref(true)
const isLoggedIn = ref(false)

onMounted(() => {
  // Simulate boot time matching the CSS animation
  setTimeout(() => {
    isLoading.value = false
  }, 3500)
})

const handleLogin = () => {
  isLoggedIn.value = true
}
</script>

<template>
  <Transition name="fade">
    <BootLoader v-if="isLoading" />
  </Transition>
  
  <LoginScreen v-if="!isLoggedIn" @login="handleLogin" />
  <div v-else class="desktop">
    <h1>Welcome to my Portfolio</h1>
  </div>
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

<style scoped>
.desktop {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f0f0f0;
  color: #333;
}
</style>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
import gsap from 'gsap'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js'

const emit = defineEmits(['boot'])
const container = ref(null)

// three.js globals
let scene, camera, renderer
let laptopGroup, lidGroup, screenMesh
let animationId
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()
let isZooming = false

// extras
let glow = null          // power button glow mesh
let textMesh = null      // 3D text mesh
let glowTween = null

onMounted(() => {
  init()
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('click', handleClick)
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
})

// --------------------------------------------------------
// INIT
// --------------------------------------------------------

function init() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color('#0d0d0d')

  // Camera
  camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    200
  )
  camera.position.set(0, 4, 10)

  // Renderer
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1.2
  container.value.appendChild(renderer.domElement)

  // --------------------------------------------------------
  // LIGHTS
  // --------------------------------------------------------

  const ambient = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambient)

  const spot = new THREE.SpotLight(0xffffff, 5)
  spot.position.set(5, 12, 10)
  spot.target.position.set(0, 1.5, 0)
  spot.castShadow = true
  scene.add(spot)
  scene.add(spot.target)

  // Rim light za Apple vibe
  const rim = new THREE.DirectionalLight(0xffffff, 1.5)
  rim.position.set(-4, 6, -6)
  scene.add(rim)

  // --------------------------------------------------------
  // DESK & WALL
  // --------------------------------------------------------

  const deskGeometry = new THREE.BoxGeometry(30, 0.5, 20)
  const deskMaterial = new THREE.MeshStandardMaterial({
    color: 0x1a1a1a,
    roughness: 0.8,
    metalness: 0.1
  })
  const desk = new THREE.Mesh(deskGeometry, deskMaterial)
  desk.position.y = -0.25
  desk.receiveShadow = true
  scene.add(desk)

  const wallGeometry = new THREE.PlaneGeometry(40, 20)
  const wallMaterial = new THREE.MeshStandardMaterial({
    color: 0x111111,
    roughness: 0.8,
    metalness: 0.1
  })
  const wall = new THREE.Mesh(wallGeometry, wallMaterial)
  wall.position.set(0, 5, -6)
  wall.receiveShadow = true
  scene.add(wall)

  
  // --------------------------------------------------------
  // LAPTOP GROUP
  // --------------------------------------------------------

  laptopGroup = new THREE.Group()
  // Sit on desk (desk top is at 0)
  laptopGroup.position.set(0, 0.06, 0)
  scene.add(laptopGroup)

  // Camera LookAt
  camera.lookAt(0, 0.5, 0)

  const aluminum = new THREE.MeshStandardMaterial({
    color: 0xbbbbbb,
    roughness: 0.3,
    metalness: 0.9
  })

  const blackPlastic = new THREE.MeshStandardMaterial({
    color: 0x111111,
    roughness: 0.6
  })

  // Base
  const baseGeo = new THREE.BoxGeometry(3.5, 0.12, 2.4)
  const base = new THREE.Mesh(baseGeo, aluminum)
  base.castShadow = true
  base.receiveShadow = true
  laptopGroup.add(base)

  // Keyboard (grid tastera)
  const keyboard = new THREE.Group()
  keyboard.position.set(0, 0.065, 0.25)

  for (let x = -1.2; x <= 1.2; x += 0.2) {
    for (let z = -0.6; z <= 0.6; z += 0.2) {
      const keyGeo = new THREE.BoxGeometry(0.16, 0.02, 0.16)
      const key = new THREE.Mesh(keyGeo, blackPlastic)
      key.position.set(x, 0, z)
      key.castShadow = true
      keyboard.add(key)
    }
  }
  laptopGroup.add(keyboard)

  // Lid group (ekran)
  lidGroup = new THREE.Group()
  lidGroup.position.set(0, 0.06, -1.2)
  laptopGroup.add(lidGroup)

  const lidGeo = new THREE.BoxGeometry(3.5, 0.08, 2.4)
  lidGeo.translate(0, 0.04, 1.2) // pivot u šarki
  const lid = new THREE.Mesh(lidGeo, aluminum)
  lid.castShadow = true
  lidGroup.add(lid)

  // Bezel
  const bezelGeo = new THREE.PlaneGeometry(3.3, 2.1)
  bezelGeo.translate(0, 0.09, 1.2)
  bezelGeo.rotateX(-Math.PI / 2)
  const bezel = new THREE.Mesh(bezelGeo, blackPlastic)
  lidGroup.add(bezel)

  // Screen
  const screenGeo = new THREE.PlaneGeometry(3.1, 1.9)
  screenGeo.translate(0, 0.091, 1.2)
  screenGeo.rotateX(-Math.PI / 2)
  const screenMat = new THREE.MeshBasicMaterial({ color: 0x000000 })
  screenMesh = new THREE.Mesh(screenGeo, screenMat)
  lidGroup.add(screenMesh)

  // Otvoren ugao
  lidGroup.rotation.x = -1.5

  // Power button hit area (nevidljivo)
  const hitGeo = new THREE.CircleGeometry(0.25, 32)
  hitGeo.rotateX(-Math.PI / 2)
  const hitMat = new THREE.MeshBasicMaterial({ visible: false })
  const power = new THREE.Mesh(hitGeo, hitMat)
  power.name = 'powerButton'
  power.position.set(1.5, 0.07, -0.8)
  laptopGroup.add(power)

  // Glow disk ispod dugmeta
  const glowGeo = new THREE.CircleGeometry(0.1, 32)
  glowGeo.rotateX(-Math.PI / 2)
  const glowMat = new THREE.MeshBasicMaterial({
    transparent: true,
    opacity: 0.4
  })
  glow = new THREE.Mesh(glowGeo, glowMat)
  glow.position.set(1.5, 0.061, -0.8)
  laptopGroup.add(glow)

  // Pulsiranje glowa
  glowTween = gsap.to(glow.material, {
    opacity: 0.8,
    duration: 1.2,
    yoyo: true,
    repeat: -1,
    ease: 'sine.inOut'
  })

  // --------------------------------------------------------
  // EVENT LISTENERS
  // --------------------------------------------------------

  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('click', handleClick)
  window.addEventListener('resize', handleResize)

  animate()
}

// --------------------------------------------------------
// RAYCAST: MOUSEMOVE
// --------------------------------------------------------

function handleMouseMove(e) {
  if (isZooming) return

  mouse.x = (e.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1

  raycaster.setFromCamera(mouse, camera)
  const hits = raycaster.intersectObjects(scene.children, true)
  const btnHover = hits.some(h => h.object.name === 'powerButton')

  if (glow) {
    if (btnHover) {
      glowTween.pause()
      glow.material.opacity = 1
    } else {
      glowTween.play()
    }
  }

  // Parallax tilt
  gsap.to(laptopGroup.rotation, {
    x: mouse.y * 0.1,
    y: mouse.x * 0.1,
    duration: 1.2
  })
}

// --------------------------------------------------------
// CLICK
// --------------------------------------------------------

function handleClick(e) {
  if (isZooming) return

  mouse.x = (e.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1

  raycaster.setFromCamera(mouse, camera)
  const hits = raycaster.intersectObjects(scene.children, true)

  const btn = hits.find(h => h.object.name === 'powerButton')
  if (btn) {
    boot()
  }
}

// --------------------------------------------------------
// BOOT SEQUENCE
// --------------------------------------------------------

function boot() {
  isZooming = true
  screenMesh.material.color.setHex(0x333333)

  const tl = gsap.timeline({
    onComplete: () => emit('boot')
  })

  tl.to(camera.position, {
    x: 0,
    y: 2,
    z: -0.3,
    duration: 2,
    ease: 'power2.inOut'
  })

  tl.to('.overlay', { opacity: 1, duration: 1 }, '-=1')
}

// --------------------------------------------------------

function handleResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function animate() {
  animationId = requestAnimationFrame(animate)
  renderer.render(scene, camera)
}
</script>

<template>
  <div ref="container" class="three-container">
    <div class="instruction">Click the Power Button to Start</div>
    <div class="overlay"></div>
  </div>
</template>

<style scoped>
.three-container {
  width: 100vw;
  height: 100vh;
  background: black;
  position: relative;
  overflow: hidden;
}

.overlay {
  position: absolute;
  inset: 0;
  background: black;
  opacity: 0;
  z-index: 10;
  pointer-events: none;
  transition: opacity 0.5s;
}

.instruction {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  color: rgba(255, 255, 255, 0.5);
  font-size: 14px;
  user-select: none;
  pointer-events: none;
  z-index: 5;
  font-family: -apple-system, BlinkMacSystemFont, system-ui, sans-serif;
}
</style>

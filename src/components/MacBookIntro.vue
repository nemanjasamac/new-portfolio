<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { PointerLockControls } from 'three/examples/jsm/controls/PointerLockControls.js'
import gsap from 'gsap'
import macbookUrl from '../assets/models/macbook.glb'
import officeUrl from '../assets/models/office.glb'
const emit = defineEmits(['boot'])
const container = ref(null)

// three.js globals
let scene, camera, renderer
let laptopGroup, screenMesh
let controls
let animationId
const raycaster = new THREE.Raycaster()
const mouse = new THREE.Vector2()
let isZooming = false
let isBooting = false
let hoverTimer = 0
const HOVER_THRESHOLD = 1.0 // seconds to trigger boot

// extras
let glow = null          // power button glow mesh
let textMesh = null      // 3D text mesh
let glowTween = null

// Movement state
const moveState = {
  forward: false,
  backward: false,
  left: false,
  right: false
}
const velocity = new THREE.Vector3()
const direction = new THREE.Vector3()
let prevTime = performance.now()

onMounted(() => {
  init()
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
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
    1000
  )
  camera.position.set(-2, 1.6, 4) // Lower and closer

  // Controls
  controls = new PointerLockControls(camera, document.body)
  controls.pointerSpeed = 0.3 // Slower sensitivity
  
  const overlay = document.querySelector('.intro-overlay')
  const subText = document.querySelector('.sub-text')
  
  // Click on the overlay to lock controls
  overlay.addEventListener('click', () => {
    controls.lock()
  })
  
  controls.addEventListener('lock', () => {
    overlay.style.display = 'none'
  })
  
  controls.addEventListener('unlock', () => {
    if (isBooting) return
    overlay.style.display = 'flex'
    if (subText) subText.textContent = '(Click to Resume)'
  })

  scene.add(camera)

  // Key listeners
  const onKeyDown = (event) => {
    switch (event.code) {
      case 'ArrowUp':
      case 'KeyW':
        moveState.forward = true
        break
      case 'ArrowLeft':
      case 'KeyA':
        moveState.left = true
        break
      case 'ArrowDown':
      case 'KeyS':
        moveState.backward = true
        break
      case 'ArrowRight':
      case 'KeyD':
        moveState.right = true
        break
    }
  }

  const onKeyUp = (event) => {
    switch (event.code) {
      case 'ArrowUp':
      case 'KeyW':
        moveState.forward = false
        break
      case 'ArrowLeft':
      case 'KeyA':
        moveState.left = false
        break
      case 'ArrowDown':
      case 'KeyS':
        moveState.backward = false
        break
      case 'ArrowRight':
      case 'KeyD':
        moveState.right = false
        break
    }
  }

  document.addEventListener('keydown', onKeyDown)
  document.addEventListener('keyup', onKeyUp)

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

  const loader = new GLTFLoader()

  // Load Office
  loader.load(officeUrl, (gltf) => {
    const office = gltf.scene
    office.scale.set(10, 10, 10) 
    office.position.set(0, -1, 0) 
    
    office.traverse((child) => {
      if (child.isMesh) {
        child.receiveShadow = true
        child.castShadow = true
      }
    })
    scene.add(office)
  })

  // --------------------------------------------------------
  // LAPTOP GROUP
  // --------------------------------------------------------

  laptopGroup = new THREE.Group()
  // Sit on desk (adjusted position)
  laptopGroup.position.set(0, -8, -1)
  laptopGroup.rotation.y = -Math.PI / 2
  scene.add(laptopGroup)

  // Camera LookAt
  camera.lookAt(0, 0.5, 0)

  // Load GLB Model
  loader.load(macbookUrl, (gltf) => {
    const model = gltf.scene
    // Adjust scale and rotation if needed
    model.scale.set(15, 15, 15) 
    model.position.set(0, -1, 0)
    model.rotation.y = 0 // Rotate to face camera if needed

    model.traverse((child) => {
      if (child.isMesh) {
        child.castShadow = true
        child.receiveShadow = true
        
        // Try to identify screen mesh by name or material name
        const name = child.name.toLowerCase()
        const matName = child.material ? child.material.name.toLowerCase() : ''
        
        if (name.includes('screen') || name.includes('display') || 
            matName.includes('screen') || matName.includes('display')) {
          screenMesh = child
          // Clone material to avoid affecting other meshes sharing same material
          screenMesh.material = screenMesh.material.clone()
          screenMesh.material.color.setHex(0x000000)
          screenMesh.material.emissive.setHex(0x000000)
        }
      }
    })

    laptopGroup.add(model)
  })

  // Power button hit area (nevidljivo)
  const hitGeo = new THREE.CircleGeometry(0.2, 32)
  hitGeo.rotateX(-Math.PI / 2)
  const hitMat = new THREE.MeshBasicMaterial({ visible: false })
  const power = new THREE.Mesh(hitGeo, hitMat)
  power.name = 'powerButton'
  // Match the glow position
  power.position.set(1.95, -0.8, -1.2)
  laptopGroup.add(power)

  // Glow disk ispod dugmeta
  const glowGeo = new THREE.CircleGeometry(0.1, 32)
  glowGeo.rotateX(-Math.PI / 2)
  const glowMat = new THREE.MeshBasicMaterial({
    transparent: true,
    opacity: 0.4,
    color: 0xffffff
  })
  glow = new THREE.Mesh(glowGeo, glowMat)
  glow.position.set(1.95, -0.8, -1.2)
  laptopGroup.add(glow)

  // Arrow and Text
  createArrowAndText()

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

  // window.addEventListener('mousemove', handleMouseMove) // Removed for PointerLock
  window.addEventListener('click', handleClick)
  window.addEventListener('resize', handleResize)

  animate()
}

// --------------------------------------------------------
// CREATE ARROW AND TEXT
// --------------------------------------------------------

function createArrowAndText() {
  const arrowGroup = new THREE.Group()
  arrowGroup.position.set(1.95, 0.8, -1.2) // Above the button

  // Arrow Body (Cylinder)
  const cylinderGeo = new THREE.CylinderGeometry(0.02, 0.02, 0.3, 32)
  const arrowMat = new THREE.MeshBasicMaterial({ color: 0xffffff })
  const cylinder = new THREE.Mesh(cylinderGeo, arrowMat)
  cylinder.position.y = 0.15
  arrowGroup.add(cylinder)

  // Arrow Head (Cone)
  const coneGeo = new THREE.ConeGeometry(0.08, 0.2, 32)
  const cone = new THREE.Mesh(coneGeo, arrowMat)
  cone.position.y = 0 // Tip at 0? No, center at 0.
  // Cone height is 0.2. Center is at 0. Tip is at 0.1, base at -0.1.
  // We want tip pointing down.
  cone.rotation.x = Math.PI
  cone.position.y = 0
  arrowGroup.add(cone)

  // Text Label
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  canvas.width = 512
  canvas.height = 128
  
  // ctx.fillStyle = 'rgba(0,0,0,0.5)'; // Debug background
  // ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  ctx.font = 'bold 60px -apple-system, BlinkMacSystemFont, sans-serif'
  ctx.fillStyle = 'white'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'bottom'
  ctx.shadowColor = 'rgba(0,0,0,0.8)'
  ctx.shadowBlur = 10
  ctx.fillText('Turn on samacOS', canvas.width / 2, canvas.height - 10)
  
  const texture = new THREE.CanvasTexture(canvas)
  const material = new THREE.SpriteMaterial({ map: texture })
  const sprite = new THREE.Sprite(material)
  sprite.position.y = 0.5
  sprite.scale.set(2, 0.5, 1) // Aspect ratio 4:1
  arrowGroup.add(sprite)

  laptopGroup.add(arrowGroup)

  // Float animation
  gsap.to(arrowGroup.position, {
    y: 1.0,
    duration: 1.5,
    yoyo: true,
    repeat: -1,
    ease: 'sine.inOut'
  })
}

// --------------------------------------------------------
// RAYCAST: MOUSEMOVE (Adapted for Center Screen)
// --------------------------------------------------------

function checkIntersection(delta) {
  if (isZooming) return

  // Raycast from center of screen
  raycaster.setFromCamera(new THREE.Vector2(0, 0), camera)
  const hits = raycaster.intersectObjects(scene.children, true)
  const btnHover = hits.some(h => h.object.name === 'powerButton')

  if (glow) {
    if (btnHover) {
      glowTween.pause()
      glow.material.opacity = 1
      
      // Auto-trigger boot if hovering
      hoverTimer += delta
      if (hoverTimer > HOVER_THRESHOLD) {
        boot()
      }
    } else {
      glowTween.play()
      hoverTimer = 0
    }
  }
}

// --------------------------------------------------------
// CLICK
// --------------------------------------------------------

function handleClick(e) {
  if (isZooming) return
  if (!controls.isLocked) return // Only click if locked

  // Raycast from center
  raycaster.setFromCamera(new THREE.Vector2(0, 0), camera)
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
  isBooting = true
  isZooming = true
  if (screenMesh) {
    screenMesh.material.color.setHex(0x333333)
  }

  const tl = gsap.timeline({
    onComplete: () => {
      controls.unlock()
      emit('boot')
    }
  })

  tl.to(camera.position, {
    x: laptopGroup.position.x,
    y: laptopGroup.position.y + 0.5,
    z: laptopGroup.position.z + 0.7,
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

  const time = performance.now()
  const delta = (time - prevTime) / 1000

  if (controls.isLocked) {
    velocity.x -= velocity.x * 10.0 * delta
    velocity.z -= velocity.z * 10.0 * delta

    direction.z = Number(moveState.forward) - Number(moveState.backward)
    direction.x = Number(moveState.right) - Number(moveState.left)
    direction.normalize() // this ensures consistent movements in all directions

    if (moveState.forward || moveState.backward) velocity.z -= direction.z * 400.0 * delta
    if (moveState.left || moveState.right) velocity.x -= direction.x * 400.0 * delta

    controls.moveRight(-velocity.x * delta)
    controls.moveForward(-velocity.z * delta)

    // Simple bounds checking (Room limits)
    // Assuming room is roughly centered at 0,0 and about 10x10 units based on office scale
    const minX = -12, maxX = 12
    const minZ = -12, maxZ = 12
    
    camera.position.x = Math.max(minX, Math.min(maxX, camera.position.x))
    camera.position.z = Math.max(minZ, Math.min(maxZ, camera.position.z))
  }

  prevTime = time

  checkIntersection(delta)
  renderer.render(scene, camera)
}
</script>

<template>
  <div ref="container" class="three-container">
    <div class="crosshair"></div>
    
    <div class="intro-overlay">
      <div class="intro-content">
        <h1>Welcome</h1>
        <p>Move around to turn on samacOS</p>
        <p class="sub-text">(Click to Start)</p>
      </div>
    </div>

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

.crosshair {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 10px;
  height: 10px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  z-index: 20;
  border: 1px solid rgba(0, 0, 0, 0.5);
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

.intro-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 30;
  cursor: pointer;
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
}

.intro-content {
  text-align: center;
  color: white;
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  animation: fadeIn 1s ease-out;
}

.intro-content h1 {
  font-size: 48px;
  font-weight: 200;
  margin-bottom: 16px;
  letter-spacing: 2px;
}

.intro-content p {
  font-size: 18px;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 8px;
  font-weight: 300;
}

.intro-content .sub-text {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 20px;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>

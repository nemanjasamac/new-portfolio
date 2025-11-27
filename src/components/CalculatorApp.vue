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

const windowRef = ref(null)
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: 300, y: 150 })

const displayValue = ref('0')
const previousValue = ref(null)
const operator = ref(null)
const newNumber = ref(true)

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.calc-btn')) return
  
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

const appendNumber = (num) => {
  if (newNumber.value) {
    displayValue.value = String(num)
    newNumber.value = false
  } else {
    displayValue.value = displayValue.value === '0' ? String(num) : displayValue.value + num
  }
}

const appendDot = () => {
  if (newNumber.value) {
    displayValue.value = '0.'
    newNumber.value = false
  } else if (displayValue.value.indexOf('.') === -1) {
    displayValue.value += '.'
  }
}

const clear = () => {
  displayValue.value = '0'
  previousValue.value = null
  operator.value = null
  newNumber.value = true
}

const toggleSign = () => {
  displayValue.value = String(parseFloat(displayValue.value) * -1)
}

const percentage = () => {
  displayValue.value = String(parseFloat(displayValue.value) / 100)
}

const setOperator = (op) => {
  if (operator.value && !newNumber.value) {
    calculate()
  }
  previousValue.value = parseFloat(displayValue.value)
  operator.value = op
  newNumber.value = true
}

const calculate = () => {
  if (operator.value === null || newNumber.value && previousValue.value === null) return

  const current = parseFloat(displayValue.value)
  const prev = previousValue.value
  let result = 0

  switch (operator.value) {
    case '+':
      result = prev + current
      break
    case '-':
      result = prev - current
      break
    case '*':
      result = prev * current
      break
    case '/':
      result = prev / current
      break
  }

  displayValue.value = String(result)
  previousValue.value = null
  operator.value = null
  newNumber.value = true
}

const fontSize = computed(() => {
  const len = displayValue.value.length
  if (len > 9) return '2.5rem'
  if (len > 6) return '3.5rem'
  return '4.5rem'
})
</script>

<template>
  <div 
    v-if="isOpen"
    ref="windowRef"
    class="calculator-window"
    :style="{ 
      left: `${position.x}px`, 
      top: `${position.y}px`,
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <div class="title-bar">
      <div class="window-controls">
        <div class="control-btn close" @click.stop="emit('close')"></div>
        <div class="control-btn minimize"></div>
        <div class="control-btn maximize"></div>
      </div>
    </div>

    <div class="display" :style="{ fontSize: fontSize }">
      {{ displayValue }}
    </div>

    <div class="buttons-grid">
      <button class="calc-btn gray" @click="clear">{{ displayValue === '0' ? 'AC' : 'C' }}</button>
      <button class="calc-btn gray" @click="toggleSign">+/-</button>
      <button class="calc-btn gray" @click="percentage">%</button>
      <button class="calc-btn orange" @click="setOperator('/')">÷</button>

      <button class="calc-btn dark" @click="appendNumber(7)">7</button>
      <button class="calc-btn dark" @click="appendNumber(8)">8</button>
      <button class="calc-btn dark" @click="appendNumber(9)">9</button>
      <button class="calc-btn orange" @click="setOperator('*')">×</button>

      <button class="calc-btn dark" @click="appendNumber(4)">4</button>
      <button class="calc-btn dark" @click="appendNumber(5)">5</button>
      <button class="calc-btn dark" @click="appendNumber(6)">6</button>
      <button class="calc-btn orange" @click="setOperator('-')">−</button>

      <button class="calc-btn dark" @click="appendNumber(1)">1</button>
      <button class="calc-btn dark" @click="appendNumber(2)">2</button>
      <button class="calc-btn dark" @click="appendNumber(3)">3</button>
      <button class="calc-btn orange" @click="setOperator('+')">+</button>

      <button class="calc-btn dark zero" @click="appendNumber(0)">0</button>
      <button class="calc-btn dark" @click="appendDot">.</button>
      <button class="calc-btn orange" @click="calculate">=</button>
    </div>
  </div>
</template>

<style scoped>
.calculator-window {
  position: fixed;
  width: 232px;
  background: rgba(30, 30, 30, 0.95);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 10px;
  box-shadow: 0 20px 50px rgba(0,0,0,0.5), 0 0 0 1px rgba(255,255,255,0.1);
  overflow: hidden;
  user-select: none;
  padding-bottom: 12px;
}

.title-bar {
  height: 28px;
  display: flex;
  align-items: center;
  padding-left: 12px;
}

.window-controls {
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

.display {
  color: white;
  text-align: right;
  padding: 0 12px;
  font-weight: 300;
  margin-bottom: 10px;
  height: 80px;
  line-height: 80px;
  overflow: hidden;
  white-space: nowrap;
}

.buttons-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1px;
  padding: 0 12px;
}

.calc-btn {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: none;
  font-size: 20px;
  font-weight: 400;
  cursor: pointer;
  margin-bottom: 10px;
  transition: filter 0.1s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.calc-btn:active {
  filter: brightness(1.3);
}

.calc-btn.gray {
  background: #a5a5a5;
  color: black;
}

.calc-btn.dark {
  background: #333333;
  color: white;
}

.calc-btn.orange {
  background: #ff9f0a;
  color: white;
  font-size: 24px;
  padding-bottom: 4px;
}

.calc-btn.zero {
  grid-column: span 2;
  width: 102px;
  border-radius: 25px;
  justify-content: flex-start;
  padding-left: 20px;
}
</style>
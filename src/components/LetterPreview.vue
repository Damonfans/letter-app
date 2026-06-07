<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import html2canvas from 'html2canvas'

const props = defineProps({
  message: String,
  selectedPaper: String,
  selectedFont: String,
  papers: Object,
  fonts: Object,
  stickers: {
    type: Array,
    default: () => []
  }
})

const currentFontFamily = computed(() => {
  if (props.selectedFont && props.fonts?.[props.selectedFont]) {
    return props.fonts[props.selectedFont].family
  }
  return '油茶馓子体'
})

const emit = defineEmits(['update-stickers'])

const letterPreview = ref(null)

const currentDate = computed(() => {
  const now = new Date()
  const year = now.getFullYear()
  const month = now.getMonth() + 1
  const day = now.getDate()
  return `${year}年${month}月${day}日`
})

const localStickers = ref([])
const activeStickerId = ref(null)
const isDraggingSticker = ref(false)
const dragStartPos = ref({ x: 0, y: 0 })
const stickerStartPos = ref({ x: 0, y: 0 })
const isGenerating = ref(false)
const generateProgress = ref(0)

const setActiveSticker = (id) => {
  activeStickerId.value = id
}

const startDragSticker = (event, sticker) => {
  if (event.button === 2) return
  
  event.preventDefault()
  event.stopPropagation()
  
  isDraggingSticker.value = true
  activeStickerId.value = sticker.id
  dragStartPos.value = { x: event.clientX, y: event.clientY }
  stickerStartPos.value = { x: sticker.x, y: sticker.y }
}

const startRotateSticker = (event, sticker) => {
  event.preventDefault()
  event.stopPropagation()
  
  isRotating.value = true
  activeStickerId.value = sticker.id
  
  const rect = letterPreview.value.getBoundingClientRect()
  const centerX = rect.left + sticker.x
  const centerY = rect.top + sticker.y
  
  stickerStartRotation.value = sticker.rotation || 0
  rotateStartAngle.value = Math.atan2(event.clientY - centerY, event.clientX - centerX) * (180 / Math.PI)
}

const handleRightDrag = (event) => {
  if (!isRotating.value) return
  
  const sticker = localStickers.value.find(s => s.id === activeStickerId.value)
  if (!sticker) return

  const rect = letterPreview.value.getBoundingClientRect()
  const centerX = rect.left + sticker.x
  const centerY = rect.top + sticker.y
  
  const currentAngle = Math.atan2(event.clientY - centerY, event.clientX - centerX) * (180 / Math.PI)
  const deltaAngle = currentAngle - rotateStartAngle.value
  
  sticker.rotation = stickerStartRotation.value + deltaAngle
  
  console.log('拖拽旋转中 - 贴纸ID:', sticker.id, 
    '当前角度:', currentAngle.toFixed(2), 
    '角度差:', deltaAngle.toFixed(2), 
    '最终旋转:', sticker.rotation.toFixed(2))
}

const handleMouseMove = (event) => {
  if (isDraggingSticker.value) {
    const sticker = localStickers.value.find(s => s.id === activeStickerId.value)
    if (!sticker) return

    const dx = event.clientX - dragStartPos.value.x
    const dy = event.clientY - dragStartPos.value.y
    
    sticker.x = stickerStartPos.value.x + dx
    sticker.y = stickerStartPos.value.y + dy
  }
}

const handleMouseUp = () => {
  if (isDraggingSticker.value) {
    isDraggingSticker.value = false
    emit('update-stickers', [...localStickers.value])
    
    console.log('鼠标松开 - 当前所有贴纸状态:', JSON.parse(JSON.stringify(localStickers.value.map(s => ({
      id: s.id,
      rotation: s.rotation,
      x: s.x,
      y: s.y,
      width: s.width,
      height: s.height
    })))))
  }
}

const handleWheel = (event, stickerId) => {
  event.preventDefault()
  event.stopPropagation()
  
  const sticker = localStickers.value.find(s => s.id === stickerId)
  if (!sticker) return

  if (event.ctrlKey || event.metaKey) {
    const scaleDelta = event.deltaY > 0 ? -0.1 : 0.1
    const baseWidth = sticker.baseWidth || sticker.width || 80
    const baseHeight = sticker.baseHeight || sticker.height || 80
    
    const newScale = Math.max(0.3, Math.min(5, (sticker.scale || 1) + scaleDelta))
    sticker.scale = newScale
    sticker.width = baseWidth * newScale
    sticker.height = baseHeight * newScale
    
    console.log('Ctrl+滚轮缩放 - 贴纸ID:', sticker.id, '缩放:', newScale.toFixed(2))
  } else {
    const angles = [0, 45, 90, 180, -90, -45]
    const currentRotation = sticker.rotation || 0
    
    let currentIndex = angles.findIndex(angle => Math.abs(angle - currentRotation) < 5)
    if (currentIndex === -1) currentIndex = 0
    
    let newIndex
    if (event.deltaY > 0) {
      newIndex = (currentIndex + 1) % angles.length
    } else {
      newIndex = (currentIndex - 1 + angles.length) % angles.length
    }
    
    sticker.rotation = angles[newIndex]
    
    console.log('滚轮旋转 - 贴纸ID:', sticker.id, '旋转角度:', sticker.rotation, '度')
  }
}

const handleDrop = (event) => {
  event.preventDefault()
  
  const previewRect = letterPreview.value.getBoundingClientRect()
  const src = event.dataTransfer.getData('sticker-src') || ''

  if (src) {
    const img = new Image()
    img.onload = () => {
      const maxSize = 100
      let width = img.width
      let height = img.height
      
      if (width > maxSize || height > maxSize) {
        const ratio = Math.min(maxSize / width, maxSize / height)
        width = width * ratio
        height = height * ratio
      }
      
      const sticker = {
        id: Date.now() + Math.random(),
        src: src,
        x: event.clientX - previewRect.left - width / 2,
        y: event.clientY - previewRect.top - height / 2,
        rotation: Math.random() * 30 - 15,
        scale: 1,
        baseWidth: width,
        baseHeight: height,
        width: width,
        height: height
      }
      localStickers.value.push(sticker)
      emit('update-stickers', [...localStickers.value])
      scheduleAutoGenerate()
    }
    img.src = src
    return
  }

  const files = event.dataTransfer.files
  if (files.length > 0) {
    const file = files[0]
    if (file.type.startsWith('image/')) {
      const reader = new FileReader()
      reader.onload = (e) => {
        const img = new Image()
        img.onload = () => {
          const maxSize = 100
          let width = img.width
          let height = img.height
          
          if (width > maxSize || height > maxSize) {
            const ratio = Math.min(maxSize / width, maxSize / height)
            width = width * ratio
            height = height * ratio
          }
          
          const sticker = {
            id: Date.now() + Math.random(),
            src: e.target.result,
            x: event.clientX - previewRect.left - width / 2,
            y: event.clientY - previewRect.top - height / 2,
            rotation: Math.random() * 30 - 15,
            scale: 1,
            baseWidth: width,
            baseHeight: height,
            width: width,
            height: height
          }
          localStickers.value.push(sticker)
          emit('update-stickers', [...localStickers.value])
          scheduleAutoGenerate()
        }
        img.src = e.target.result
      }
      reader.readAsDataURL(file)
    }
  }
}

const handleDragOver = (event) => {
  event.preventDefault()
}

const removeSticker = (stickerId) => {
  localStickers.value = localStickers.value.filter(s => s.id !== stickerId)
  emit('update-stickers', [...localStickers.value])
}

const generateLetter = async () => {
  if (!props.message.trim() && localStickers.value.length === 0) {
    alert('请先写下您的留言或添加贴纸')
    return
  }

  console.log('开始生成 - 生成前贴纸状态:', JSON.parse(JSON.stringify(localStickers.value.map(s => ({
    id: s.id,
    rotation: s.rotation,
    x: s.x,
    y: s.y,
    width: s.width,
    height: s.height
  })))))

  isGenerating.value = true
  generateProgress.value = 0

  try {
    activeStickerId.value = null
    
    generateProgress.value = 20
    await new Promise(resolve => setTimeout(resolve, 300))
    
    console.log('进度20% - 等待300ms后贴纸状态:', JSON.parse(JSON.stringify(localStickers.value.map(s => ({
      id: s.id,
      rotation: s.rotation
    })))))
    
    generateProgress.value = 40
    await new Promise(resolve => setTimeout(resolve, 300))
    
    console.log('进度40% - 等待600ms后贴纸状态:', JSON.parse(JSON.stringify(localStickers.value.map(s => ({
      id: s.id,
      rotation: s.rotation
    })))))
    
    generateProgress.value = 60
    await new Promise(resolve => setTimeout(resolve, 300))
    
    console.log('进度60% - 等待900ms后贴纸状态:', JSON.parse(JSON.stringify(localStickers.value.map(s => ({
      id: s.id,
      rotation: s.rotation
    })))))
    
    generateProgress.value = 80
    
    // 禁用贴纸动画，确保 html2canvas 读取正确的 transform 值
    const stickerElements = letterPreview.value.querySelectorAll('.sticker')
    stickerElements.forEach(el => {
      el.style.animation = 'none'
    })
    
    // 强制重绘，让浏览器应用样式更改
    await new Promise(resolve => requestAnimationFrame(resolve))
    
    const domTransforms = Array.from(stickerElements).map(el => ({
      id: el.getAttribute('data-sticker-id') || 'unknown',
      transform: el.style.transform,
      computedTransform: window.getComputedStyle(el).transform
    }))
    console.log('进度80% - DOM实际transform值:', domTransforms)
    
    const canvas = await html2canvas(letterPreview.value, {
      scale: 2,
      backgroundColor: null,
      useCORS: true,
      allowTaint: true
    })
    
    // 恢复动画（可选，组件销毁时会自动重置）
    stickerElements.forEach(el => {
      el.style.animation = ''
    })
    
    generateProgress.value = 100
    
    const link = document.createElement('a')
    link.download = `书信_${Date.now()}.png`
    link.href = canvas.toDataURL('image/png')
    link.click()
    
    await new Promise(resolve => setTimeout(resolve, 500))
  } catch (error) {
    console.error('生成失败:', error)
    alert('生成失败，请重试')
  } finally {
    isGenerating.value = false
    generateProgress.value = 0
  }
}

const getStickerStyle = (sticker) => {
  const rotation = sticker.rotation || 0
  const style = {
    left: `${sticker.x}px`,
    top: `${sticker.y}px`,
    width: `${sticker.width || 80}px`,
    height: `${sticker.height || 80}px`,
    transform: `translate(-50%, -50%) rotate(${rotation}deg)`,
    '--rotation': `${rotation}deg`  // CSS 变量供动画最终帧使用
  }
  
  console.log('计算样式 - 贴纸ID:', sticker.id, 'rotation:', sticker.rotation, 'transform:', style.transform)
  
  return style
}

onMounted(() => {
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
  window.addEventListener('contextmenu', (e) => e.preventDefault())
})

onUnmounted(() => {
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('mouseup', handleMouseUp)
})

const updateStickers = (newStickers) => {
  localStickers.value = newStickers.map(s => ({
    ...s,
    rotation: s.rotation || 0,
    scale: s.scale || 1,
    baseWidth: s.baseWidth || s.width || 80,
    baseHeight: s.baseHeight || s.height || 80,
    width: s.width || 80,
    height: s.height || 80
  }))
}

defineExpose({
  addSticker: (stickerData) => {
    const previewRect = letterPreview.value.getBoundingClientRect()
    const img = new Image()
    img.onload = () => {
      const maxSize = 100
      let width = img.width
      let height = img.height
      
      if (width > maxSize || height > maxSize) {
        const ratio = Math.min(maxSize / width, maxSize / height)
        width = width * ratio
        height = height * ratio
      }
      
      const sticker = {
        id: Date.now() + Math.random(),
        src: stickerData.src,
        x: stickerData.x - previewRect.left - width / 2,
        y: stickerData.y - previewRect.top - height / 2,
        rotation: Math.random() * 30 - 15,
        scale: 1,
        baseWidth: width,
        baseHeight: height,
        width: width,
        height: height
      }
      localStickers.value.push(sticker)
      emit('update-stickers', [...localStickers.value])
    }
    img.src = stickerData.src
  },
  updateStickers
})
</script>

<template>
  <div class="preview-section">
    <h3 class="preview-title">实时预览</h3>
    
    <div 
      class="letter-preview"
      :class="'paper-' + selectedPaper"
      ref="letterPreview"
      @drop="handleDrop"
      @dragover="handleDragOver"
    >
      <div v-if="selectedPaper === 'lined'" class="lined-paper-bg">
        <div v-for="i in 20" :key="i" class="lined-row"></div>
      </div>
      <div class="letter-date">{{ currentDate }}</div>
      <div class="letter-content" :style="{ fontFamily: currentFontFamily }">{{ message }}</div>
      
      <div 
        v-for="sticker in localStickers" 
        :key="sticker.id"
        :data-sticker-id="sticker.id"
        class="sticker"
        :class="{ 'sticker-active': activeStickerId === sticker.id }"
        :style="getStickerStyle(sticker)"
        @mousedown.left="startDragSticker($event, sticker)"
        @wheel="handleWheel($event, sticker.id)"
        @click="setActiveSticker(sticker.id)"
      >
        <img :src="sticker.src" draggable="false" />
        <button 
          v-if="activeStickerId === sticker.id" 
          class="sticker-remove"
          @click.stop="removeSticker(sticker.id)"
        >×</button>
      </div>
      
      <div class="font-info">
        <span>信纸：{{ papers?.[selectedPaper]?.name }}</span>
        <span>字体：{{ fonts?.[selectedFont]?.name || '油茶馓子体' }}</span>
      </div>
    </div>
    
    <div class="sticker-hint" v-if="localStickers.length">
      <p>💡 左键拖拽移动 · 滚轮切换角度(0°/±45°/±90°/180°) · Ctrl+滚轮缩放 · 点击删除</p>
    </div>
    
    <div class="generate-area">
      <button class="generate-btn" @click="generateLetter" :disabled="isGenerating">
        {{ isGenerating ? '生成中...' : '生成信纸' }}
      </button>
      <div v-if="isGenerating" class="generate-progress">
        <div class="generate-progress-bar">
          <div class="generate-progress-fill" :style="{ width: generateProgress + '%' }"></div>
        </div>
        <span class="generate-progress-text">{{ generateProgress }}%</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.preview-section {
  flex: 0 0 500px;
}

.preview-title {
  font-size: 1rem;
  color: #888;
  margin-bottom: 15px;
  text-align: center;
  letter-spacing: 2px;
  font-weight: normal;
}

.letter-preview {
  position: relative;
  padding: 50px 40px;
  min-height: 600px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
  transition: all 0.3s;
  overflow: hidden;
}

.letter-preview::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}

.letter-preview.paper-aged {
  background: linear-gradient(135deg, #f4e4bc 0%, #e8d4a8 50%, #dcc090 100%);
}

.letter-preview.paper-aged::before {
  background: 
    repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(139, 115, 85, 0.03) 2px,
      rgba(139, 115, 85, 0.03) 4px
    );
}

.letter-preview.paper-lined {
  background: #fff;
}

.lined-paper-bg {
  position: absolute;
  top: 50px;
  left: 0;
  right: 0;
  bottom: 50px;
  pointer-events: none;
}

.lined-row {
  height: 32px;
  border-bottom: 1px solid #c8d4e8;
  box-sizing: border-box;
}

.letter-preview.paper-kraft {
  background: linear-gradient(135deg, #c9a86c 0%, #b8956a 50%, #a88258 100%);
}

.letter-preview.paper-kraft::before {
  background: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)' opacity='0.08'/%3E%3C/svg%3E");
}

.letter-preview.paper-postcard {
  background: linear-gradient(to right, #fff 0%, #fff 65%, #f5f5f5 65%, #f5f5f5 100%);
  border: 4px solid #f0f0f0;
}

.letter-preview.paper-postcard::after {
  content: '';
  position: absolute;
  top: 30px;
  right: 30px;
  width: 60px;
  height: 70px;
  border: 2px solid #ddd;
  background: #fff;
}

.letter-date {
  text-align: right;
  margin-bottom: 40px;
  color: #666;
  font-size: 0.95rem;
}

.letter-content {
  line-height: 2.2;
  color: #3a3a3a;
  font-size: 1.2rem;
  white-space: pre-wrap;
  word-break: break-word;
  min-height: 300px;
}

.letter-content:empty::before {
  content: '在此处写下你的文字...';
  color: #bbb;
}

.paper-kraft .letter-content,
.paper-kraft .letter-date {
  color: #2d2d2d;
}

.font-info {
  display: flex;
  justify-content: space-between;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  font-size: 0.85rem;
  color: #999;
}

.sticker {
  position: absolute;
  cursor: grab;
  transition: box-shadow 0.2s;
  z-index: 10;
  animation: stickerDrop 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.sticker:active {
  cursor: grabbing;
}

.sticker img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  pointer-events: none;
  border-radius: 4px;
  filter: drop-shadow(2px 4px 6px rgba(0, 0, 0, 0.25));
  transition: filter 0.3s, transform 0.2s;
}

.sticker-active img {
  filter: drop-shadow(3px 6px 10px rgba(0, 0, 0, 0.35));
}

.sticker:hover img {
  transform: scale(1.05);
}

.sticker-active {
  z-index: 20;
}

.sticker-active img {
  filter: drop-shadow(4px 8px 12px rgba(0, 0, 0, 0.4));
}

.sticker-remove {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #ff5252;
  color: white;
  border: 2px solid white;
  cursor: pointer;
  font-size: 14px;
  line-height: 1;
  z-index: 30;
  box-shadow: 0 2px 8px rgba(255, 82, 82, 0.5);
  transition: transform 0.2s;
}

.sticker-remove:hover {
  transform: scale(1.2);
}

.generate-area {
  margin-top: 20px;
}

.generate-progress {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 10px;
}

.generate-progress-bar {
  flex: 1;
  height: 4px;
  background: #e8e0d6;
  border-radius: 2px;
  overflow: hidden;
}

.generate-progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #8b7355, #a08565);
  transition: width 0.4s ease;
  border-radius: 2px;
}

.generate-progress-text {
  font-size: 0.85rem;
  color: #8b7355;
  min-width: 2.5em;
  text-align: right;
  font-variant-numeric: tabular-nums;
}

@keyframes stickerDrop {
  0% {
    opacity: 0;
    transform: translate(-50%, -50%) rotate(0deg) scale(0.3) translateY(-50px);
  }
  50% {
    transform: translate(-50%, -50%) rotate(15deg) scale(1.1) translateY(0);
  }
  70% {
    transform: translate(-50%, -50%) rotate(-5deg) scale(0.95) translateY(0);
  }
  85% {
    transform: translate(-50%, -50%) rotate(3deg) scale(1.02) translateY(0);
  }
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) rotate(var(--rotation, 0deg)) scale(1) translateY(0);
  }
}

.sticker-hint {
  text-align: center;
  padding: 10px;
  margin-top: 10px;
}

.sticker-hint p {
  font-size: 0.85rem;
  color: #888;
  margin: 0;
}

.generate-btn {
  width: 100%;
  padding: 15px;
  background: #8b7355;
  color: white;
  border: none;
  border-radius: 4px;
  font-family: '油茶馓子体', 'STKaiti', 'KaiTi', serif;
  font-size: 1.1rem;
  letter-spacing: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.generate-btn:hover:not(:disabled) {
  background: #7a6548;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 115, 85, 0.3);
}

.generate-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

@media (max-width: 900px) {
  .preview-section {
    flex: none;
    width: 100%;
    max-width: 500px;
  }

  .letter-preview {
    padding: 30px 25px;
    min-height: 500px;
  }
}
</style>
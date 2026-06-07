<script setup>
import { ref } from 'vue'

const emit = defineEmits(['add-sticker'])

const stickers = ref([])
const isDragging = ref(false)
const draggedSticker = ref(null)
const dragOffset = ref({ x: 0, y: 0 })

const handleFileUpload = (event) => {
  const files = event.target.files
  if (!files) return

  Array.from(files).forEach(file => {
    if (file.type.startsWith('image/')) {
      const reader = new FileReader()
      reader.onload = (e) => {
        stickers.value.push({
          id: Date.now() + Math.random(),
          src: e.target.result,
          name: file.name
        })
      }
      reader.readAsDataURL(file)
    }
  })
  event.target.value = ''
}

const startDrag = (event, sticker) => {
  event.preventDefault()
  isDragging.value = true
  draggedSticker.value = sticker
  
  const rect = event.target.getBoundingClientRect()
  dragOffset.value = {
    x: event.clientX - rect.left - rect.width / 2,
    y: event.clientY - rect.top - rect.height / 2
  }

  const ghost = event.target.cloneNode(true)
  ghost.style.position = 'fixed'
  ghost.style.pointerEvents = 'none'
  ghost.style.opacity = '0.8'
  ghost.style.zIndex = '9999'
  ghost.style.width = '80px'
  ghost.style.height = '80px'
  ghost.style.objectFit = 'contain'
  ghost.id = 'sticker-ghost'
  document.body.appendChild(ghost)

  moveGhost(event)
}

const moveGhost = (event) => {
  const ghost = document.getElementById('sticker-ghost')
  if (ghost) {
    ghost.style.left = (event.clientX - 40) + 'px'
    ghost.style.top = (event.clientY - 40) + 'px'
  }
}

const endDrag = (event) => {
  if (!isDragging.value || !draggedSticker.value) return
  
  const ghost = document.getElementById('sticker-ghost')
  if (ghost) {
    ghost.remove()
  }

  emit('add-sticker', {
    id: Date.now() + Math.random(),
    src: draggedSticker.value.src,
    x: event.clientX,
    y: event.clientY
  })

  isDragging.value = false
  draggedSticker.value = null
}

const handleMouseMove = (event) => {
  if (isDragging.value) {
    moveGhost(event)
  }
}

const handleMouseUp = (event) => {
  if (isDragging.value) {
    endDrag(event)
  }
}

const removeSticker = (index) => {
  stickers.value.splice(index, 1)
}

if (typeof window !== 'undefined') {
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
}
</script>

<template>
  <div class="sticker-manager">
    <h4 class="section-title">贴纸库</h4>
    
    <label class="upload-btn">
      <input 
        type="file" 
        accept="image/*" 
        multiple 
        @change="handleFileUpload"
        hidden
      />
      <span>+ 添加贴纸</span>
    </label>

    <div class="sticker-list" v-if="stickers.length">
      <div 
        v-for="(sticker, index) in stickers" 
        :key="sticker.id"
        class="sticker-item"
        draggable="true"
        @mousedown="startDrag($event, sticker)"
      >
        <img :src="sticker.src" :alt="sticker.name" />
        <button class="remove-btn" @click.stop="removeSticker(index)">×</button>
      </div>
    </div>

    <p class="hint" v-else>上传图片作为贴纸</p>
    <p class="hint">拖拽到右侧预览区</p>
  </div>
</template>

<style scoped>
.sticker-manager {
  margin-top: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 8px;
}

.section-title {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 12px;
  font-weight: normal;
  letter-spacing: 1px;
}

.upload-btn {
  display: block;
  padding: 10px 15px;
  background: #8b7355;
  color: white;
  border-radius: 4px;
  text-align: center;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
  margin-bottom: 15px;
}

.upload-btn:hover {
  background: #7a6548;
  transform: translateY(-1px);
}

.sticker-list {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.sticker-item {
  position: relative;
  width: 60px;
  height: 60px;
  background: white;
  border-radius: 6px;
  padding: 5px;
  cursor: grab;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.sticker-item:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.sticker-item:active {
  cursor: grabbing;
}

.sticker-item img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  pointer-events: none;
}

.remove-btn {
  position: absolute;
  top: -6px;
  right: -6px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #ff5252;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 12px;
  line-height: 1;
  opacity: 0;
  transition: opacity 0.2s;
}

.sticker-item:hover .remove-btn {
  opacity: 1;
}

.hint {
  font-size: 0.8rem;
  color: #999;
  margin-top: 10px;
  text-align: center;
}
</style>

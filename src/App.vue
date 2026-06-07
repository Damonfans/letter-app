<script setup>
import LetterPreview from './components/LetterPreview.vue'
import ControlPanel from './components/ControlPanel.vue'
import MessageInput from './components/MessageInput.vue'
import StickerManager from './components/StickerManager.vue'
import { ref } from 'vue'

const message = ref('')
const selectedPaper = ref('aged')
const selectedFont = ref('youcha')
const stickers = ref([])
const letterPreviewRef = ref(null)

const papers = {
  aged: { name: '泛黄旧信纸', preview: '岁月痕迹' },
  lined: { name: '横线信纸', preview: '工整有致' },
  kraft: { name: '牛皮纸', preview: '质朴厚重' },
  postcard: { name: '明信片', preview: '远方寄语' }
}

const fonts = {
  youcha: { name: '油茶馓子体', family: '油茶馓子体', preview: '温润典雅' },
  pozheng: { name: '迫真打字油印体', family: '迫真打字油印体', preview: '复古打字' }
}

const handleAddSticker = (stickerData) => {
  if (letterPreviewRef.value) {
    letterPreviewRef.value.addSticker(stickerData)
  }
}

const handleUpdateStickers = (newStickers) => {
  stickers.value = newStickers
}
</script>

<template>
  <div class="app">
    <header class="header">
      <h1>书 信</h1>
      <p>慢下来，写下值得被记住的文字</p>
    </header>

    <main class="main-container">
      <ControlPanel 
        v-model:selectedPaper="selectedPaper"
        v-model:selectedFont="selectedFont"
        :papers="papers"
        :fonts="fonts"
      >
        <MessageInput v-model="message" />
        <StickerManager @add-sticker="handleAddSticker" />
      </ControlPanel>
      <LetterPreview 
        ref="letterPreviewRef"
        :message="message"
        :selectedPaper="selectedPaper"
        :selectedFont="selectedFont"
        :papers="papers"
        :fonts="fonts"
        :stickers="stickers"
        @update-stickers="handleUpdateStickers"
      />
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 40px;
  padding: 30px 0;
}

.header h1 {
  font-size: 2.5rem;
  color: #4a4a4a;
  font-weight: normal;
  letter-spacing: 8px;
  margin-bottom: 10px;
}

.header p {
  color: #888;
  font-size: 1rem;
  letter-spacing: 2px;
}

.main-container {
  display: flex;
  gap: 40px;
  justify-content: center;
  flex-wrap: wrap;
  max-width: 1200px;
  margin: 0 auto;
}

@media (max-width: 900px) {
  .main-container {
    flex-direction: column;
    align-items: center;
  }

  .header h1 {
    font-size: 1.8rem;
  }
}
</style>

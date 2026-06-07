<script setup>
import { computed } from 'vue'

const props = defineProps({
  selectedPaper: String,
  selectedFont: String,
  papers: Object,
  fonts: Object
})

const emit = defineEmits(['update:selectedPaper', 'update:selectedFont'])

const localSelectedPaper = computed({
  get: () => props.selectedPaper,
  set: (value) => emit('update:selectedPaper', value)
})

const localSelectedFont = computed({
  get: () => props.selectedFont,
  set: (value) => emit('update:selectedFont', value)
})
</script>

<template>
  <div class="control-panel">
    <section class="section">
      <h2 class="section-title">信纸选择</h2>
      <div class="paper-options">
        <div 
          v-for="(paper, key) in papers" 
          :key="key"
          class="paper-option"
          :class="[{ selected: selectedPaper === key }, 'paper-' + key]"
          @click="localSelectedPaper = key"
        >
          <div class="preview">{{ paper.preview }}</div>
          <div class="label">{{ paper.name }}</div>
        </div>
      </div>
    </section>

    <section class="section">
      <h2 class="section-title">字体选择</h2>
      <div class="font-options">
        <div 
          v-for="(font, key) in fonts" 
          :key="key"
          class="font-option"
          :class="{ selected: selectedFont === key }"
          @click="localSelectedFont = key"
        >
          <div class="font-preview" :style="{ fontFamily: font.family }">
            书信
          </div>
          <div class="font-name">{{ font.name }}</div>
          <div class="font-desc">{{ font.preview }}</div>
        </div>
      </div>
    </section>

    <section class="section">
      <h2 class="section-title">书写留言</h2>
      <slot></slot>
    </section>
  </div>
</template>

<style scoped>
.control-panel {
  flex: 0 0 400px;
  background: rgba(255, 255, 255, 0.6);
  padding: 30px;
  border-radius: 4px;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.08);
}

.section {
  margin-bottom: 30px;
}

.section-title {
  font-size: 1.1rem;
  color: #5a5a5a;
  margin-bottom: 15px;
  padding-bottom: 8px;
  border-bottom: 1px solid #ddd;
  letter-spacing: 2px;
  font-weight: normal;
}

.paper-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.paper-option {
  position: relative;
  cursor: pointer;
  border-radius: 4px;
  overflow: hidden;
  transition: all 0.3s;
  border: 2px solid transparent;
}

.paper-option:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.paper-option.selected {
  border-color: #8b7355;
}

.paper-option .preview {
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  color: #666;
}

.paper-option .label {
  text-align: center;
  padding: 8px;
  font-size: 0.9rem;
  background: rgba(255, 255, 255, 0.8);
  color: #5a5a5a;
}

.paper-aged {
  background: linear-gradient(135deg, #f4e4bc 0%, #e8d4a8 50%, #dcc090 100%);
}

.paper-lined {
  background: #fff;
  background-image: repeating-linear-gradient(
    transparent,
    transparent 23px,
    #c8d4e8 23px,
    #c8d4e8 24px
  );
  background-position: 0 10px;
}

.paper-kraft {
  background: linear-gradient(135deg, #c9a86c 0%, #b8956a 50%, #a88258 100%);
}

.paper-postcard {
  background: linear-gradient(to right, #fff 0%, #fff 60%, #f0f0f0 60%, #f0f0f0 100%);
  border: 3px solid #f5f5f5;
}

.font-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.font-option {
  cursor: pointer;
  border-radius: 4px;
  overflow: hidden;
  transition: all 0.3s;
  border: 2px solid transparent;
  background: #fff;
}

.font-option:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.font-option.selected {
  border-color: #8b7355;
}

.font-preview {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.6rem;
  color: #4a4a4a;
  background: #faf8f4;
}

.font-name {
  text-align: center;
  padding: 6px 8px 2px;
  font-size: 0.9rem;
  color: #5a5a5a;
}

.font-desc {
  text-align: center;
  padding: 0 8px 8px;
  font-size: 0.75rem;
  color: #aaa;
}

@media (max-width: 480px) {
  .control-panel {
    flex: none;
    width: 100%;
    max-width: 500px;
    padding: 20px;
  }

  .paper-options {
    grid-template-columns: 1fr;
  }
}
</style>

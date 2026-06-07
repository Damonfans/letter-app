<script setup>
const props = defineProps({
  modelValue: String
})

const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <textarea 
    :value="modelValue"
    @input="emit('update:modelValue', $event.target.value)"
    class="message-input-area"
    placeholder="在此写下你的文字，如同在真实信纸上一般..."
    rows="8"
  ></textarea>
</template>

<style scoped>
.message-input-area {
  width: 100%;
  min-height: 200px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: '油茶馓子体', 'STKaiti', 'KaiTi', serif;
  font-size: 1.1rem;
  line-height: 2;
  resize: vertical;
  background: rgba(255, 255, 255, 0.8);
  transition: all 0.3s;
}

.message-input-area:focus {
  outline: none;
  border-color: #8b7355;
  box-shadow: 0 0 0 3px rgba(139, 115, 85, 0.1);
}

.message-input-area::placeholder {
  color: #bbb;
}
</style>

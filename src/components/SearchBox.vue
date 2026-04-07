<template>
  <div class="search-wrapper">
    <Icon icon="lucide:search" class="search-icon" />
    <input 
      v-model="model" 
      type="text" 
      :placeholder="placeholder" 
      class="search-input"
    />
    <button 
      v-if="model" 
      @click="model = ''" 
      class="clear-button"
      aria-label="Clear search"
    >
      ✕
    </button>
  </div>
</template>

<script setup>
import { Icon } from '@iconify/vue';

const model = defineModel({ type: String, default: '' })

defineProps({
  placeholder: {
    type: String,
    default: ''
  }
})
</script>

<style scoped>
.search-wrapper {
  position: relative;
  width: 100%;
  max-width: 400px;
}

.search-icon {
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  pointer-events: none;
}

.search-input {
  width: 100%;
  box-sizing: border-box;
  padding: 0.75rem 2.5rem 0.75rem 2.5rem;
  border-radius: 8px;
  border: 1px solid var(--border-color);
  background-color: var(--input-bg);
  color: var(--text-main);
  font-size: 1rem;
  transition: border-color 0.2s, background-color 0.2s, color 0.2s;
}

.search-input:focus {
  outline: none;
  border-color: var(--accent-color);
}

.clear-button {
  position: absolute;
  right: 0.5rem;
  top: 50%;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  color: var(--text-muted);
  font-size: 1rem;
  cursor: pointer;
  padding: 0.25rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.2s ease;
}

.clear-button:hover {
  color: var(--text-main);
}
</style>
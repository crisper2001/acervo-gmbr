<template>
  <div class="pagination">
    <button
      :disabled="currentPage === 1"
      @click="$emit('update:currentPage', currentPage - 1)"
      class="page-btn nav-btn"
    >
      &laquo;
    </button>
    
    <button
      v-for="(page, index) in visiblePages"
      :key="index"
      :class="['page-btn', { active: page === currentPage, dots: page === '...' }]"
      :disabled="page === '...'"
      @click="page !== '...' && $emit('update:currentPage', page)"
    >
      {{ page }}
    </button>
    
    <button
      :disabled="currentPage === totalPages"
      @click="$emit('update:currentPage', currentPage + 1)"
      class="page-btn nav-btn"
    >
      &raquo;
    </button>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  currentPage: {
    type: Number,
    required: true
  },
  totalPages: {
    type: Number,
    required: true
  }
});

defineEmits(['update:currentPage']);

const visiblePages = computed(() => {
  const pages = [];
  const total = props.totalPages;
  const current = props.currentPage;
  
  if (total <= 7) {
    for (let i = 1; i <= total; i++) pages.push(i);
  } else {
    if (current <= 4) {
      pages.push(1, 2, 3, 4, 5, '...', total);
    } else if (current >= total - 3) {
      pages.push(1, '...', total - 4, total - 3, total - 2, total - 1, total);
    } else {
      pages.push(1, '...', current - 1, current, current + 1, '...', total);
    }
  }
  return pages;
});
</script>

<style scoped>
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  margin-top: 3rem;
  flex-wrap: wrap;
}

.page-btn {
  background-color: var(--input-bg, #ffffff);
  color: var(--text-main, #1a1a1a);
  border: 1px solid var(--border-color, #cccccc);
  min-width: 2.5rem;
  height: 2.5rem;
  padding: 0 0.5rem;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.page-btn:hover:not(:disabled):not(.active) {
  border-color: var(--accent-color, #42b883);
  color: var(--accent-color, #42b883);
}

.page-btn.active {
  background-color: var(--accent-color, #42b883);
  color: #1a1a1a;
  border-color: var(--accent-color, #42b883);
}

.page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-btn.dots {
  border: none;
  background: transparent;
  opacity: 1;
  cursor: default;
  color: var(--text-muted, #666666);
}
</style>
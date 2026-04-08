<template>
  <div class="item-card" @click="$emit('open', item)">
    <div class="thumbnail-wrapper">
      <div v-if="!isLoaded" class="skeleton skeleton-overlay"></div>
      <img :src="item.thumbnail" :alt="item.title" :class="['thumbnail', { 'img-loaded': isLoaded }]" @load="isLoaded = true" @error="handleImageError" loading="lazy" />
    </div>
    <div class="info">
      <div class="header">
        <h3 class="title" :title="item.title">{{ item.title }}</h3>
        <span :class="['grade', item.grade ? `grade-${item.grade.toLowerCase()}` : 'grade-na']">
          {{ item.grade || 'N/A' }}
        </span>
      </div>
      <div class="meta">
        <div class="meta-row">
          <span class="author clickable" @click.stop="$emit('search-author', item.author)">
            <Icon icon="lucide:user" />
            {{ item.author }}
          </span>
          <span class="year clickable" @click.stop="$emit('search-year', item.year.toString())">
            <Icon icon="lucide:calendar" />
            {{ item.year }}
          </span>
        </div>
        <span class="tag-label">
          {{ item.tag ? (labels[item.tag.toLowerCase()] || item.tag) : labels.project }}
        </span>
      </div>
      
      <!-- Action button to external download/play link -->
      <a v-if="item.download" :href="item.download" target="_blank" rel="noopener noreferrer" class="download-btn" @click.stop>
        <Icon icon="lucide:download" />
        {{ labels.playNow }}
      </a>
      <span v-else class="download-btn disabled" @click.stop>
        <Icon icon="lucide:x-circle" />
        {{ labels.unavailable }}
      </span>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { Icon } from '@iconify/vue';

const isLoaded = ref(false);

defineProps({
  item: {
    type: Object,
    required: true
  },
  labels: {
    type: Object,
    default: () => ({ playNow: 'Download', unavailable: 'Unavailable', project: 'Project', engine: 'Engine' })
  }
});

defineEmits(['search-author', 'search-year', 'open']);

const handleImageError = (e) => {
  const src = e.target.src;
  if (src.endsWith('.png')) {
    e.target.src = src.replace('.png', '.jpg');
  } else if (src.endsWith('.jpg')) {
    e.target.src = 'data:image/svg+xml;charset=UTF-8,%3Csvg xmlns="http://www.w3.org/2000/svg" width="240" height="240" viewBox="0 0 240 240"%3E%3Crect fill="%23333" width="240" height="240"/%3E%3Ctext fill="%23888" font-family="sans-serif" font-size="16" dy="8" font-weight="bold" x="50%25" y="50%25" text-anchor="middle"%3ENo Image%3C/text%3E%3C/svg%3E';
  }
};
</script>

<style scoped>
.item-card {
  display: flex;
  flex-direction: column;
  background: var(--card-bg, #2a2a2a);
  border: 3px solid transparent;
  border-radius: 8px;
  overflow: hidden;
  transition: border-color 0.2s ease;
  color: var(--text-main, #ffffff);
  cursor: pointer;
}

.item-card:hover {
  border-color: var(--accent-color, #42b883);
}

.thumbnail-wrapper {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
}

.skeleton-overlay {
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  z-index: 1;
}

.thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  color: transparent;
}

.thumbnail:not(.img-loaded) {
  opacity: 0;
}

.info {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.25rem;
  gap: 0.5rem;
}

.title {
  margin: 0;
  font-size: 1.2rem;
  font-weight: bold;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.grade {
  display: inline-block;
  padding: 0.15rem 0.5rem;
  border-radius: 4px;
  font-size: 0.85rem;
  font-weight: bold;
  flex-shrink: 0;
}

.grade-a {
  background-color: #4CAF50;
  color: white;
}

.grade-b {
  background-color: #2196F3;
  color: white;
}

.grade-c {
  background-color: #FFEB3B;
  color: black;
}

.grade-d {
  background-color: #FF9800;
  color: white;
}

.grade-e {
  background-color: #F44336;
  color: white;
}

.grade-na {
  background-color: #9e9e9e;
  color: white;
}

.meta {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  font-size: 0.8rem;
  color: var(--text-muted, #888);
  margin-bottom: 0.75rem;
}

.meta-row {
  display: flex;
  justify-content: space-between;
}

.author, .year {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.tag-label {
  align-self: flex-start;
  font-size: 0.65rem;
  text-transform: uppercase;
  color: var(--text-muted);
  border: 1px solid var(--border-color);
  padding: 0.1rem 0.4rem;
  border-radius: 12px;
}

.clickable {
  cursor: pointer;
  transition: color 0.2s ease;
}

.clickable:hover {
  color: var(--accent-color, #42b883);
}

.download-btn {
  margin-top: auto;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  background-color: var(--accent-color, #42b883);
  color: #1a1a1a;
  text-align: center;
  padding: 0.75rem;
  text-decoration: none;
  font-weight: bold;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.download-btn:hover {
  background-color: var(--accent-hover, #33a06f);
}

.download-btn.disabled {
  background-color: var(--border-color);
  color: var(--text-muted);
  cursor: not-allowed;
}

.download-btn.disabled:hover {
  background-color: var(--border-color);
}
</style>

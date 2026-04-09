<template>
  <div class="item-card" @click="$emit('open', item)">
    <div class="thumbnail-wrapper">
      <div v-if="!isLoaded" class="skeleton skeleton-overlay"></div>
      <img :src="item.thumbnail" :alt="item.title" :class="['thumbnail', { 'img-loaded': isLoaded }]" @load="isLoaded = true" @error="handleImageError" loading="lazy" />
    </div>
    <div class="info">
      <div class="header">
        <h3 class="title" :title="item.title">{{ item.title }}</h3>
        <div v-if="item.grade" class="grade-badge">
          <img :src="getGradeImage(item.grade)" :alt="`Grade ${item.grade}`" class="grade-img" />
        </div>
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

const getGradeImage = (grade) => {
  const gradeName = grade ? grade.toLowerCase() : 'na';
  return `${import.meta.env.BASE_URL}grades/${gradeName}.png`;
};
</script>

<style scoped>
.item-card {
  display: flex;
  flex-direction: column;
  background: var(--card-bg, #2a2a2a);
  border: 3px solid transparent;
  border-radius: 12px;
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
  align-items: center;
  margin-bottom: 0.25rem;
  gap: 0.5rem;
}

.title {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 800;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
}

.grade-badge {
  height: 32px;
  width: 32px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.grade-img {
  width: 13px;
  height: 19px;
  object-fit: contain;

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
  font-weight: bold;
  text-transform: uppercase;
  color: var(--text-muted);
  background: var(--bg-color);
  padding: 0.2rem 0.6rem;
  border-radius: 16px;
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
  padding: 0.85rem;
  text-decoration: none;
  font-weight: bold;
  border-radius: 8px;
  letter-spacing: 0.5px;
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

<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content">
      <button class="close-btn" @click="$emit('close')">&times;</button>

      <div class="modal-body">
        <div class="image-gallery">
          <div v-if="isLoadingCurrent" class="skeleton skeleton-fill"></div>

          <button v-if="!isChecking && validImages.length > 1" class="carousel-btn prev" @click="prevImage"
            :disabled="currentImageIndex === 0">
            &#10094;
          </button>

          <TransitionGroup :name="slideDirection" tag="div" class="carousel-container">
            <img v-for="(img, index) in imageSources" :key="img.id"
              v-show="!img.failed && validImages.indexOf(img) === currentImageIndex" :src="img.url" loading="lazy"
              :alt="`${item.title} screenshot ${index + 1}`" :class="['gallery-image', { 'img-loaded': img.loaded }]"
              @load="img.loaded = true" @error="handleGalleryError(index, $event)" />
            <img v-if="!isChecking && validImages.length === 0" key="fallback-thumbnail" :src="item.thumbnail"
              :alt="`${item.title} thumbnail`" :class="['gallery-image', { 'img-loaded': fallbackLoaded }]" 
              @load="fallbackLoaded = true" @error="handleFallbackError" />
          </TransitionGroup>

          <button v-if="!isChecking && validImages.length > 1" class="carousel-btn next" @click="nextImage"
            :disabled="currentImageIndex === validImages.length - 1">
            &#10095;
          </button>

          <div v-if="!isChecking && validImages.length > 1" class="carousel-indicators">
            <span v-for="(img, index) in validImages" :key="index"
              :class="['indicator', { active: index === currentImageIndex }]" @click="goToImage(index)"></span>
          </div>
        </div>
        <div class="modal-info">
          <div class="modal-header">
            <h2>{{ item.title }}</h2>
            <span :class="['grade', item.grade ? `grade-${item.grade.toLowerCase()}` : 'grade-na']">
              {{ item.grade || 'N/A' }}
            </span>
          </div>

          <div class="modal-meta">
            <div class="meta-item">
              <span class="meta-label">{{ labels.author }}</span>
              <span class="meta-value">{{ item.author }}</span>
            </div>
            <div class="meta-item">
              <span class="meta-label">{{ labels.year }}</span>
              <span class="meta-value">{{ item.year }}</span>
            </div>
            <div class="meta-item">
              <span class="meta-label">{{ labels.tag }}</span>
              <span class="meta-value">{{ item.tag ? (labels[item.tag.toLowerCase()] || item.tag) : labels.project }}</span>
            </div>
          </div>

          <div class="modal-actions">
            <a v-if="item.download" :href="item.download" target="_blank" rel="noopener noreferrer"
              class="download-btn">
              <Icon icon="lucide:download" />
              {{ labels.playNow }}
            </a>
            <span v-else class="download-btn disabled">
              <Icon icon="lucide:x-circle" />
              {{ labels.unavailable }}
            </span>
            <a v-if="item.topic" :href="item.topic" target="_blank" rel="noopener noreferrer" class="forum-btn">
              <Icon icon="lucide:external-link" />
              {{ labels.forumLink }}
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref, computed } from 'vue';
import { Icon } from '@iconify/vue';

const props = defineProps({
  item: {
    type: Object,
    required: true
  },
  labels: {
    type: Object,
    default: () => ({ playNow: 'Download', unavailable: 'Unavailable', author: 'Author', year: 'Year', forumLink: 'Forum Topic', tag: 'Tag', project: 'Project', engine: 'Engine' })
  }
});

const emit = defineEmits(['close']);

const currentImageIndex = ref(0);
const slideDirection = ref('slide-right');
const isChecking = ref(true);
const fallbackLoaded = ref(false);

const imageSources = ref(
  Array.from({ length: 5 }, (_, i) => ({
    id: i,
    url: `${import.meta.env.BASE_URL}projects/${props.item.name}/${i + 1}.png`,
    failed: false,
    loaded: false
  }))
);

const validImages = computed(() => imageSources.value.filter(img => !img.failed));

const isLoadingCurrent = computed(() => {
  if (isChecking.value) return true;
  if (validImages.value.length === 0) return !fallbackLoaded.value;
  const currentImg = validImages.value[currentImageIndex.value];
  return currentImg ? !currentImg.loaded : false;
});

const checkImages = async () => {
  const promises = imageSources.value.map(async (img) => {
    try {
      let response = await fetch(img.url, { method: 'HEAD' });
      if (response.ok && response.headers.get('content-type')?.startsWith('image/')) return;

      const jpgUrl = img.url.replace('.png', '.jpg');
      response = await fetch(jpgUrl, { method: 'HEAD' });
      if (response.ok && response.headers.get('content-type')?.startsWith('image/')) {
        img.url = jpgUrl;
        return;
      }

      img.failed = true;
    } catch (e) {
      img.failed = true;
    }
  });

  await Promise.all(promises);
  isChecking.value = false;

  if (currentImageIndex.value >= validImages.value.length) {
    currentImageIndex.value = Math.max(0, validImages.value.length - 1);
  }
};

checkImages();

const nextImage = () => {
  if (currentImageIndex.value < validImages.value.length - 1) {
    slideDirection.value = 'slide-right';
    currentImageIndex.value++;
  }
};

const prevImage = () => {
  if (currentImageIndex.value > 0) {
    slideDirection.value = 'slide-left';
    currentImageIndex.value--;
  }
};

const goToImage = (index) => {
  slideDirection.value = index > currentImageIndex.value ? 'slide-right' : 'slide-left';
  currentImageIndex.value = index;
};

const handleKeyDown = (e) => {
  if (e.key === 'Escape') {
    emit('close');
  } else if (e.key === 'ArrowRight') {
    nextImage();
  } else if (e.key === 'ArrowLeft') {
    prevImage();
  }
};

onMounted(() => {
  document.body.style.overflow = 'hidden';
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  document.body.style.overflow = '';
  window.removeEventListener('keydown', handleKeyDown);
});

const handleGalleryError = (index, e) => {
  const src = e.target.src;
  if (src.endsWith('.png')) {
    imageSources.value[index].url = src.replace('.png', '.jpg');
  } else {
    imageSources.value[index].failed = true;
    if (currentImageIndex.value >= validImages.value.length) {
      currentImageIndex.value = Math.max(0, validImages.value.length - 1);
    }
  }
};

const handleFallbackError = (e) => {
  const src = e.target.src;
  if (src.endsWith('.png')) {
    e.target.src = src.replace('.png', '.jpg');
  } else if (src.endsWith('.jpg')) {
    e.target.src = 'data:image/svg+xml;charset=UTF-8,%3Csvg xmlns="http://www.w3.org/2000/svg" width="240" height="240" viewBox="0 0 240 240"%3E%3Crect fill="%23333" width="240" height="240"/%3E%3Ctext fill="%23888" font-family="sans-serif" font-size="16" dy="8" font-weight="bold" x="50%25" y="50%25" text-anchor="middle"%3ENo Image%3C/text%3E%3C/svg%3E';
  }
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1rem;
  backdrop-filter: blur(4px);
}

.modal-content {
  background: var(--card-bg, #2a2a2a);
  border-radius: 12px;
  padding: 2rem;
  width: 900px;
  height: 600px;
  max-width: 95vw;
  max-height: 95vh;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  position: relative;
  color: var(--text-main, #ffffff);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 20px;
  background: none;
  border: none;
  font-size: 1.8rem;
  color: var(--text-muted);
  cursor: pointer;
  transition: color 0.2s;
}

.close-btn:hover {
  color: var(--text-main);
}

.modal-body {
  display: flex;
  gap: 2rem;
  margin-top: 1rem;
  flex: 1;
  min-height: 0;
}

.image-gallery {
  flex: 1.5;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  overflow: hidden;
  height: 100%;
}

.skeleton-fill {
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  z-index: 5;
  border-radius: 8px;
}

/* Carousel styles */
.carousel-container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  position: relative;
}

.gallery-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  color: transparent;
}

.gallery-image:not(.img-loaded) {
  opacity: 0;
}

/* Slide animations */
.slide-right-enter-active,
.slide-right-leave-active,
.slide-left-enter-active,
.slide-left-leave-active {
  transition: all 0.4s ease;
}

.slide-right-leave-active,
.slide-left-leave-active {
  position: absolute;
}

.slide-right-enter-from {
  opacity: 0;
  transform: translateX(30px);
}

.slide-right-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.slide-left-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.slide-left-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  font-size: 1.5rem;
  padding: 1rem 0.5rem;
  cursor: pointer;
  z-index: 10;
  transition: background 0.2s;
}

.carousel-btn:hover:not(:disabled) {
  background: rgba(0, 0, 0, 0.8);
}

.carousel-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.carousel-btn.prev {
  left: 0;
  border-radius: 0 4px 4px 0;
}

.carousel-btn.next {
  right: 0;
  border-radius: 4px 0 0 4px;
}

.carousel-indicators {
  position: absolute;
  bottom: 1rem;
  display: flex;
  gap: 0.5rem;
  z-index: 10;
}

.indicator {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: background 0.2s;
}

.indicator.active {
  background: rgba(255, 255, 255, 0.9);
}

.modal-info {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.modal-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.modal-header h2 {
  margin: 0;
  font-size: 1.6rem;
}

.grade {
  display: inline-block;
  padding: 0.15rem 0.5rem;
  border-radius: 4px;
  font-size: 0.9rem;
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

.modal-meta {
  display: flex;
  gap: 2rem;
  margin-bottom: 2rem;
  background: var(--bg-color);
  padding: 1.2rem;
  border-radius: 8px;
  border: 1px solid var(--border-color);
}

.meta-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.meta-label {
  font-size: 0.85rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  font-weight: bold;
}

.meta-value {
  font-size: 1.1rem;
  font-weight: bold;
  color: var(--text-main);
}

.modal-actions {
  margin-top: auto;
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.download-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  background-color: var(--accent-color, #42b883);
  color: #1a1a1a;
  text-align: center;
  padding: 0.8rem 1.5rem;
  text-decoration: none;
  font-weight: bold;
  border-radius: 6px;
  transition: background-color 0.2s ease;
  font-size: 1.1rem;
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

.forum-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  background-color: transparent;
  color: var(--text-main);
  border: 1px solid var(--border-color);
  text-align: center;
  padding: calc(0.8rem - 1px) 1.5rem;
  text-decoration: none;
  font-weight: bold;
  border-radius: 6px;
  transition: all 0.2s ease;
  font-size: 1.1rem;
}

.forum-btn:hover {
  border-color: var(--accent-color);
  color: var(--accent-color);
}

@media (max-width: 768px) {
  .modal-body {
    flex-direction: column;
  }

  .image-gallery {
    flex: none;
    height: 300px;
  }
}

/* Transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.9);
}
</style>
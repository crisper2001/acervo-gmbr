<template>
  <div class="lightbox-overlay" @click.self="closeLightbox">
    <button class="lightbox-close" @click="closeLightbox">
      <Icon icon="lucide:x" />
    </button>
    
    <button v-if="images.length > 1" class="lightbox-btn prev" @click.stop="prevImage" :disabled="currentIndex === 0">
      <Icon icon="lucide:chevron-left" />
    </button>
    
    <img 
      :src="currentImageUrl" 
      draggable="false"
      :class="['lightbox-image', { zoomed: isZoomed, dragging: isDragging }]" 
      :style="imageStyle"
      @pointerdown.prevent.stop="startDrag"
      @pointermove.prevent.stop="onDrag"
      @pointerup.prevent.stop="endDrag"
      @pointercancel.prevent.stop="endDrag"
      @click.stop
    />
    
    <button v-if="images.length > 1" class="lightbox-btn next" @click.stop="nextImage" :disabled="currentIndex === images.length - 1">
      <Icon icon="lucide:chevron-right" />
    </button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { Icon } from '@iconify/vue';

const props = defineProps({
  images: {
    type: Array,
    required: true
  },
  initialIndex: {
    type: Number,
    default: 0
  }
});

const emit = defineEmits(['close']);

const currentIndex = ref(props.initialIndex);
const isZoomed = ref(false);

const isDragging = ref(false);
const hasDragged = ref(false);
const panX = ref(0);
const panY = ref(0);
let startPointerX = 0;
let startPointerY = 0;
let startX = 0;
let startY = 0;

const currentImageUrl = computed(() => {
  const img = props.images[currentIndex.value];
  return typeof img === 'string' ? img : img?.url;
});

const imageStyle = computed(() => {
  return {
    // We use translate before scale to maintain 1:1 cursor tracking in physical pixels
    transform: isZoomed.value ? `translate(${panX.value}px, ${panY.value}px) scale(1.75)` : 'translate(0px, 0px) scale(1)',
    transition: isDragging.value ? 'none' : 'transform 0.3s ease'
  };
});

const nextImage = () => {
  if (currentIndex.value < props.images.length - 1) {
    currentIndex.value++;
    isZoomed.value = false;
    panX.value = 0;
    panY.value = 0;
  }
};

const prevImage = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--;
    isZoomed.value = false;
    panX.value = 0;
    panY.value = 0;
  }
};

const toggleZoom = () => {
  isZoomed.value = !isZoomed.value;
  if (!isZoomed.value) {
    panX.value = 0;
    panY.value = 0;
  }
};

const startDrag = (e) => {
  isDragging.value = true;
  hasDragged.value = false;
  startPointerX = e.clientX;
  startPointerY = e.clientY;
  startX = e.clientX - panX.value;
  startY = e.clientY - panY.value;
  
  if (e.target.setPointerCapture) {
    e.target.setPointerCapture(e.pointerId);
  }
};

const onDrag = (e) => {
  if (!isDragging.value) return;
  
  const moveX = e.clientX - startPointerX;
  const moveY = e.clientY - startPointerY;
  
  // Threshold to differentiate a tiny micro-movement (clicking) from an actual intent to drag
  if (Math.abs(moveX) > 5 || Math.abs(moveY) > 5) {
    hasDragged.value = true;
  }
  
  if (isZoomed.value) {
    panX.value = e.clientX - startX;
    panY.value = e.clientY - startY;
  }
};

const endDrag = (e) => {
  if (!isDragging.value) return;
  isDragging.value = false;
  
  if (e.target.hasPointerCapture && e.target.hasPointerCapture(e.pointerId)) {
    e.target.releasePointerCapture(e.pointerId);
  }
  
  if (!hasDragged.value && e.type === 'pointerup') {
    toggleZoom();
  }
};

const closeLightbox = () => {
  emit('close');
};

const handleKeyDown = (e) => {
  if (e.key === 'Escape') {
    closeLightbox();
  } else if (e.key === 'ArrowRight') {
    nextImage();
  } else if (e.key === 'ArrowLeft') {
    prevImage();
  }
};

onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script>

<style scoped>
.lightbox-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  backdrop-filter: blur(5px);
}

.lightbox-image {
  max-width: 90vw;
  max-height: 90vh;
  object-fit: contain;
  box-shadow: 0 5px 25px rgba(0, 0, 0, 0.5);
  cursor: zoom-in;
  user-select: none;
  -webkit-user-drag: none;
}

.lightbox-image.zoomed {
  cursor: grab;
}

.lightbox-image.zoomed.dragging {
  cursor: grabbing;
}

.lightbox-close {
  position: absolute;
  top: 20px;
  right: 25px;
  background: none;
  border: none;
  color: white;
  font-size: 2.5rem;
  cursor: pointer;
  z-index: 2010;
  transition: color 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightbox-close:hover {
  color: #cccccc;
}

.lightbox-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: white;
  font-size: 2rem;
  cursor: pointer;
  transition: background 0.2s;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2010;
}

.lightbox-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.2);
}

.lightbox-btn:disabled {
  opacity: 0.2;
  cursor: not-allowed;
}

.lightbox-btn.prev {
  left: 30px;
}

.lightbox-btn.next {
  right: 30px;
}
</style>
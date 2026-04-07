<template>
  <div class="app-container">
    <header class="header">
      <div class="header-left">
        <h1>Acervo GMBR</h1>
      </div>

      <div class="search-container">
        <SearchBox v-model="searchQuery" :placeholder="t('searchPlaceholder')" />
        <select v-model="sortBy" class="sort-select">
          <option value="name-asc">{{ t('sortAZ') }}</option>
          <option value="name-desc">{{ t('sortZA') }}</option>
          <option value="year-desc">{{ t('sortNewest') }}</option>
          <option value="year-asc">{{ t('sortOldest') }}</option>
          <option value="grade-asc">{{ t('sortGradeAsc') }}</option>
          <option value="grade-desc">{{ t('sortGradeDesc') }}</option>
        </select>
      </div>

      <div class="controls">
        <button @click="toggleTheme" class="control-btn">
          <Icon :icon="theme === 'light' ? 'lucide:moon' : 'lucide:sun'" />
          {{ theme === 'light' ? t('themeDark') : t('themeLight') }}
        </button>
        <button @click="toggleLanguage" class="control-btn">
          <Icon :icon="locale === 'pt' ? 'circle-flags:us' : 'circle-flags:br'" />
          {{ locale === 'pt' ? 'EN' : 'PT' }}
        </button>
      </div>
    </header>

    <main>
      <div v-if="loading" class="loading-state">
        <p>{{ t('loading') }}</p>
      </div>
      
      <template v-else>
        <div class="results-summary">
          <span class="item-count">{{ t('projectsFound', { count: filteredItems.length }) }}</span>
        </div>

        <Pagination 
          v-if="totalPages > 1"
          :current-page="currentPage"
          :total-pages="totalPages"
          @update:current-page="setPage"
          class="top-pagination"
        />

        <div class="items-grid">
          <ItemCard 
            v-for="item in paginatedItems" 
            :key="item.id" 
            :item="item" 
            :labels="{ playNow: t('playNow'), unavailable: t('downloadUnavailable'), project: t('project'), engine: t('engine') }"
            @search-author="setSearchQuery"
            @search-year="setSearchQuery"
            @open="selectedItem = $event"
          />
          <div v-for="n in (itemsPerPage - paginatedItems.length)" :key="'placeholder-' + n" class="item-placeholder" aria-hidden="true">
            <div class="placeholder-thumb"></div>
            <div class="placeholder-info"></div>
          </div>
        </div>
      </template>

      <Pagination 
        v-if="!loading && totalPages > 1"
        :current-page="currentPage"
        :total-pages="totalPages"
        @update:current-page="setPage"
      />
    </main>
    
    <Transition name="modal">
      <ItemModal 
        v-if="selectedItem" 
        :item="selectedItem" 
        :labels="{ playNow: t('playNow'), unavailable: t('downloadUnavailable'), author: t('author'), year: t('year'), forumLink: t('forumLink'), tag: t('tag'), project: t('project'), engine: t('engine') }"
        @close="selectedItem = null" 
      />
    </Transition>

    <footer class="footer">
      <small>
        <p>Super Games</p>
        <p>2026</p>
      </small>      
      <p><small>{{ t('footerDisclaimer') }}</small></p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { Icon } from '@iconify/vue';
import ItemCard from './components/ItemCard.vue';
import ItemModal from './components/ItemModal.vue';
import SearchBox from './components/SearchBox.vue';
import Pagination from './components/Pagination.vue';
import itemsData from './data/itens.json';

// Reactive state variables
const items = ref([]);
const loading = ref(true);
const searchQuery = ref('');
const sortBy = ref('grade-asc');
const theme = ref(localStorage.getItem('theme') || 'light');
const currentPage = ref(1);
const selectedItem = ref(null);
const itemsPerPage = 30;

const { t, locale } = useI18n();

watch(theme, (newTheme) => {
  document.documentElement.setAttribute('data-theme', newTheme);
  localStorage.setItem('theme', newTheme);
}, { immediate: true });

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light';
};

const toggleLanguage = () => {
  locale.value = locale.value === 'pt' ? 'en' : 'pt';
  localStorage.setItem('locale', locale.value);
};

const setSearchQuery = (query) => {
  searchQuery.value = query;
};

const setPage = (page) => {
  currentPage.value = page;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

watch([searchQuery, sortBy], () => {
  currentPage.value = 1;
});

const filteredItems = computed(() => {
  let result = items.value;

  const query = searchQuery.value.trim().toLowerCase();

  if (query) {
    // Matches strings like "2010 - 2015" or "2010-2015"
    const rangeMatch = query.match(/^(\d{4})\s*-\s*(\d{4})$/);
    
    if (rangeMatch) {
      const startYear = parseInt(rangeMatch[1], 10);
      const endYear = parseInt(rangeMatch[2], 10);
      result = result.filter(item => item.year >= startYear && item.year <= endYear);
    } else {
      result = result.filter(item => 
        item.title.toLowerCase().includes(query) ||
        item.name.toLowerCase().includes(query) ||
        item.author.toLowerCase().includes(query) ||
        item.year.toString().includes(query)
      );
    }
  }

  return [...result].sort((a, b) => {
    if (sortBy.value === 'name-asc') return a.title.localeCompare(b.title);
    if (sortBy.value === 'name-desc') return b.title.localeCompare(a.title);
    if (sortBy.value === 'year-desc') return b.year - a.year;
    if (sortBy.value === 'year-asc') return a.year - b.year;
    if (sortBy.value === 'grade-asc') {
      const gradeA = a.grade || 'Z';
      const gradeB = b.grade || 'Z';
      const gradeDiff = gradeA.localeCompare(gradeB);
      if (gradeDiff !== 0) return gradeDiff;
      return a.title.localeCompare(b.title);
    }
    if (sortBy.value === 'grade-desc') {
      const gradeA = a.grade || 'Z';
      const gradeB = b.grade || 'Z';
      const gradeDiff = gradeB.localeCompare(gradeA);
      if (gradeDiff !== 0) return gradeDiff;
      return a.title.localeCompare(b.title);
    }
    return 0;
  });
});

const totalPages = computed(() => Math.ceil(filteredItems.value.length / itemsPerPage));

const paginatedItems = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredItems.value.slice(start, end);
});

onMounted(async () => {
  setTimeout(() => {
    items.value = itemsData.map(item => {
      const title = item.title || item.displayed_name;
      const name = item.name || 
        title.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").replace(/[^a-z0-9]+/g, '-').replace(/(^-|-$)/g, '') + '-' + item.id;
        
      const { thumbnail, displayed_name, ...rest } = item;
      
      return {
        ...rest,
        title,
        name,
        thumbnail: `${import.meta.env.BASE_URL}thumbnails/${name}.png`
      };
    });
    loading.value = false;
  }, 800);
});
</script>

<style>
:root {
  --bg-color: #f5f5f5;
  --text-main: #1a1a1a;
  --text-muted: #666666;
  --card-bg: #ffffff;
  --border-color: #cccccc;
  --input-bg: #ffffff;
  --accent-color: #42b883;
  --accent-hover: #33a06f;
}

:root[data-theme="dark"] {
  --bg-color: #121212;
  --text-main: #ffffff;
  --text-muted: #b3b3b3;
  --card-bg: #2a2a2a;
  --border-color: #444444;
  --input-bg: #2a2a2a;
}

body {
  margin: 0;
  background-color: var(--bg-color);
  color: var(--text-main);
  transition: background-color 0.3s ease, color 0.3s ease;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.skeleton {
  background: linear-gradient(90deg, rgba(130, 130, 130, 0.1) 25%, rgba(130, 130, 130, 0.2) 50%, rgba(130, 130, 130, 0.1) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
</style>

<style scoped>
.app-container {
  padding: 2rem;
  font-family: Arial, sans-serif;
}

.controls {
  display: flex;
  gap: 1rem;
}

.control-btn {
  background-color: var(--card-bg);
  color: var(--text-main);
  border: 1px solid var(--border-color);
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.control-btn:hover {
  border-color: var(--accent-color);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  gap: 1rem;
  flex-wrap: wrap;
}

.header-left {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.header-left h1 {
  font-size: 1.8rem;
  margin: 0;
  color: var(--accent-color);
  white-space: nowrap;
}

.loading-state {
  text-align: center;
  font-size: 1.5rem;
  color: var(--text-muted);
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 2rem;
}

.item-placeholder {
  visibility: hidden;
  pointer-events: none;
  display: flex;
  flex-direction: column;
}

.placeholder-thumb {
  width: 100%;
  aspect-ratio: 1 / 1;
}

.placeholder-info {
  padding: 1rem;
  min-height: 140px; /* Approximates the height of the item info block */
}

.search-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
  flex-grow: 1;
}

.sort-select {
  padding: 0.75rem 1rem;
  border-radius: 8px;
  border: 1px solid var(--border-color);
  background-color: var(--input-bg);
  color: var(--text-main);
  font-size: 1rem;
  cursor: pointer;
}

.sort-select:focus {
  outline: none;
  border-color: var(--accent-color);
}

.results-summary {
  text-align: center;
  margin-bottom: 1.5rem;
}

.item-count {
  font-size: 1rem;
  color: var(--text-muted);
}

.top-pagination {
  margin-top: 0;
  margin-bottom: 2rem;
}

.footer {
  margin-top: 4rem;
  text-align: center;
  color: var(--text-muted);
  padding-top: 2rem;
  border-top: 1px solid var(--border-color);
}
</style>

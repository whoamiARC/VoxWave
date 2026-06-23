<template>
  <div class="language-switcher" ref="switcherRef">
    <button class="switcher-trigger" @click="toggleDropdown" aria-label="切换语言">
      <span class="trigger-dot" aria-hidden="true"></span>
      <span class="trigger-label">{{ currentLabel }}</span>
      <span class="caret">{{ open ? '▲' : '▼' }}</span>
    </button>
    <ul v-if="open" class="switcher-dropdown">
      <li
        v-for="loc in availableLocales"
        :key="loc.key"
        class="switcher-option"
        :class="{ active: loc.key === locale }"
        @click="switchLocale(loc.key)"
      >
        {{ loc.label }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { availableLocales } from '@/i18n/index.js'

const { locale } = useI18n()
const open = ref(false)
const switcherRef = ref(null)

const currentLabel = computed(() => {
  const found = availableLocales.find(l => l.key === locale.value)
  return found ? found.label : locale.value
})

const toggleDropdown = () => {
  open.value = !open.value
}

const switchLocale = (key) => {
  locale.value = key
  localStorage.setItem('locale', key)
  document.documentElement.lang = key
  open.value = false
}

const onClickOutside = (e) => {
  if (switcherRef.value && !switcherRef.value.contains(e.target)) {
    open.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', onClickOutside)
  document.documentElement.lang = locale.value
})

onUnmounted(() => {
  document.removeEventListener('click', onClickOutside)
})
</script>

<style scoped>
.language-switcher {
  position: relative;
  display: inline-block;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
}

.switcher-trigger {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: rgba(15, 23, 42, 0.7);
  color: #e2e8f0;
  border: 1px solid #1e293b;
  padding: 5px 12px;
  border-radius: 8px;
  font-size: 11px;
  letter-spacing: 0.12em;
  font-weight: 600;
  cursor: pointer;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.switcher-trigger:hover {
  border-color: rgba(34, 211, 238, 0.5);
  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.08);
}

.trigger-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #22d3ee;
  box-shadow: 0 0 6px rgba(34, 211, 238, 0.7);
}

.caret {
  font-size: 0.6rem;
  color: #64748b;
}

.switcher-dropdown {
  position: absolute;
  top: calc(100% + 6px);
  right: 0;
  background: #0f172a;
  border: 1px solid #1e293b;
  border-radius: 8px;
  list-style: none;
  padding: 6px 0;
  margin: 0;
  min-width: 100%;
  z-index: 1000;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
}

.switcher-option {
  padding: 7px 14px;
  font-size: 11px;
  letter-spacing: 0.08em;
  color: #94a3b8;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.15s ease, color 0.15s ease;
}

.switcher-option:hover {
  background: rgba(34, 211, 238, 0.08);
  color: #e2e8f0;
}

.switcher-option.active {
  color: #22d3ee;
}
</style>

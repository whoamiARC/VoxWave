<template>
  <header class="app-header">
    <div class="header-left">
      <slot name="brand">
        <AppBrand />
      </slot>
      <div v-if="$slots.context" class="header-context">
        <slot name="context" />
      </div>
    </div>

    <div class="header-center">
      <slot name="center">
        <div v-if="layoutModes && layoutModes.length" class="view-switcher">
          <button
            v-for="mode in layoutModes"
            :key="mode.key"
            class="switch-btn"
            :class="{ active: mode.key === activeMode }"
            @click="$emit('change-mode', mode.key)"
          >
            <span class="switch-dot" aria-hidden="true"></span>
            {{ mode.label }}
          </button>
        </div>
      </slot>
    </div>

    <div class="header-right">
      <slot name="right">
        <span class="status-indicator" :class="statusClass">
          <span class="dot"></span>
          <span class="status-text">{{ statusText }}</span>
        </span>
        <span class="header-divider"></span>
        <div v-if="stepLabel" class="workflow-step">
          <span class="step-num">STEP {{ stepIndex }}/5</span>
          <span class="step-name">{{ stepLabel }}</span>
        </div>
        <span v-if="stepLabel" class="header-divider"></span>
        <LanguageSwitcher />
      </slot>
    </div>
  </header>
</template>

<script setup>
import { computed } from 'vue'
import AppBrand from './AppBrand.vue'
import LanguageSwitcher from './LanguageSwitcher.vue'

const props = defineProps({
  // 视图模式 (graph | split | workbench)
  layoutModes: { type: Array, default: () => [] },
  activeMode: { type: String, default: '' },
  // 步骤指示
  stepIndex: { type: Number, default: 0 },
  stepLabel: { type: String, default: '' },
  // 状态指示
  status: { type: String, default: 'processing' } // processing | ready | completed | error | live
})

defineEmits(['change-mode'])

const statusClass = computed(() => props.status)
const statusText = computed(() => {
  switch (props.status) {
    case 'ready': return 'READY'
    case 'completed': return 'COMPLETED'
    case 'error': return 'ERROR'
    case 'live': return 'LIVE'
    case 'running': return 'RUNNING'
    case 'generating': return 'GENERATING'
    case 'building': return 'BUILDING'
    case 'analyzing': return 'ANALYZING'
    case 'preparing': return 'PREPARING'
    case 'initializing': return 'INITIALIZING'
    default: return 'PROCESSING'
  }
})
</script>

<style scoped>
.app-header {
  height: 64px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 22px;
  background:
    linear-gradient(180deg, rgba(15, 23, 42, 0.96) 0%, rgba(11, 18, 32, 0.96) 100%);
  border-bottom: 1px solid #1e293b;
  position: relative;
  z-index: 100;
  backdrop-filter: blur(8px);
}

.app-header::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: -1px;
  height: 1px;
  background: linear-gradient(90deg, transparent 0%, #22d3ee 30%, #a78bfa 70%, transparent 100%);
  opacity: 0.45;
}

.header-left,
.header-right {
  display: flex;
  align-items: center;
  gap: 16px;
  z-index: 2;
}

.header-center {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
}

.header-context {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-left: 16px;
  border-left: 1px solid #1e293b;
  margin-left: 4px;
}

.header-divider {
  width: 1px;
  height: 16px;
  background: #1e293b;
}

.view-switcher {
  display: inline-flex;
  background: rgba(15, 23, 42, 0.7);
  padding: 4px;
  border-radius: 8px;
  gap: 2px;
  border: 1px solid #1e293b;
}

.switch-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  border: 0;
  background: transparent;
  padding: 6px 14px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 600;
  color: #64748b;
  letter-spacing: 0.08em;
  border-radius: 6px;
  cursor: pointer;
  transition: color 0.18s ease, background 0.18s ease, box-shadow 0.18s ease;
}

.switch-btn:hover {
  color: #e2e8f0;
}

.switch-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #1e293b;
  transition: background 0.18s ease, box-shadow 0.18s ease;
}

.switch-btn.active {
  background: rgba(34, 211, 238, 0.08);
  color: #22d3ee;
  box-shadow: inset 0 0 0 1px rgba(34, 211, 238, 0.3);
}

.switch-btn.active .switch-dot {
  background: #22d3ee;
  box-shadow: 0 0 8px rgba(34, 211, 238, 0.7);
}

.status-indicator {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 4px 10px;
  border-radius: 999px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  letter-spacing: 0.12em;
  background: rgba(15, 23, 42, 0.7);
  border: 1px solid #1e293b;
}

.status-text {
  color: #94a3b8;
  font-weight: 700;
}

.dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #475569;
}

.status-indicator.processing .dot,
.status-indicator.running .dot,
.status-indicator.generating .dot,
.status-indicator.building .dot,
.status-indicator.analyzing .dot,
.status-indicator.preparing .dot,
.status-indicator.initializing .dot {
  background: #22d3ee;
  box-shadow: 0 0 10px rgba(34, 211, 238, 0.8);
  animation: pulse 1.4s ease-in-out infinite;
}

.status-indicator.ready .dot,
.status-indicator.completed .dot,
.status-indicator.live .dot {
  background: #34d399;
  box-shadow: 0 0 8px rgba(52, 211, 153, 0.7);
}

.status-indicator.error .dot {
  background: #f87171;
  box-shadow: 0 0 8px rgba(248, 113, 113, 0.7);
}

.status-indicator.processing .status-text,
.status-indicator.running .status-text,
.status-indicator.generating .status-text,
.status-indicator.building .status-text,
.status-indicator.analyzing .status-text,
.status-indicator.preparing .status-text,
.status-indicator.initializing .status-text {
  color: #22d3ee;
}

.status-indicator.ready .status-text,
.status-indicator.completed .status-text,
.status-indicator.live .status-text {
  color: #34d399;
}

.status-indicator.error .status-text {
  color: #f87171;
}

.workflow-step {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
}

.step-num {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.16em;
  color: #64748b;
}

.step-name {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: #e2e8f0;
  text-transform: uppercase;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(0.85); }
}

@media (max-width: 880px) {
  .app-header { padding: 0 16px; height: auto; min-height: 64px; flex-wrap: wrap; gap: 10px; }
  .header-center { position: static; transform: none; order: 3; width: 100%; justify-content: flex-start; }
  .view-switcher { flex-wrap: wrap; }
}
</style>

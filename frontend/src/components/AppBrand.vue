<template>
  <button class="app-brand" type="button" @click="goHome" aria-label="返回 VoxWave 首页">
    <span class="app-brand-mark" aria-hidden="true">
      <span class="mark-glow"></span>
      <span class="mark-wave"></span>
    </span>
    <span class="app-brand-copy">
      <strong>舆澜</strong>
      <em>VOXWAVE · COCKPIT</em>
    </span>
  </button>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()
const goHome = () => router.push('/')
</script>

<style scoped>
.app-brand {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  border: 0;
  background: transparent;
  color: inherit;
  padding: 0;
  cursor: pointer;
  font: inherit;
}

.app-brand-mark {
  position: relative;
  width: 36px;
  height: 36px;
  border-radius: 10px;
  background: #0b1220;
  border: 1px solid #1e293b;
  overflow: hidden;
  box-shadow: inset 0 0 0 1px rgba(34, 211, 238, 0.08);
}

.mark-glow {
  position: absolute;
  inset: 6px;
  border-radius: 50%;
  background: radial-gradient(circle at 50% 50%, rgba(34, 211, 238, 0.55), rgba(34, 211, 238, 0) 65%);
}

.mark-wave {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(90deg, transparent 6px, #22d3ee 6px 9px, transparent 9px 18px,
                            #a78bfa 18px 21px, transparent 21px 30px, #22d3ee 30px 33px, transparent 33px) top 12px left 6px / 24px 3px no-repeat,
    radial-gradient(circle at 22px 24px, #a78bfa 0 2.4px, transparent 3px);
  filter: drop-shadow(0 0 4px rgba(34, 211, 238, 0.4));
}

.app-brand-copy {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.05;
  letter-spacing: 0;
}

.app-brand-copy strong {
  font-size: 16px;
  font-weight: 800;
  color: #e2e8f0;
  letter-spacing: 0.04em;
}

.app-brand-copy em {
  margin-top: 3px;
  font-style: normal;
  font-family: 'JetBrains Mono', 'Space Mono', ui-monospace, monospace;
  font-size: 10px;
  color: #64748b;
  letter-spacing: 0.18em;
}

.app-brand:hover .mark-glow {
  background: radial-gradient(circle at 50% 50%, rgba(167, 139, 250, 0.6), rgba(34, 211, 238, 0) 65%);
}
</style>

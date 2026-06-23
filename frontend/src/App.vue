<template>
  <router-view />
</template>

<script setup>
</script>

<style>
:root {
  color-scheme: dark;
  --vx-bg: #0b1220;
  --vx-bg-panel: #0f172a;
  --vx-bg-elevated: #131c30;
  --vx-border: #1e293b;
  --vx-border-subtle: #1a2236;
  --vx-text: #e2e8f0;
  --vx-text-muted: #94a3b8;
  --vx-text-faint: #64748b;
  --vx-accent-cyan: #22d3ee;
  --vx-accent-purple: #a78bfa;
  --vx-accent-magenta: #f472b6;
  --vx-success: #34d399;
  --vx-warning: #fbbf24;
  --vx-danger: #f87171;
}

* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
  min-width: 320px;
  background: var(--vx-bg);
  color: var(--vx-text);
  font-family: 'Inter', 'Noto Sans SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  min-height: 100vh;
  background:
    radial-gradient(1200px 600px at 12% -10%, rgba(34, 211, 238, 0.08), transparent 60%),
    radial-gradient(1100px 500px at 95% 110%, rgba(167, 139, 250, 0.08), transparent 60%),
    var(--vx-bg);
}

#app {
  min-height: 100vh;
  font-family: 'Inter', 'Noto Sans SC', system-ui, sans-serif;
  color: var(--vx-text);
  background: transparent;
}

button,
input,
textarea,
select {
  font: inherit;
  color: inherit;
}

button {
  letter-spacing: 0;
}

::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

::-webkit-scrollbar-track {
  background: #0b1220;
}

::-webkit-scrollbar-thumb {
  background: #1e293b;
  border-radius: 999px;
  border: 2px solid #0b1220;
}

::-webkit-scrollbar-thumb:hover {
  background: #334155;
}

/* 公用网格点背景，可给需要 cockpit 质感的容器加 .vx-grid */
.vx-grid {
  background-image:
    linear-gradient(rgba(148, 163, 184, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(148, 163, 184, 0.05) 1px, transparent 1px);
  background-size: 32px 32px;
  background-position: -1px -1px;
}

/* 公用面板卡片样式：可作为 Step 内部容器的兜底 */
.vx-card {
  background: var(--vx-bg-panel);
  border: 1px solid var(--vx-border);
  border-radius: 12px;
}
</style>

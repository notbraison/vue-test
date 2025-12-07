<template>
  <button
    class="theme-toggle"
    :aria-pressed="isDark.toString()"
    @click="toggle"
    title="Toggle dark / light"
  >
    <span v-if="isDark">🌙</span>
    <span v-else>☀️</span>
  </button>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const STORAGE_KEY = 'theme';
const isDark = ref(false);

function applyClass(dark: boolean) {
  const el = document.documentElement;
  if (dark) el.classList.add('dark');
  else el.classList.remove('dark');
}

function toggle() {
  isDark.value = !isDark.value;
  applyClass(isDark.value);
  try { localStorage.setItem(STORAGE_KEY, isDark.value ? 'dark' : 'light'); } catch {}
}

onMounted(() => {
  // prefer stored setting, else fall back to prefers-color-scheme
  try {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored === 'dark' || stored === 'light') {
      isDark.value = stored === 'dark';
      applyClass(isDark.value);
      return;
    }
  } catch {}

  // fallback to system preference
  const prefersDark = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
  isDark.value = prefersDark;
  applyClass(isDark.value);
});
</script>

<style scoped>
.theme-toggle {
  background: transparent;
  border: 0;
  font-size: 18px;
  cursor: pointer;
  padding: 6px 8px;
  border-radius: 6px;
}
.theme-toggle:hover { background: rgba(0,0,0,0.04); }
</style>

<template>
  <div class="video-card">
    <div class="video-wrapper" ref="wrapper">
      <video
        ref="video"
        :src="videoSrc"
        class="player"
        playsinline
        @timeupdate="onTimeUpdate"
        @loadedmetadata="onLoadedMetadata"
        @ended="onEnded"
        @dblclick.prevent="toggleFullscreen"
      ></video>

      <div class="controls" :class="controlsClass">
        <button class="btn play" @click="togglePlay">
          <span v-if="!playing">►</span>
          <span v-else>❚❚</span>
        </button>

        <div class="progress">
          <input
            type="range"
            min="0"
            :max="(duration as any)"
            step="0.1"
            v-model.number="currentTime"
            @input="seek"
          />
        </div>

        <div class="time">{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</div>

        <input class="volume" type="range" min="0" max="1" step="0.01" v-model.number="volume" @input="onVolume" />

        <select v-model="playbackRate" @change="onRateChange" class="rate">
          <option v-for="r in rates" :key="r" :value="r">{{ r }}x</option>
        </select>

        <button class="btn pip" @click="togglePip">PIP</button>

        <button class="btn fs" @click="toggleFullscreen">⤢</button>
      </div>
    </div>

    <div class="meta">
      <h3 class="title">This is just a Test</h3>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';


// Use Vite's import meta URL to resolve the asset path reliably
const videoSrc = new URL('../assets/video/This is _Just a test_.mp4', import.meta.url).href;

const video = ref<HTMLVideoElement | null>(null);
const wrapper = ref<HTMLElement | null>(null);
const playing = ref(false);
const duration = ref(0);
const currentTime = ref(0);
const volume = ref(1);
const playbackRate = ref(1);
const rates = [0.5, 0.75, 1, 1.25, 1.5, 2];
const hideControls = ref(false);

const controlsClass = computed(() => ({ hidden: hideControls.value }));
// helper: sometimes template type-check wants .value explicitly

function togglePlay() {
  if (!video.value) return;
  if (video.value.paused) {
    video.value.play();
    playing.value = true;
  } else {
    video.value.pause();
    playing.value = false;
  }
}

function onLoadedMetadata() {
  if (!video.value) return;
  duration.value = Math.floor(video.value.duration);
}

function onTimeUpdate() {
  if (!video.value) return;
  currentTime.value = video.value.currentTime;
}

function seek() {
  if (!video.value) return;
  video.value.currentTime = currentTime.value;
}

function onVolume() {
  if (!video.value) return;
  video.value.volume = volume.value;
}

function onRateChange() {
  if (!video.value) return;
  video.value.playbackRate = playbackRate.value;
}

// Accept either a number or a ref-like object from the template
function formatTime(s: number | { value: number }) {
  const n = typeof s === 'object' && s && 'value' in s ? (s as unknown as { value: number }).value : (s as number);
  const sec = Number(n) || 0;
  if (!isFinite(sec)) return '0:00';
  const minutes = Math.floor(sec / 60);
  const seconds = Math.floor(sec % 60)
    .toString()
    .padStart(2, '0');
  return `${minutes}:${seconds}`;
}

async function togglePip() {
  if (!video.value) return;
  try {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const doc: any = document;
    if (doc.pictureInPictureElement) {
      await doc.exitPictureInPicture();
    } else {
      const vid = video.value as unknown as { requestPictureInPicture?: () => Promise<void> };
      if (vid?.requestPictureInPicture) {
        await vid.requestPictureInPicture();
      }
    }
  } catch {
    // ignore PIP errors silently
  }
}

function toggleFullscreen() {
  if (!wrapper.value) return;
  const el = wrapper.value as HTMLElement;
  if (!document.fullscreenElement) {
    el.requestFullscreen?.();
  } else {
    document.exitFullscreen?.();
  }
}

function onEnded() {
  playing.value = false;
}

function onKey(e: KeyboardEvent) {
  if (!video.value) return;
  if (e.code === 'Space') {
    e.preventDefault();
    togglePlay();
  } else if (e.code === 'ArrowRight') {
    video.value.currentTime = Math.min(video.value.currentTime + 5, duration.value);
  } else if (e.code === 'ArrowLeft') {
    video.value.currentTime = Math.max(video.value.currentTime - 5, 0);
  } else if (e.code === 'KeyM') {
    video.value.muted = !video.value.muted;
    volume.value = video.value.muted ? 0 : video.value.volume;
  }
}

// (no extra computed refs needed — template unwraps refs)

onMounted(() => {
  window.addEventListener('keydown', onKey);
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey);
});
</script>

<script lang="ts">
// provide a classic default export for tooling/type-checkers that expect it
export default {
  name: 'VideoCard',
};
</script>

<style scoped>


.video-card {
  max-width: 960px;
  margin: 1.5rem auto;
  background: var(--color-background-soft, #fff);
  border: 1px solid var(--color-border, #eee);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 6px 18px rgba(0,0,0,0.06);
}



.video-wrapper {
  position: relative;
  background: #000;
}

.player {
  width: 100%;
  height: auto;
  display: block;
  background: black;
}

.controls {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 8px;
  display: flex;
  gap: 8px;
  align-items: center;
  padding: 8px;
  background: linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.5) 100%);
  z-index: 30;
  pointer-events: auto;
}

.controls.hidden { display: none; }
.btn { background: rgba(255,255,255,0.06); border: 0; color: #fff; padding: 6px 8px; border-radius: 4px; cursor: pointer; }
.progress { flex: 1; }
.progress input[type=range] { width: 100%; }
.time { color: #fff; font-size: 13px; min-width: 80px; text-align: center; }
.volume { width: 90px; }
.rate {
  background: rgba(255,255,255,0.06);
  color: var(--color-text);
  border: 0;
  padding: 4px 6px;
  border-radius: 4px;
  -webkit-appearance: none;
  appearance: none;
}

/* Make option text visible across browsers */
.rate option { color: var(--color-text); background: var(--color-background); }

.meta { padding: 12px 16px; color: var(--color-text); }
.title { margin: 0 0 4px 0; font-size: 18px; color: var(--color-heading); }
.subtitle { margin: 0; color: var(--color-text); font-size: 13px; }
</style>

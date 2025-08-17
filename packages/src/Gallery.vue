<template>
  <div class="gallery-container" @touchstart="onTouchStart" @touchend="onTouchEnd">
    <div class="gallery-box">

      <div class="top-bar">
        <button @click="prev" :disabled="!hasPrev" aria-label="Previous" title="Previous">
          <!-- SVG left arrow -->
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20">
            <path fill="currentColor" d="m5.83 9l5.58-5.58L10 2l-8 8l8 8l1.41-1.41L5.83 11H18V9z" />
          </svg>
        </button>
        <span class="counter">{{ currentIndex + 1 }} / {{ props.images.length }}</span>
        <button @click="next" :disabled="!hasNext" aria-label="Next" title="Next">
          <!-- SVG right arrow -->
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20">
            <path fill="currentColor" d="M2 11h12.2l-5.6 5.6L10 18l8-8l-8-8l-1.4 1.4L14.2 9H2z" />
          </svg>
        </button>
      </div>

      <div class="thumbnails">
        <button
          v-for="(img, idx) in surroundingImages"
          :key="idx"
          type="button"
          class="thumb"
          :class="{ active: currentIndex === (currentIndex - offset + idx) }"
          @pointerdown.stop
          @click.stop="goTo(currentIndex - offset + idx)"
          :aria-pressed="currentIndex === (currentIndex - offset + idx)"
          :title="props.captions?.[currentIndex - offset + idx] || `Image ${currentIndex - offset + idx + 1}`"
        >
          <img :src="img" :alt="props.captions?.[currentIndex - offset + idx] || ''" draggable="false" @dragstart.prevent />
        </button>
      </div>

      <div class="all-images-hidden" aria-hidden="true">
        <img
          v-for="(src, i) in props.images"
          :key="'hidden-' + i"
          :src="src"
          :alt="props.captions?.[i] || ''"
          loading="eager"
          draggable="false"
        />
      </div>

      <div class="caption" v-if="props.captions?.[currentIndex]">
        {{ props.captions[currentIndex] }}
      </div>

      <div class="image-wrapper">
        <transition :name="direction" mode="out-in">
          <img
            :src="props.images[currentIndex]"
            :alt="altText"
            :key="props.images[currentIndex]"
          />
        </transition>
      </div>

    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, onMounted, onUnmounted  } from 'vue'

interface Props {
  images: string[]
  captions?: string[]
}

const props = defineProps<Props>()
const currentIndex = ref(0)
const direction = ref('fade')
const isMobile = ref(false)
let touchStartX = 0

function checkMobile() {
  isMobile.value = window.innerWidth <= 960
}

function onTouchStart(e: TouchEvent) {
  touchStartX = e.changedTouches[0].clientX
}
onMounted(() => {
  checkMobile()
  window.addEventListener('resize', checkMobile)
})
onUnmounted(() => {
  window.removeEventListener('resize', checkMobile)
})
function onTouchEnd(e: TouchEvent) {
  const touchEndX = e.changedTouches[0].clientX
  const diff = touchEndX - touchStartX
  const threshold = 40
  if (diff > threshold) prev()
  else if (diff < -threshold) next()
}

const hasPrev = computed(() => currentIndex.value > 0)
const hasNext = computed(() => currentIndex.value < props.images.length - 1)

function prev() {
  if (hasPrev.value) {
    direction.value = 'slide-right'
    currentIndex.value--
  }
}

function next() {
  if (hasNext.value) {
    direction.value = 'slide-left'
    currentIndex.value++
  }
}

function goTo(absIndex: number) {
  if (absIndex === currentIndex.value) return
  direction.value = absIndex > currentIndex.value ? 'slide-left' : 'slide-right'
  currentIndex.value = absIndex
}


const surroundingImages = computed(() => {
  const result: string[] = []

  const backCount = isMobile.value ? 1 : 2
  const forwardCount = isMobile.value ? 2 : 2

  for (let i = -backCount; i <= forwardCount; i++) {
    const idx = currentIndex.value + i
    if (idx >= 0 && idx < props.images.length) result.push(props.images[idx])
  }

  return result
})


const offset = computed(() => {
  const backCount = isMobile.value ? 1 : 2
  return Math.min(backCount, currentIndex.value)
})


function escapeHTML(str: string): string {
  return str
    .replace(/&/g, '&amp;')
    .replace(/"/g, '&quot;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
}

const altText = computed(() => {
  const caption = props.captions?.[currentIndex.value]
  return caption ? escapeHTML(caption) : `Image ${currentIndex.value + 1}`
})
</script>

<style scoped>
.gallery-container {
  display: flex;
  justify-content: center;
  padding: 2rem 0 1rem 0;
}
.gallery-box {
  width: 100%;
  max-width: 1000px;
  background-color: var(--vp-code-block-bg);
  border-radius: 1rem;
  padding: 1rem;
  transition: background-color 0.25s, border-color 0.25s;
  overflow: hidden;
}
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.8rem;
}
.top-bar button {
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  border: 1px solid var(--vp-c-border);
  padding: 0.4rem 1rem;
  font-size: 1.5rem;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: all 0.2s ease;
}
.top-bar button:hover:not(:disabled) {
  background-color: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-1);
}
.top-bar button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.counter {
  font-size: 0.95rem;
  color: var(--vp-c-text-2);
}
.thumbnails {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  overflow-x: auto;
  padding-bottom: 0.25rem;
}

.thumb {
  width: 55px;
  height: 35px;
  border-radius: 0.4rem;
  overflow: hidden;
  opacity: 0.5;
  filter: brightness(0.6);
  transition: transform 0.25s ease, opacity 0.25s ease, filter 0.25s ease, background-color 0.2s ease;
  flex: 0 0 auto;
  border: 0;
  background: transparent;
  padding: 0;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.thumb:focus {
  outline: 2px solid var(--vp-c-brand-soft);
  outline-offset: 2px;
}
.thumb.active {
  filter: none;
  opacity: 1;
  transform: scale(1.1);
}
.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  pointer-events: none;
  user-select: none;
}
.caption {
  overflow-x: auto;
  padding: 1rem 1rem 1rem;
  text-align: center;
  max-width: 100%;
  word-break: break-word;
  font-size: 0.95rem;
  line-height: 1.4;
  color: var(--vp-c-text-2);
}
.image-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: auto;
  transition: height 0.25s ease;
}
.image-wrapper img {
  max-width: 100%;
  height: auto;
  border-radius: 0.75rem;
  transition: all 0.3s ease;
  object-fit: contain;
}
@media (max-width: 600px) {
  .image-wrapper img {
    max-height: 80vh;
  }
  .image-wrapper {
    min-height: auto;
  }
}
.all-images-hidden {
  position: absolute;
  left: -9999px;
  top: -9999px;
  width: 1px;
  height: 1px;
  overflow: hidden;
  pointer-events: none;
  user-select: none;
  visibility: hidden;
}
.all-images-hidden img {
  width: 1px;
  height: 1px;
  opacity: 0;
  pointer-events: none;
}
.fade-enter-active, .fade-leave-active,
.slide-left-enter-active, .slide-left-leave-active,
.slide-right-enter-active, .slide-right-leave-active {
  transition: all 0.25s ease;
  position: relative;
}

.fade-enter-from, .fade-leave-to { opacity: 0; }

.slide-left-enter-from {
  transform: translateX(100%);
  opacity: 0;
}
.slide-left-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

.slide-right-enter-from {
  transform: translateX(-100%);
  opacity: 0;
}
.slide-right-leave-to {
  transform: translateX(100%);
  opacity: 0;
}


@media (hover: hover) and (pointer: fine) and (min-width: 960px) {
  .thumbnails {
    gap: 0.75rem; 
  }


  .thumb:hover:not(.active) {
    opacity: 0.95;
    filter: none;
    background-color: rgba(0,0,0,0.02);
  }


  .thumb:focus {
    outline: 2px solid var(--vp-c-brand-soft);
    outline-offset: 2px;
  }
}
</style>

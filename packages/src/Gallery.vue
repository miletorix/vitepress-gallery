<template>
  <div class="gallery-container" @touchstart="onTouchStart" @touchend="onTouchEnd">
    <div class="gallery-box">

      <div class="top-bar">
        <button @click="prev" :disabled="!hasPrev" aria-label="Previous">
          <!-- SVG left arrow -->
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20">
            <path fill="currentColor" d="m5.83 9l5.58-5.58L10 2l-8 8l8 8l1.41-1.41L5.83 11H18V9z" />
          </svg>
        </button>
        <span class="counter">{{ currentIndex + 1 }} / {{ images.length }}</span>
        <button @click="next" :disabled="!hasNext" aria-label="Next">
          <!-- SVG right arrow -->
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20">
            <path fill="currentColor" d="M2 11h12.2l-5.6 5.6L10 18l8-8l-8-8l-1.4 1.4L14.2 9H2z" />
          </svg>
        </button>
      </div>

      <div class="thumbnails">
        <div
          v-for="(img, index) in surroundingImages"
          :key="index"
          class="thumb"
          :class="{ active: index + offset === currentIndex }"
        >
          <img :src="img" :alt="'Thumbnail ' + (index + offset + 1)" />
        </div>
      </div>

      <div class="caption" v-if="captions?.[currentIndex]">
        {{ captions[currentIndex] }}
      </div>

      <div class="image-wrapper">
        <transition :name="direction" mode="out-in">
          <img
            :src="images[currentIndex]"
            :alt="altText"
            :key="images[currentIndex]"
          />
        </transition>
      </div>

    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'

interface Props {
  images: string[]
  captions?: string[]
}

const props = defineProps<Props>()
const currentIndex = ref(0)
const direction = ref('fade')
let touchStartX = 0

function onTouchStart(e: TouchEvent) {
  touchStartX = e.changedTouches[0].clientX
}

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

const offset = computed(() => Math.max(0, currentIndex.value - 2))

const surroundingImages = computed(() => {
  const result: string[] = []
  for (let i = -2; i <= 2; i++) {
    const index = currentIndex.value + i
    if (index >= 0 && index < props.images.length) {
      result.push(props.images[index])
    }
  }
  return result
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
  /* border: 2px solid transparent; */
  border-radius: 1rem;
  /* box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08); */
  padding: 1rem;
  transition: background-color 0.25s, border-color 0.25s;
  overflow: hidden;
}

/* .gallery-box:hover {
  border-color: var(--vp-c-brand-soft);
} */

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
}

.thumb {
  width: 55px;
  height: 35px;
  border-radius: 0.4rem;
  overflow: hidden;
  opacity: 0.5;
  filter: brightness(0.6);
  transition: all 0.25s ease;
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
}


.caption {
  overflow-x: auto;
  padding: 1rem 1rem 1rem;
  text-align: center;
  max-width: 100%;
  word-break: break-word;
  font-size: 0.95rem;
  /* font-style: italic; */
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


.fade-enter-active, .fade-leave-active,
.slide-left-enter-active, .slide-left-leave-active,
.slide-right-enter-active, .slide-right-leave-active {
  transition: all 0.25s ease;
  position: relative;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

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
</style>
# vitepress-gallery
A responsive, swipeable image gallery component for VitePress with thumbnails, navigation, captions, and smooth transitions.

![npm](https://img.shields.io/npm/v/@miletorix/vitepress-gallery) ![npm](https://img.shields.io/npm/dw/@miletorix/vitepress-gallery) ![license](https://img.shields.io/npm/l/@miletorix/vitepress-gallery)


<p align="center">
  <img src="/assets/demo-1.png" alt="demo" width="800">
</p>

📦 NPM Package:  
👉 [https://www.npmjs.com/package/@miletorix/vitepress-gallery](https://www.npmjs.com/package/@miletorix/vitepress-gallery)

## Installation

```sh
npm i @miletorix/vitepress-gallery
```

## Usage

### Configuration

```typescript
// docs/.vitepress/theme/index.ts
import type { Theme } from 'vitepress'
import DefaultTheme from 'vitepress/theme'
 
import { Gallery } from '@miletorix/vitepress-gallery' // [!code ++]
import '@miletorix/vitepress-gallery/style.css' // [!code ++]

export default {
  extends: DefaultTheme,
  enhanceApp(ctx) {
    ctx.app.component('Gallery', Gallery) // [!code ++]
  }
}
```

### Image Group

```vue
<Gallery 
  :images="[
    'demo-1.png',
    'demo-2.jpg',
    'demo-3.jpg',
    'demo-4.jpg'
  ]" 
  :captions="[
    'Image caption Nr.1 ...',
    'Image caption Nr.2 ...',
    'Image caption Nr.3 ...',
    'Image caption Nr.4 ...'
  ]"
/>
```
<!--
  From: https://github.com/francoischalifour/medium-zoom/blob/master/examples/vue/src/components/ImageZoom.vue
-->
<script setup lang="ts">
import mediumZoom, { type Zoom, type ZoomOptions } from 'medium-zoom'
import { watch, type ComponentPublicInstance } from 'vue'

interface Props {
  options?: ZoomOptions
}

const props = defineProps<Props>()

let zoom: Zoom | null = null

function getZoom() {
  if (zoom === null) {
    zoom = mediumZoom(props.options)
  }

  return zoom
}

function attachZoom(ref: Element | ComponentPublicInstance | null) {
  const image = ref as HTMLImageElement | null
  const zoom = getZoom()

  if (image) {
    zoom.attach(image)
  } else {
    zoom.detach()
  }
}

watch(
  () => props.options,
  (options) => {
    const zoom = getZoom()
    zoom.update(options || {})
  }
)
</script>

<template>
  <img :ref="attachZoom" />
</template>

<script setup lang="ts">
import ImageZoom from '@/components/ImageZoom.vue'
import { ref, watch } from 'vue'

const toGetImages = Object.keys(
  import.meta.glob('@/assets/photos/*.{png,jpg,jpeg,gif,svg}', { eager: true })
).map((imagePath) => getImage(imagePath))

async function getImage(src: string): Promise<HTMLImageElement> {
  return new Promise((resolve) => {
    const image = new Image()
    image.src = src
    image.addEventListener('load', () => {
      resolve(image)
    })
  })
}

const gallery = ref<HTMLDivElement | null>(null)
const board = ref<
  {
    height: number
    elements: HTMLImageElement[]
  }[]
>([])
const width = ref(0)

watch(
  width,
  async () => {
    const images = await Promise.all(toGetImages)
    const gridTemplateColumns = getComputedStyle(gallery.value!).gridTemplateColumns
    const columns = gridTemplateColumns.split(' ').length
    const columnWidth = parseFloat(gridTemplateColumns)
    board.value = Array.from({ length: columns }, () => ({
      height: 0,
      elements: Array<HTMLImageElement>()
    }))

    images.forEach((image) => {
      const indexOfMinHeightColumn = board.value
        .map((column) => column.height)
        .reduce((minIndex, current, i, array) => (current < array[minIndex] ? i : minIndex), 0)
      board.value[indexOfMinHeightColumn].elements.push(image)
      board.value[indexOfMinHeightColumn].height += (columnWidth / image.width) * image.height
    })

    addEventListener('resize', () => {
      width.value = window.innerWidth
    })
  },
  { immediate: true }
)
</script>

<template>
  <div class="gallery" ref="gallery">
    <div class="gallery-column" v-for="[i, column] in board.entries()" :key="`col-${i}`">
      <ImageZoom
        class="gallery-img"
        v-for="[j, image] in column.elements.entries()"
        :src="image.src"
        :key="`img-${j}`"
      />
    </div>
  </div>
</template>

<style scoped>
.gallery {
  display: grid;
  grid-column: 1/-1;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  grid-gap: 1.5rem;
  align-items: start;
}

.gallery-column {
  display: grid;
  grid-gap: 1.5rem;
}

.gallery-img {
  width: 100%;
  object-fit: cover;
  display: block;
  cursor: pointer;
}
</style>

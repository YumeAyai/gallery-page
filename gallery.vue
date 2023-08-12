<script setup>
import {
  ref,
  watch,
  onMounted,
  onBeforeUnmount,
  getCurrentInstance,
  toRef,
} from "vue";

const displayedPhotos = ref([]);

const windowWidth = ref(window.innerWidth);

const previewImageUrl = ref("");
const isPreviewVisible = ref(false);

const handleResize = () => {
  windowWidth.value =
    window.innerWidth ||
    document.documentElement.clientWidth ||
    document.body.clientWidth;
};

const showImagePreview = (event, photoPath) => {
  previewImageUrl.value = photoPath;
  isPreviewVisible.value = true;
};

const hideImagePreview = () => {
  isPreviewVisible.value = false;
};

const adjustContainerWidth = () => {
  const imagePreview = document.getElementById("imagePreview");
  const previewImage = document.getElementById("previewImage");

  if (imagePreview && previewImage) {
    if (previewImage.naturalHeight > imagePreview.clientHeight) {
      imagePreview.style.width = `${
        (imagePreview.clientHeight / previewImage.naturalHeight) *
        previewImage.naturalWidth
      }px`;
    } else {
      imagePreview.style.width = "auto";
    }
  }
};

const generateGallery = () => {
  var gallery = document.querySelector(".gallery");
  var gridTemplateColumns = getComputedStyle(gallery).gridTemplateColumns;
  var columnWidths = gridTemplateColumns.split(" ");
  var columnCount = columnWidths.length;
  const columnIds = new Map();
  for (let i = 0; i < columnCount; i++) {
    const col = document.createElement("div");
    col.setAttribute(gallery.attributes[0].name, "");
    col.className = "gallery__column";
    col.setAttribute("id", "col" + i);
    columnIds.set("col" + i, 0);
    gallery.appendChild(col);
  }

  const photosContext = import.meta.globEager(
    "@/assets/photos/*.{png,jpg,jpeg,gif,svg}"
  );
  displayedPhotos.value = Object.values(photosContext).map(
    (photo) => photo.default
  );
  var minValue = 0;
  var minKey = "col0";
  displayedPhotos.value.forEach((photoPath, index) => {
    const galleryItem = document.createElement("div");
    galleryItem.setAttribute(gallery.attributes[0].name, "");
    galleryItem.className = "gallery__item";

    const img = document.createElement("img");
    img.src = photoPath;
    img.alt = "Photo " + (index + 1);
    img.setAttribute(gallery.attributes[0].name, "");
    img.className = "gallery__img";
    img.onclick = function (event) {
      showImagePreview(event, photoPath);
    };

    galleryItem.appendChild(img);

    var columnWidth = parseFloat(columnWidths[0]);
    var imgheight = (columnWidth / img.width) * img.height;
    var gap = parseFloat(getComputedStyle(gallery).getPropertyValue("row-gap"));
    document.getElementById(minKey).appendChild(galleryItem);
    minValue += imgheight + gap;
    columnIds.set(minKey, minValue);

    for (var [key, value] of columnIds) {
      if (value < minValue) {
        minKey = key;
        minValue = value;
      }
    }
  });
};

onMounted(async () => {
  generateGallery();
  window.addEventListener("resize", handleResize);
  window.addEventListener("resize", adjustContainerWidth);
});
onBeforeUnmount(() => {
  window.removeEventListener("resize", handleResize);
  window.removeEventListener("resize", adjustContainerWidth);
});
watch(windowWidth, () => {
  const gallery = document.querySelector(".gallery");
  while (gallery.firstChild) {
    gallery.removeChild(gallery.firstChild);
  }
  generateGallery();
});
</script>

<template>
  <div
    v-if="isPreviewVisible"
    class="gallery__img__overlay"
    id="overlay"
    @click="hideImagePreview"
  ></div>
  <div v-if="isPreviewVisible" class="gallery__img__preview" id="imagePreview">
    <img :src="previewImageUrl" alt="Preview" id="previewImage" />
  </div>
  <div class="gallery" ref="gallery"></div>
</template>

<style scoped>
.gallery {
  display: grid;
  grid-column: 1/-1;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  grid-gap: 1.5rem;
  align-items: start;
}

.gallery__column {
  display: grid;
  grid-gap: 1.5rem;
}

.gallery__item {
  position: relative;
  overflow: hidden;
}

.gallery__img {
  width: 100%;
  object-fit: cover;
  display: block;
  cursor: pointer;
}

.gallery__img__preview {
  display: block;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border: 2px solid #fff;
  background-color: #fff;
  padding: 10px;
  overflow: auto;
  z-index: 3;
}
.gallery__img__preview img {
  max-width: 80vw;
  max-height: 80vh;
}
.gallery__img__overlay {
  display: block;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 2;
}
</style>

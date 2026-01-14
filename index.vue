<template>
  <div class="screen-content">
    <div ref="screenScaleBody" class="screen-scale-body">
      <slot></slot>
    </div>
  </div>
</template>
<script setup lang="ts">
import { debounce } from 'lodash-es';
import { ref, onMounted, nextTick, onUnmounted } from 'vue';
const props = defineProps({
  width: {
    type: Number,
    default: 1920
  },
  height: {
    type: Number,
    default: 1080
  }
});

let realWidth = ref(0);
let realHeight = ref(0);
const screenScaleBody = ref<any>();

/**
 * 初始化大屏容器宽高
 */
const initSize = () => {
  return new Promise(resolve => {
    nextTick(() => {
      realWidth.value = document.body.clientWidth;
      realHeight.value = document.body.clientHeight;
      resolve(undefined);
    });
  });
};

const autoScale = (scale:any) => {
  const currentWidth = document.body.clientWidth;
  const currentHeight = document.body.clientHeight;
  screenScaleBody.value.style.transform = `scale(${scale},${scale})`;
  let mx = Math.max((currentWidth - props.width * scale) / 2, 0);
  let my = Math.max((currentHeight - props.height * scale) / 2, 0);
  screenScaleBody.value.style.margin = `${my}px ${mx}px`;
};

const updateScale = () => {
  screenScaleBody.value.style.width = `${props.width}px`;
  screenScaleBody.value.style.height = `${props.height}px`;

  // 获取真实视口尺寸
  const currentWidth = realWidth.value;
  const currentHeight = realHeight.value;

  // // 计算缩放比例
  const widthScale = currentWidth / props.width;
  const heightScale = currentHeight / props.height;

  const scale = Math.min(widthScale, heightScale);
  screenScaleBody.value.style.transform = `scale(${widthScale},${heightScale})`;
  autoScale(scale);
};

const onResize = debounce(async () => {
  await initSize();
  updateScale();
}, 200);

onMounted(async () => {
  await initSize();
  updateScale();
  window.addEventListener('resize', onResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', onResize);
});
</script>
<style scoped>
* {
  margin: 0;
  padding: 0;
}
.screen-content {
  overflow: 'hidden';
  width: 100vw;
  height: 100vh;
  position: absolute;
  
}
.screen-scale-body {
  position: relative;
  overflow: hidden;
  transform-origin: left top;
  transition-property: all;
  transition-timing-function: cubic-bezier(0.075, 0.82, 0.165, 1);
  transition-duration: 500ms;
}
</style>

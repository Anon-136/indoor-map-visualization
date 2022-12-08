<script lang="ts" setup>
import { onMounted, reactive, ref } from "vue";
import panzoom, { PanZoom } from 'panzoom'

const position = reactive({ x: 0, y: 0, set: false })
const wrapperRef = ref<HTMLElement>()
const panzoomRef = ref<PanZoom>()

function onPin({ layerX, layerY }: { layerX: number, layerY: number }) {
  const instance = panzoomRef.value
  if (!instance) return
  const transform = instance.getTransform()
  const x = (layerX - transform.x) / transform.scale
  const y = (layerY - transform.y) / transform.scale
  console.log(x, y);
  position.x = x
  position.y = y
}

function onReset() {
  const instance = panzoomRef.value
  if (!instance) return
  instance.zoomAbs(0, 0, 1)
  instance.smoothMoveTo(0, 0)
}

onMounted(() => {
  const element = document.querySelector('#panzoom') as HTMLElement
  panzoomRef.value = panzoom(element, {
    onTouch() {
      return
    },
    onClick: (e: any) => {
      if (e.path[1] === wrapperRef.value) {
        if (!position.set) {
          position.set = true
        }
        onPin(e)
        return false
      }
    },
    onDoubleClick: () => {
      // double tap does not work
      console.log('double click');
      onReset()
    },
    maxZoom: 3,
    minZoom: 1,
    bounds: true,
    boundsPadding: 0.5,
    zoomDoubleClickSpeed: 1,
  })
})
</script>

<template>
  <div>
    <div id="panzoom" ref="wrapperRef" class="relative aspect-square">
      <img src="https://picsum.photos/1000">
      <svg class="absolute -translate-x-1/2 -translate-y-full " height="10" width="10"
        :style="{ top: position.y, left: position.x, display: position.set ? 'block' : 'none' }">
        <circle cx="5" cy="5" r="4" stroke="black" stroke-width="1" fill="red" />
      </svg>
    </div>
  </div>
</template>

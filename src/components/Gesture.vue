<script lang="ts" setup>
import { onMounted, reactive, ref, watchEffect } from 'vue';
import { Gesture, } from '@use-gesture/vanilla'
import { computed } from '@vue/reactivity';

const target = ref<HTMLDivElement>()
const img = ref<HTMLImageElement>()
const image = reactive({ x: 0, y: 0, s: 1, width: 0, height: 0 })
const click = reactive({ x: 0, y: 0, timeStamp: 0, tap: false })
const pin = computed(() => {
  return {
    x: (click.x + image.x) + (click.x - image.width / 2) * (image.s - 1),
    y: (click.y + image.y) + (click.y - image.height / 2) * (image.s - 1),
    display: click.tap ? 'block' : 'none'
  }
})

const DOUBLE_CLICK_THRESHOLD = 200
const SNAP_THRESHOLD = 25
let shouldCancleTap = false

onMounted(() => {
  const targetElement = target.value
  const imageElement = img.value
  if (!targetElement || !imageElement) return

  imageElement.onload = () => {
    new Gesture(targetElement, {
      onDrag({ active, offset: [ox, oy], tap, event: e, timeStamp }) {
        // first && console.log('start', tap ? 'tap' : active ? 'drag' : 'wtf');
        if (tap) {
          // double click
          const isDoubleTap = click.timeStamp > 0 && timeStamp - click.timeStamp < DOUBLE_CLICK_THRESHOLD
          click.timeStamp = timeStamp
          if (isDoubleTap) {
            console.log('double tap');
            shouldCancleTap = true
            if (image.s === 1 && image.x === 0 && image.y === 0) {
              image.s = 2
              console.log('zoom in');
            } else {
              image.s = 1
              image.x = 0
              image.y = 0
              console.log('zoom out');
            }
            return
          }
          console.log('a tap');
          setTimeout(() => {
            if (shouldCancleTap) {
              shouldCancleTap = false
              console.log('cancle single tap');
            } else {
              console.log('single tap');
              const event = e as PointerEvent
              if (event.target === imageElement) {
                console.log('click', event.offsetX, event.offsetY);
                click.x = event.offsetX
                click.y = event.offsetY
                if (!click.tap) click.tap = true
              }
            }
          }, DOUBLE_CLICK_THRESHOLD)
        } else if (active) {
          console.log('drag', ox, oy);
          image.x = ox
          image.y = oy
        }
      },
      // https://codesandbox.io/s/amazing-tree-sfuexl?file=/src/App.jsx:1205-1700
      onPinch: ({ origin: [ox, oy], first, movement: [ds], offset: [s], memo }) => {
        console.log('pinch', s);
        if (first) {
          const tx = image.x
          const ty = image.y
          const cx = image.width / 2
          const cy = image.height / 2
          const rx = ox - (tx + cx)
          const ry = oy - (ty + cy)
          memo = [rx, ry, tx, ty]
        }
        const [rx, ry, tx, ty] = memo
        image.x = rx - ds * rx + tx
        image.y = ry - ds * ry + ty
        image.s = s
        return memo
      },
      onWheel: ({ offset: [ox, oy], pinching }) => {
        if (!pinching) {
          console.log('wheel', ox, oy);
          image.x = -ox
          image.y = -oy
        }
      }
    },
      {
        drag: {
          filterTaps: true,
          from: () => [image.x, image.y]
        },
        pinch: {
          scaleBounds: { min: 1, max: 8 }
        },
        wheel: {
          from: () => [-image.x, -image.y]
        }
      })

    // resize image when window resize
    const setImageSize = () => {
      const oldWidth = image.width
      const oldHeight = image.height
      const width = imageElement.clientWidth
      const height = imageElement.clientHeight
      image.width = width
      image.height = height
      click.x = click.x * width / oldWidth
      click.y = click.y * height / oldHeight
      console.log('resize', width, height);
    }
    setImageSize()
    window.addEventListener('resize', setImageSize)
  }
})

watchEffect(() => {
  if (Math.abs(image.x) + Math.abs(image.y) < SNAP_THRESHOLD) {
    console.log('pan snap');
    image.x = 0
    image.y = 0
  }
})
</script>

<template>
  <div ref="target" class="w-full h-full flex flex-col justify-center items-center touch-none cursor-grab">
    <div class="relative">
      <svg class="absolute z-30" height="10" width="10"
        :style="{ transform: `translateX(${pin.x - 5}px) translateY(${pin.y - 5}px) scale(${image.s})`, display: pin.display }">
        <circle cx="5" cy="5" r="4" stroke="black" stroke-width="1" fill="red" />
      </svg>
      <img ref="img" src="https://picsum.photos/1000" draggable="false" class="touch-none select-none"
        :style="{ transform: `translateX(${image.x}px) translateY(${image.y}px) scale(${image.s})` }" />
    </div>
  </div>
</template>

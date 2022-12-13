<script lang="ts" setup>
import { onMounted, reactive, ref, watchEffect } from 'vue';
import { Gesture } from '@use-gesture/vanilla'
import { computed } from '@vue/reactivity';

const target = ref<HTMLImageElement>()
const image = reactive({ x: 0, y: 0, s: 1, width: 0, height: 0 })
const click = reactive({ x: 0, y: 0, timeStamp: 0, tap: false })
const dot = computed(() => {
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
  const element = target.value
  if (!element) return

  // resize image when window resize
  const setImageSize = () => {
    const oldWidth = image.width
    const oldHeight = image.height
    const width = element.clientWidth
    const height = element.clientHeight
    image.width = width
    image.height = height
    click.x = click.x * width / oldWidth
    click.y = click.y * height / oldHeight
    console.log('resize', width, height);
  }
  setImageSize()
  window.addEventListener('resize', setImageSize)

  const gesture = new Gesture(element, {
    onDrag({ active, offset: [ox, oy], tap, event: e, timeStamp }) {
      console.log(tap ? 'tap' : active ? 'drag' : 'wtf');
      if (tap) {
        // double click
        console.log(click.timeStamp, timeStamp, timeStamp - click.timeStamp);
        const isDoubleTap = click.timeStamp > 0 && timeStamp - click.timeStamp < DOUBLE_CLICK_THRESHOLD
        click.timeStamp = timeStamp
        if (isDoubleTap) {
          console.log('double tap');
          shouldCancleTap = true
          image.x = 0
          image.y = 0
          image.s = 1
          return
        }
        console.log('single tap');
        setTimeout(() => {
          console.log('cancleTap', shouldCancleTap);
          if (shouldCancleTap) {
            shouldCancleTap = false
            console.log('cancle');
          } else {
            const event = e as PointerEvent
            console.log('click', event.offsetX, event.offsetY);
            click.x = event.offsetX
            click.y = event.offsetY
            if (!click.tap) click.tap = true
          }
        }, DOUBLE_CLICK_THRESHOLD)
      } else if (active) {
        console.log(active, ox, oy);
        image.x = ox
        image.y = oy
      }
    },
    onPinch({ offset: [s], active }) {
      console.log(s);

      if (active) {
        image.s = s
      }
    },
  },
    {
      drag: {
        bounds: {
          left: -image.width / 2,
          right: image.width / 2,
          top: -image.height / 2,
          bottom: image.height / 2
        },
        rubberband: true,
        filterTaps: true,
      },
      pinch: {
        scaleBounds: { min: 1, max: 8 }
      }
    })
  return () => {
    gesture.destroy()
  }
})

watchEffect(() => {
  if (Math.abs(image.x) + Math.abs(image.y) < SNAP_THRESHOLD) {
    console.log('pan snap');
    image.x = 0
    image.y = 0
  }
})

watchEffect(() => {
  if (image.s === 1) {
    console.log('zoom snap');
    image.x = 0
    image.y = 0
  }
}) 
</script>

<template>
  <div class="relative">
    <svg class="absolute -translate-x-1/2 -translate-y-full z-30" height="10" width="10"
      :style="{ transform: `translateX(${dot.x}px) translateY(${dot.y}px)`, display: dot.display }">
      <circle cx="5" cy="5" r="4" stroke="black" stroke-width="1" fill="red" />
    </svg>
    <img ref="target" src="https://picsum.photos/1000" draggable="false" class="touch-none select-none cursor-grab"
      :style="{ transform: `translateX(${image.x}px) translateY(${image.y}px) scale(${image.s})` }" />
  </div>
</template>

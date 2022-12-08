<script lang="ts" setup>
import { ref } from "vue";
import Panzoom from "./components/Panzoom.vue";
import FullScreenModal from "./components/FullScreenModal.vue";
import Gesture from "./components/Gesture.vue";
const msg = ref("Hello World!");
</script>

<template>
  <h1 class="text-3xl font-bold underline">{{ msg }}</h1>
  <div class="flex flex-col gap-2">
    <FullScreenModal text="Gesture" default-open>
      <Gesture />
    </FullScreenModal>
    <FullScreenModal text="Panzoom">
      <Panzoom />
    </FullScreenModal>
  </div>
</template>
<style>

</style>

<script lang="ts" setup>
import { ref } from "vue";
import Panzoom from "./components/Panzoom.vue";
import FullScreenModal from "./components/FullScreenModal.vue";

const msg = ref("Hello World!");
</script>

<template>
  <h1 class="text-3xl font-bold underline">{{ msg }}</h1>
  <FullScreenModal>
    <Panzoom />
  </FullScreenModal>
</template>
<style>

</style>

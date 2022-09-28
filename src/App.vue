<script lang="ts" setup>
import { ref } from "vue";
import Modal from "./components/Modal.vue";

const msg = ref("Hello World!");
</script>

<template>
  <h1 class="text-3xl font-bold underline">{{ msg }}</h1>
  <Modal />
</template>
<style>

</style>

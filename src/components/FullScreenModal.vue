<script lang="ts" setup>
import { ref } from "vue";
import {
  TransitionRoot,
  TransitionChild,
  Dialog,
  DialogPanel,
} from "@headlessui/vue";

const props = defineProps<{
  defaultOpen?: boolean
  text: string
}>()
const isOpen = ref(props.defaultOpen ?? false);


function closeModal() {
  isOpen.value = false;
}

function openModal() {
  isOpen.value = true;
}
</script>

<template>
  <div>
    <button type="button" @click="openModal"
      class="rounded-md bg-black bg-opacity-20 px-4 py-2 text-sm font-medium text-white hover:bg-opacity-30 focus:outline-none focus-visible:ring-2 focus-visible:ring-white focus-visible:ring-opacity-75">
      {{ props.text }}
    </button>
  </div>
  <TransitionRoot appear :show="isOpen" as="template">
    <Dialog as="div" @close="closeModal" class="relative z-10">
      <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0" enter-to="opacity-100"
        leave="duration-200 ease-in" leave-from="opacity-100" leave-to="opacity-0">
        <div class="fixed inset-0 bg-gray-800" />
      </TransitionChild>
      <div class="fixed inset-0">
        <TransitionChild as="template" enter="duration-300 ease-out" enter-from="opacity-0 scale-95"
          enter-to="opacity-100 scale-100" leave="duration-200 ease-in" leave-from="opacity-100 scale-100"
          leave-to="opacity-0 scale-95">
          <DialogPanel class="w-full h-full relative transform overflow-hidden transition-all">
            <div class="w-full h-full flex flex-col justify-center items-center ">
              <slot />
            </div>
            <button type="button"
              class="absolute bottom-0 right-0 m-4 z-20 inline-flex justify-center rounded-md border border-transparent bg-blue-100 px-4 py-2 text-sm font-medium text-blue-900 hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
              @click="closeModal">
              Done
            </button>
          </DialogPanel>
        </TransitionChild>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

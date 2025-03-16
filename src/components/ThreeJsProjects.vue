<script setup lang="ts">
import { ref } from "vue";
import TextAnimated from "./TextAnimated.vue";

const selectedComponent = ref(null);
const components = ref([{ component: TextAnimated, label: "Text Animated" }]);
</script>

<template>
  <div class="w-full h-full p-4">
    <div class="h-full flex flex-col">
      <select
        v-model="selectedComponent"
        class="w-full rounded-lg p-2 border focus:outline-none focus:ring-2"
      >
        <option
          v-for="(component, index) in components"
          :key="index"
          :value="component.component"
        >
          {{ component.label }}
        </option>
      </select>

      <div class="mt-4 rounded-lg p-4 flex-1">
        <component :is="selectedComponent" v-if="selectedComponent" class="h-full w-full" />
        <div v-else class="text-center h-full w-full flex items-center justify-center">
          Select a component to display
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>

<template>
  <div class="w-full h-[200px] overflow-hidden rounded-lg bg-gray-100">
    <iframe
      :src="url"
      :title="title"
      class="w-[200%] h-[200%] border-0 origin-top-left scale-50 pointer-events-none"
      loading="lazy"
    ></iframe>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  url: string;
  title: string;
}>();
</script>

<style>
/* No needed styles as we're using Tailwind */
</style>

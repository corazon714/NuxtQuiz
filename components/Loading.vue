<script lang="ts" setup>
const props = defineProps<{
  text: string
}>()
</script>

<template>
  <div class="flex flex-col text-9xl items-center text-center justify-center mt-[40vh] gap-y-5">
    <h3 class="text-6xl">
      {{ props.text }}
    </h3>
    <UIcon name="i-heroicons-arrow-path-solid" class="animate-spin" />
  </div>
</template>

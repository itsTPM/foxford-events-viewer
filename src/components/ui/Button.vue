<script setup>
defineProps({
  as: {
    type: String,
    default: "button",
  },
  active: {
    type: Boolean,
    default: false,
  },
});
</script>

<template>
  <component
    :is="as"
    class="w-fit bg-black/5 border border-black/10 hover:bg-black/15 hover:border-black/20 transition-colors px-4 py-3 focus-visible:ring-4 focus-visible:outline-none"
    :class="{
      'bg-blue-800/25 border-blue-800/50 hover:bg-blue-800/50 hover:border-blue-800/75':
        active,
    }"
    type="button"
  >
    <slot></slot>
  </component>
</template>

<template>
  <PhCircleNotch
    class="animate-spin text-9xl text-white fixed z-20"
    weight="bold"
  />
</template>

<script setup>
import { PhCircleNotch } from "@phosphor-icons/vue";

defineProps({
  size: {
    type: Number,
    default: 20,
  },
});
</script>

<script setup>
import { ref, onMounted } from "vue";

const histories = ref([]);

onMounted(() => {
  const saved = localStorage.getItem("pdf-histories");
  histories.value = saved ? JSON.parse(saved) : [];
});

const onGenerated = (data) => {
  histories.value.unshift(data);

  localStorage.setItem(
    "pdf-histories",
    JSON.stringify(histories.value)
  );
};
console.log(typeof window);
</script>

<template>
  <div class="min-h-screen bg-gray-100 md:px-40">
    <div class="flex flex-col gap-2">
      <AppHeader />
      <GenerateForm @generated="onGenerated" />
      <HistoryTable :items="histories" />
    </div>
  </div>
</template>

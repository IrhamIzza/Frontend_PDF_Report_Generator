<template>
  <section class="bg-white rounded-xl shadow p-4 md:p-6">
    <h2 class="text-lg font-semibold mb-4">History Generate</h2>

    <!-- Empty State -->
    <p
      v-if="sortedItems.length === 0"
      class="text-center text-gray-500 py-6"
    >
      Belum ada data
    </p>

    <!-- Table -->
    <div v-else class="overflow-x-auto">
      <table class="min-w-full border border-gray-200 text-sm">
        <thead class="bg-gray-200">
          <tr>
            <th class="px-3 py-2 border text-left">No</th>
            <th class="px-3 py-2 border text-left">Judul</th>
            <th class="px-3 py-2 border text-left">Ukuran</th>
            <th class="px-3 py-2 border text-left">Nominal</th>
            <th class="px-3 py-2 border text-left">Tanggal</th>
            <th class="px-3 py-2 border text-center">Aksi</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(item, index) in sortedItems"
            :key="index"
            :class="index % 2 === 0 ? 'bg-white' : 'bg-gray-100'"
          >
            <td class="px-3 py-2 border">{{ index + 1 }}</td>
            <td class="px-3 py-2 border">{{ item.title }}</td>
            <td class="px-3 py-2 border">{{ item.pageSize }}</td>
            <td class="px-3 py-2 border ">
              Rp {{ item.amount.toLocaleString('id-ID') }}
            </td>
            <td class="px-3 py-2 border">
              {{ formatDate(item.date) }}
            </td>

            <!-- Aksi -->
            <td class="px-3 py-2 border text-center space-x-2">
              <a
                :href="item.pdfUrl"
                target="_blank"
                class="inline-block bg-blue-500 hover:bg-blue-600
                       text-white text-xs px-3 py-1 rounded"
              >
                Lihat
              </a>

              <a
                :href="item.pdfUrl"
                download
                class="inline-block bg-green-500 hover:bg-green-600
                       text-white text-xs px-3 py-1 rounded"
              >
                Download
              </a>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  items: {
    type: Array,
    default: () => []
  }
})

const sortedItems = computed(() => {
  return [...props.items].sort(
    (a, b) => new Date(b.date) - new Date(a.date)
  )
})

const formatDate = (date) => {
  return new Date(date).toLocaleString('id-ID')
}
</script>

<template>
  <div
    v-if="isLoading"
    class="fixed inset-0 bg-black/50 flex items-center justify-center z-10"
  >
    <LoadingSpinner />
  </div>

  <!-- SUCCESS TOAST -->
  <div
    v-if="successMessage"
    class="fixed top-5 right-5 z-50 bg-green-600 text-white px-4 py-3 rounded-lg shadow-lg flex items-center gap-2 animate-slide-in"
  >
    <span>✅</span>
    <span>{{ successMessage }}</span>
  </div>

  <!-- ERROR TOAST -->
  <div
    v-if="errorMessage"
    class="fixed top-5 right-5 z-50 bg-red-600 text-white px-4 py-3 rounded-lg shadow-lg flex items-center gap-2 animate-slide-in"
  >
    <span>❌</span>
    <span>{{ errorMessage }}</span>
  </div>

  <section class="bg-white rounded-xl shadow p-4 md:p-6">
    <h2 class="text-lg font-semibold mb-4">Generate Report</h2>
    <form
      @submit.prevent="handleSubmit"
      class="grid grid-cols-1 md:grid-cols-2 gap-4"
    >
      <!-- Ukuran Halaman -->
      <div>
        <label class="block text-sm font-medium text-gray-700">
          Ukuran Halaman <span class="text-red-500">*</span>
        </label>
        <select
          v-model="form.pageSize"
          class="mt-1 w-full rounded-lg border px-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
        >
          <option value="A4">A4</option>
          <option value="A5">A5</option>
          <option value="Letter">Letter</option>
        </select>
        <p v-if="errors.pageSize" class="text-sm text-red-500 mt-1">
          {{ errors.pageSize }}
        </p>
      </div>

      <!-- Judul Laporan -->
      <div>
        <label class="block text-sm font-medium text-gray-700">
          Judul Laporan <span class="text-red-500">*</span>
        </label>
        <input
          v-model="form.title"
          type="text"
          maxlength="100"
          placeholder="Masukkan judul laporan..."
          class="mt-1 w-full rounded-lg border px-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
        />
        <p class="text-xs text-gray-500 mt-1">
          {{ form.title.length }}/100 karakter
        </p>
        <p v-if="errors.title" class="text-sm text-red-500 mt-1">
          {{ errors.title }}
        </p>
      </div>

      <!-- Deskripsi -->
      <div class="md:col-span-2">
        <label class="block text-sm font-medium text-gray-700">
          Deskripsi / Isi Laporan <span class="text-red-500">*</span>
        </label>
        <textarea
          v-model="form.description"
          rows="4"
          placeholder="Masukkan isi laporan..."
          class="mt-1 w-full rounded-lg border px-2 border-gray-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
        ></textarea>
        <p class="text-xs text-gray-500 mt-1">
          {{ form.description.length }} karakter
        </p>
        <p v-if="errors.description" class="text-sm text-red-500 mt-1">
          {{ errors.description }}
        </p>
      </div>

      <!-- Nominal -->
      <div>
        <label class="block text-sm font-medium text-gray-700">
          Nominal (Rp) <span class="text-red-500">*</span>
        </label>
        <div class="flex">
          <span
            class="inline-flex items-center px-3 rounded-l-lg border border-r-0 bg-gray-100 text-gray-600"
          >
            Rp
          </span>
          <input
            type="number"
            :value="formattedAmount"
            @input="onAmountInput"
            placeholder="0"
            class="w-full rounded-r-lg border-1 px-2 border-gray-300 focus:ring-indigo-500 focus:border-indigo-500"
          />
        </div>
        <p v-if="errors.amount" class="text-sm text-red-500 mt-1">
          {{ errors.amount }}
        </p>
      </div>

      <!-- Button -->
      <div class="flex items-end">
        <button
          type="submit"
          :disabled="isLoading"
          class="w-full bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 cursor-pointer disabled:cursor-not-allowed text-white font-semibold py-2 px-4 rounded-lg"
        >
          Generate PDF
          <PhFilePdf class="inline-block ml-1 text-xl" weight="bold" />
        </button>
      </div>
    </form>
  </section>
</template>

<script setup>
import { reactive, ref, computed } from "vue";
import jsPDF from "jspdf";
import LoadingSpinner from "./LoadingSpinner.vue";
import { PhFilePdf } from "@phosphor-icons/vue";

const emit = defineEmits(["generated"]);

const isLoading = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const form = reactive({
  pageSize: "",
  title: "",
  description: "",
  amount: 0,
});

const errors = reactive({
  pageSize: "",
  title: "",
  description: "",
  amount: "",
});

/* ===== Rupiah Formatter ===== */
const formattedAmount = computed(() =>
  form.amount ? form.amount.toLocaleString("id-ID") : ""
);

const onAmountInput = (e) => {
  const raw = e.target.value.replace(/\D/g, "");
  form.amount = raw ? Number(raw) : 0;
};

/* ===== Reset Form ===== */
const resetForm = () => {
  form.pageSize = "";
  form.title = "";
  form.description = "";
  form.amount = 0;
};

/* ===== Submit ===== */
const handleSubmit = async () => {
  if (isLoading.value) return;

  successMessage.value = "";
  errorMessage.value = "";
  Object.keys(errors).forEach((k) => (errors[k] = ""));

  let valid = true;

  if (!form.pageSize) {
    errors.pageSize = "Ukuran halaman wajib dipilih";
    valid = false;
  }

  if (!form.title.trim() || form.title.trim().length < 5) {
    errors.title = "Judul laporan minimal 5 karakter";
    valid = false;
  }

  if (!form.description.trim() || form.description.trim().length < 10) {
    errors.description = "Deskripsi minimal 10 karakter";
    valid = false;
  }

  if (form.amount < 0) {
    errors.amount = "Nominal tidak boleh kurang dari 0";
    valid = false;
  }

  if (!valid) return;

  try {
    isLoading.value = true;

    /* ===== GENERATE PDF ===== */

    const pdf = new jsPDF({
      format: form.pageSize, // A4 / A5 / Letter
      unit: "mm",
    });

    const pageWidth = pdf.internal.pageSize.getWidth();
    const margin = 20;
    const contentWidth = pageWidth - margin * 2;

    pdf.setFontSize(16);

    pdf.setFont("helvetica", "bold");
    pdf.text(form.title, pageWidth / 2, 20, { align: "center" });
    pdf.setFont("helvetica", "normal");
    pdf.setFontSize(12);
    const desc = pdf.splitTextToSize(form.description, contentWidth);
    pdf.text(desc, margin, 35);
    pdf.text(`Nominal: Rp ${form.amount.toLocaleString("id-ID")}`, margin, 80);


    const blob = pdf.output("blob");
    const pdfUrl = URL.createObjectURL(blob);

    pdf.save(`${form.title}.pdf`);

    /* ===== SIMPAN HISTORY ===== */
    emit("generated", {
      title: form.title,
      pageSize: form.pageSize,
      amount: form.amount,
      date: new Date(),
      pdfUrl,
    });

    await new Promise((resolve) => setTimeout(resolve, 500));

    successMessage.value = "PDF berhasil di-generate";

    setTimeout(() => {
      successMessage.value = "";
    }, 3000);
    resetForm();
  } catch (err) {
    errorMessage.value = "Terjadi kesalahan saat generate PDF";

    setTimeout(() => {
      errorMessage.value = "";
    }, 3000);
  } finally {
    isLoading.value = false;
  }
};
</script>

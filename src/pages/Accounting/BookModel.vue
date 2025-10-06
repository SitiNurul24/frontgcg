<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <h2 class="text-xl font-bold mb-4">Accounting</h2>

    <div class="bg-white rounded-lg shadow-md overflow-hidden">
      <div class="flex items-center justify-between bg-[#dff2f8] border-b border-[#c8e8f2] px-6 py-4">
        <div>
          <h3 class="text-lg font-semibold text-gray-700">Book Model</h3>
          <p class="text-sm text-gray-500">Manage report templates and their default book models.</p>
        </div>
        <div class="flex items-center gap-3">
          <button
            class="rounded-md bg-[#02A2DC] px-4 py-2 text-sm font-semibold text-white shadow hover:bg-[#0191c4] transition"
            @click="handleNew"
          >
            New
          </button>
          <button
            class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
            @click="handleOpen"
          >
            Open
          </button>
          <button
            class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
            @click="toggleReportTypePanel"
          >
            Report Type
          </button>
          <button
            class="flex h-9 w-9 items-center justify-center rounded-full border border-[#02A2DC] text-[#02A2DC] hover:bg-[#e6f7ff]"
            @click="goBack"
            aria-label="Close"
          >
            <span class="material-icons text-base">close</span>
          </button>
        </div>
      </div>

      <transition name="fade">
        <div v-if="showReportTypePanel" class="border-b border-[#c8e8f2] bg-[#f4fbfe] px-6 py-3 text-sm text-gray-600">
          Select a report type from the list below to filter available book models.
        </div>
      </transition>

      <div class="grid gap-6 px-6 py-6 md:grid-cols-[320px_1fr]">
        <div class="flex flex-col gap-4 rounded-lg border border-[#e5f4fa] bg-[#f8fdff] p-5">
          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700" for="reportType">Report Type<span class="text-red-500">*</span></label>
            <select
              id="reportType"
              v-model="form.reportType"
              class="mt-2 rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
            >
              <option value="" disabled>Select report type</option>
              <option v-for="type in reportTypes" :key="type" :value="type">
                {{ type }}
              </option>
            </select>
          </div>

          <div class="flex flex-col">
            <label class="text-sm font-semibold text-gray-700" for="bookModel">Book Model<span class="text-red-500">*</span></label>
            <input
              id="bookModel"
              v-model="form.bookModel"
              type="text"
              placeholder="Enter book model name"
              class="mt-2 rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
            />
          </div>

          <div class="flex justify-end gap-3 pt-2">
            <button
              class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
              @click="handleReset"
            >
              Reset
            </button>
            <button
              class="rounded-md bg-[#02A2DC] px-4 py-2 text-sm font-semibold text-white shadow hover:bg-[#0191c4] transition"
              @click="handleSave"
            >
              Save
            </button>
          </div>
        </div>

        <div class="flex flex-col gap-4">
          <div class="flex items-center justify-between">
            <div class="text-sm text-gray-600">
              Showing {{ filteredBookModels.length }} of {{ bookModels.length }} book models
            </div>
            <div class="relative w-full max-w-xs">
              <input
                v-model="searchQuery"
                type="text"
                placeholder="Search book model"
                class="w-full rounded-md border border-gray-300 px-3 py-2 pr-10 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
              />
              <span class="material-icons pointer-events-none absolute right-3 top-2.5 text-gray-400">search</span>
            </div>
          </div>

          <div class="overflow-hidden rounded-lg border border-gray-200 bg-white">
            <table class="min-w-full divide-y divide-gray-200 text-sm">
              <thead class="bg-[#02A2DC] text-left text-xs font-semibold uppercase tracking-wider text-white">
                <tr>
                  <th class="px-5 py-3">Report Type</th>
                  <th class="px-5 py-3">Book Model</th>
                  <th class="px-5 py-3 text-right">Action</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-100 bg-white">
                <tr
                  v-for="model in filteredBookModels"
                  :key="model.id"
                  class="hover:bg-[#f4fbfe]"
                >
                  <td class="px-5 py-3 text-gray-700">{{ model.reportType }}</td>
                  <td class="px-5 py-3 text-gray-700">{{ model.bookModel }}</td>
                  <td class="px-5 py-3 text-right">
                    <button
                      class="rounded-full border border-[#02A2DC] px-3 py-1 text-xs font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
                      @click="selectModel(model)"
                    >
                      View
                    </button>
                  </td>
                </tr>
                <tr v-if="filteredBookModels.length === 0">
                  <td class="px-5 py-6 text-center text-sm text-gray-500" colspan="3">
                    No book models available. Create a new model to get started.
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const reportTypes = [
  'General Ledger',
  'Profit and Loss',
  'Balance Sheet',
  'Cash Flow Statement',
]

const bookModels = ref([
  { id: 1, reportType: 'General Ledger', bookModel: 'GL Default Model' },
  { id: 2, reportType: 'Profit and Loss', bookModel: 'P&L Consolidated' },
  { id: 3, reportType: 'Balance Sheet', bookModel: 'BS Monthly Review' },
])

const form = ref({
  id: null,
  reportType: '',
  bookModel: '',
})

const searchQuery = ref('')
const showReportTypePanel = ref(false)

const filteredBookModels = computed(() => {
  const query = searchQuery.value.trim().toLowerCase()

  if (!query) {
    return bookModels.value
  }

  return bookModels.value.filter((item) => {
    return (
      item.reportType.toLowerCase().includes(query) ||
      item.bookModel.toLowerCase().includes(query)
    )
  })
})

const handleNew = () => {
  form.value = { id: null, reportType: '', bookModel: '' }
}

const handleOpen = () => {
  if (bookModels.value.length > 0) {
    selectModel(bookModels.value[0])
  }
}

const toggleReportTypePanel = () => {
  showReportTypePanel.value = !showReportTypePanel.value
}

const selectModel = (model) => {
  form.value = { ...model }
}

const handleReset = () => {
  form.value = { id: null, reportType: '', bookModel: '' }
}

const handleSave = () => {
  if (!form.value.reportType || !form.value.bookModel) {
    return
  }

  if (form.value.id) {
    bookModels.value = bookModels.value.map((item) =>
      item.id === form.value.id ? { ...form.value } : item
    )
  } else {
    const newId = Math.max(0, ...bookModels.value.map((item) => item.id)) + 1
    bookModels.value = [
      ...bookModels.value,
      { id: newId, reportType: form.value.reportType, bookModel: form.value.bookModel },
    ]
  }

  handleReset()
}

const goBack = () => {
  router.back()
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <h2 class="text-xl font-bold mb-4">Accounting</h2>

    <div class="bg-white rounded-lg shadow-md overflow-hidden">
      <div class="flex flex-wrap items-center justify-between gap-3 bg-[#dff2f8] border-b border-[#c8e8f2] px-6 py-4">
        <div>
          <h3 class="text-lg font-semibold text-gray-700">Book Model</h3>
          <p class="text-sm text-gray-500">Manage report templates and their default book models.</p>
        </div>
        <div class="flex flex-wrap items-center gap-3">
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

      <transition name="slide-fade">
        <section
          v-if="showForm"
          class="border-b border-[#c8e8f2] bg-[#f8fdff] px-6 py-6"
        >
          <div class="rounded-md border border-[#c8e8f2] bg-white shadow-sm">
            <div class="flex flex-wrap items-center justify-between gap-3 border-b border-[#c8e8f2] bg-[#ecf7fb] px-5 py-4">
              <h4 class="text-base font-semibold text-gray-700">
                {{ isEditing ? 'Edit Book Model' : 'New Book Model' }}
              </h4>
              <div class="flex flex-wrap items-center gap-2">
                <button
                  class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
                  @click="handleBack"
                >
                  Back
                </button>
                <button
                  class="rounded-md bg-[#02A2DC] px-4 py-2 text-sm font-semibold text-white shadow hover:bg-[#0191c4] transition"
                  @click="handleSave"
                >
                  Save
                </button>
                <div class="relative">
                  <button
                    class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
                    @click="toggleCopyFrom"
                  >
                    Copy From
                  </button>
                  <transition name="fade">
                    <div
                      v-if="copyFromOpen"
                      class="absolute right-0 z-10 mt-2 w-64 rounded-md border border-[#c8e8f2] bg-white shadow-lg"
                    >
                      <div class="px-4 py-3 text-xs font-semibold uppercase tracking-wider text-gray-500">
                        Existing Book Models
                      </div>
                      <ul class="max-h-48 overflow-y-auto text-sm">
                        <li
                          v-for="model in bookModels"
                          :key="model.id"
                          class="border-t border-[#eef6fa] first:border-t-0"
                        >
                          <button
                            class="flex w-full items-start gap-2 px-4 py-3 text-left hover:bg-[#f4fbfe]"
                            @click="applyCopyFrom(model)"
                          >
                            <span class="mt-0.5 h-2 w-2 rounded-full bg-[#02A2DC]"></span>
                            <span>
                              <span class="block font-semibold text-gray-700">{{ model.bookModel }}</span>
                              <span class="block text-xs text-gray-500">{{ model.reportType }}</span>
                            </span>
                          </button>
                        </li>
                        <li v-if="bookModels.length === 0" class="px-4 py-3 text-sm text-gray-500">
                          No book models available yet.
                        </li>
                      </ul>
                    </div>
                  </transition>
                </div>
              </div>
            </div>

            <div class="px-5 py-6">
              <div class="grid gap-5 md:grid-cols-2 xl:grid-cols-4">
                <div class="md:col-span-1 xl:col-span-1">
                  <label class="text-sm font-semibold text-gray-700" for="bookModel">Book Model<span class="text-red-500">*</span></label>
                  <input
                    id="bookModel"
                    v-model="form.bookModel"
                    type="text"
                    placeholder="Enter book model name"
                    class="mt-2 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                  />
                  <p v-if="errors.bookModel" class="mt-1 text-xs text-red-500">{{ errors.bookModel }}</p>
                </div>
                <div class="md:col-span-1 xl:col-span-2">
                  <label class="text-sm font-semibold text-gray-700" for="description">Description<span class="text-red-500">*</span></label>
                  <textarea
                    id="description"
                    v-model="form.description"
                    rows="3"
                    placeholder="Provide a short description"
                    class="mt-2 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                  ></textarea>
                  <p v-if="errors.description" class="mt-1 text-xs text-red-500">{{ errors.description }}</p>
                </div>
                <div class="md:col-span-1 xl:col-span-1">
                  <label class="text-sm font-semibold text-gray-700" for="chartOfAccount">Chart of Account<span class="text-red-500">*</span></label>
                  <input
                    id="chartOfAccount"
                    v-model="form.chartOfAccount"
                    type="text"
                    placeholder="COA reference"
                    class="mt-2 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                  />
                  <p v-if="errors.chartOfAccount" class="mt-1 text-xs text-red-500">{{ errors.chartOfAccount }}</p>
                </div>
                <div class="md:col-span-1">
                  <label class="text-sm font-semibold text-gray-700" for="reportType">Report Type<span class="text-red-500">*</span></label>
                  <select
                    id="reportType"
                    v-model="form.reportType"
                    class="mt-2 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                  >
                    <option value="" disabled>Select report type</option>
                    <option v-for="type in reportTypes" :key="type" :value="type">
                      {{ type }}
                    </option>
                  </select>
                  <p v-if="errors.reportType" class="mt-1 text-xs text-red-500">{{ errors.reportType }}</p>
                </div>
                <div class="md:col-span-1">
                  <label class="text-sm font-semibold text-gray-700" for="calculation">Calculation<span class="text-red-500">*</span></label>
                  <select
                    id="calculation"
                    v-model="form.calculation"
                    class="mt-2 w-full rounded-md border border-gray-300 bg-white px-3 py-2 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                  >
                    <option value="" disabled>Select calculation</option>
                    <option v-for="option in calculationOptions" :key="option" :value="option">
                      {{ option }}
                    </option>
                  </select>
                  <p v-if="errors.calculation" class="mt-1 text-xs text-red-500">{{ errors.calculation }}</p>
                </div>
              </div>

              <div class="mt-6 overflow-hidden rounded-lg border border-[#c8e8f2]">
                <table class="min-w-full divide-y divide-[#e5f4fa] text-sm">
                  <thead class="bg-[#02A2DC] text-left text-xs font-semibold uppercase tracking-wider text-white">
                    <tr>
                      <th class="px-4 py-3">Line</th>
                      <th class="px-4 py-3">Account</th>
                      <th class="px-4 py-3">Description</th>
                      <th class="px-4 py-3">Sum Lines</th>
                      <th class="px-4 py-3">Formula</th>
                      <th class="px-4 py-3">Cr.</th>
                      <th class="px-4 py-3">Format</th>
                      <th class="px-4 py-3">Display</th>
                      <th class="px-4 py-3 text-right">Actions</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-[#eef6fa] bg-white">
                    <tr v-for="(row, index) in form.details" :key="index" class="hover:bg-[#f4fbfe]">
                      <td class="px-4 py-3">
                        <input
                          v-model="row.line"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="Line"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <input
                          v-model="row.account"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="Account"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <input
                          v-model="row.description"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="Description"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <input
                          v-model="row.sumLines"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="e.g. 1,2,3"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <input
                          v-model="row.formula"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="Formula"
                        />
                      </td>
                      <td class="px-4 py-3 text-center">
                        <input
                          v-model="row.credit"
                          type="checkbox"
                          class="h-4 w-4 accent-[#02A2DC]"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <input
                          v-model="row.format"
                          type="text"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                          placeholder="Format"
                        />
                      </td>
                      <td class="px-4 py-3">
                        <select
                          v-model="row.display"
                          class="w-full rounded border border-gray-300 px-2 py-1 text-xs focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                        >
                          <option :value="true">Show</option>
                          <option :value="false">Hide</option>
                        </select>
                      </td>
                      <td class="px-4 py-3 text-right">
                        <button
                          class="rounded-md border border-red-300 px-3 py-1 text-xs font-semibold text-red-500 hover:bg-red-50"
                          @click="removeDetailRow(index)"
                        >
                          Remove
                        </button>
                      </td>
                    </tr>
                    <tr v-if="form.details.length === 0">
                      <td class="px-4 py-6 text-center text-sm text-gray-500" colspan="9">
                        No detail lines yet. Add a line to start building your model.
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>

              <div class="mt-4 flex flex-wrap items-center justify-between gap-3">
                <div class="text-xs text-gray-500">
                  Configure the line breakdown that will be used when generating the report.
                </div>
                <div class="flex flex-wrap items-center gap-2">
                  <button
                    class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition"
                    @click="addDetailRow"
                  >
                    Add
                  </button>
                  <button
                    class="rounded-md border border-[#02A2DC] px-4 py-2 text-sm font-semibold text-[#02A2DC] hover:bg-[#e6f7ff] transition disabled:cursor-not-allowed disabled:border-gray-200 disabled:text-gray-400"
                    :disabled="form.details.length === 0"
                    @click="removeLastDetailRow"
                  >
                    Remove
                  </button>
                </div>
              </div>
            </div>
          </div>
        </section>
      </transition>

      <div class="px-6 py-6">
        <div class="flex flex-col gap-5">
          <div class="flex flex-wrap items-center justify-between gap-3">
            <div class="text-sm text-gray-600">
              Showing {{ filteredBookModels.length }} of {{ bookModels.length }} book models
            </div>
          </div>

          <div class="grid w-full gap-4 sm:grid-cols-2 md:max-w-2xl">
            <div>
              <label class="text-sm font-semibold text-gray-700" for="filterReportType">
                Report Type<span class="text-red-500">*</span>
              </label>
              <div class="relative mt-2">
                <input
                  id="filterReportType"
                  v-model="reportTypeQuery"
                  type="text"
                  placeholder="Search report type"
                  class="w-full rounded-md border border-gray-300 px-3 py-2 pr-10 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                />
                <span class="material-icons pointer-events-none absolute right-3 top-2.5 text-gray-400">search</span>
              </div>
            </div>
            <div>
              <label class="text-sm font-semibold text-gray-700" for="filterBookModel">
                Book Model<span class="text-red-500">*</span>
              </label>
              <div class="relative mt-2">
                <input
                  id="filterBookModel"
                  v-model="bookModelQuery"
                  type="text"
                  placeholder="Search book model"
                  class="w-full rounded-md border border-gray-300 px-3 py-2 pr-10 text-sm shadow-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                />
                <span class="material-icons pointer-events-none absolute right-3 top-2.5 text-gray-400">search</span>
              </div>
            </div>
          </div>

          <div class="overflow-hidden rounded-lg border border-gray-200 bg-white">
            <table class="min-w-full divide-y divide-gray-200 text-sm">
              <thead class="bg-[#02A2DC] text-left text-xs font-semibold uppercase tracking-wider text-white">
                <tr>
                  <th class="px-5 py-3">Report Type</th>
                  <th class="px-5 py-3">Book Model</th>
                  <th class="px-5 py-3">Calculation</th>
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
                  <td class="px-5 py-3 text-gray-700">{{ model.calculation }}</td>
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
                  <td class="px-5 py-6 text-center text-sm text-gray-500" colspan="4">
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

const calculationOptions = ['Periodic', 'Summary']

const createDetailRow = () => ({
  line: '',
  account: '',
  description: '',
  sumLines: '',
  formula: '',
  credit: false,
  format: '',
  display: true,
})

const createEmptyForm = () => ({
  id: null,
  bookModel: '',
  description: '',
  chartOfAccount: '',
  reportType: '',
  calculation: 'Periodic',
  details: [createDetailRow()],
})

const bookModels = ref([
  {
    id: 1,
    reportType: 'General Ledger',
    bookModel: 'GL Default Model',
    description: 'Standard GL structure for monthly review.',
    chartOfAccount: 'COA-1001',
    calculation: 'Periodic',
    details: [
      {
        line: '100',
        account: '1100',
        description: 'Cash and cash equivalents',
        sumLines: '',
        formula: '',
        credit: false,
        format: 'Currency',
        display: true,
      },
    ],
  },
  {
    id: 2,
    reportType: 'Profit and Loss',
    bookModel: 'P&L Consolidated',
    description: 'Consolidated profit and loss template.',
    chartOfAccount: 'COA-2400',
    calculation: 'Summary',
    details: [
      {
        line: '200',
        account: '4100',
        description: 'Revenue',
        sumLines: '',
        formula: '',
        credit: false,
        format: 'Currency',
        display: true,
      },
      {
        line: '210',
        account: '5100',
        description: 'Cost of Goods Sold',
        sumLines: '',
        formula: '',
        credit: true,
        format: 'Currency',
        display: true,
      },
    ],
  },
])

const form = ref(createEmptyForm())
const errors = ref({})
const reportTypeQuery = ref('')
const bookModelQuery = ref('')
const showReportTypePanel = ref(false)
const showForm = ref(false)
const isEditing = ref(false)
const copyFromOpen = ref(false)

const filteredBookModels = computed(() => {
  const reportTypeFilter = reportTypeQuery.value.trim().toLowerCase()
  const bookModelFilter = bookModelQuery.value.trim().toLowerCase()

  return bookModels.value.filter((item) => {
    const matchesReportType = reportTypeFilter
      ? item.reportType.toLowerCase().includes(reportTypeFilter)
      : true
    const matchesBookModel = bookModelFilter
      ? item.bookModel.toLowerCase().includes(bookModelFilter)
      : true

    return matchesReportType && matchesBookModel
  })
})

const resetFormState = () => {
  form.value = createEmptyForm()
  errors.value = {}
  copyFromOpen.value = false
}

const handleNew = () => {
  resetFormState()
  isEditing.value = false
  showForm.value = true
}

const handleOpen = () => {
  if (bookModels.value.length > 0) {
    selectModel(bookModels.value[0])
  }
}

const toggleReportTypePanel = () => {
  showReportTypePanel.value = !showReportTypePanel.value
}

const toggleCopyFrom = () => {
  copyFromOpen.value = !copyFromOpen.value
}

const selectModel = (model) => {
  form.value = {
    id: model.id,
    bookModel: model.bookModel,
    description: model.description || '',
    chartOfAccount: model.chartOfAccount || '',
    reportType: model.reportType || '',
    calculation: model.calculation || 'Periodic',
    details: (model.details || []).map((detail) => ({ ...detail })),
  }
  errors.value = {}
  isEditing.value = true
  showForm.value = true
  copyFromOpen.value = false
}

const applyCopyFrom = (model) => {
  const cloned = {
    bookModel: `${model.bookModel} Copy`,
    description: model.description || '',
    chartOfAccount: model.chartOfAccount || '',
    reportType: model.reportType || '',
    calculation: model.calculation || 'Periodic',
    details: (model.details || []).map((detail) => ({ ...detail })),
  }
  form.value = {
    ...createEmptyForm(),
    ...cloned,
    id: null,
  }
  isEditing.value = false
  errors.value = {}
  copyFromOpen.value = false
  showForm.value = true
}

const validateForm = () => {
  const nextErrors = {}
  if (!form.value.bookModel.trim()) {
    nextErrors.bookModel = 'Book model name is required.'
  }
  if (!form.value.description.trim()) {
    nextErrors.description = 'Description is required.'
  }
  if (!form.value.chartOfAccount.trim()) {
    nextErrors.chartOfAccount = 'Chart of account is required.'
  }
  if (!form.value.reportType) {
    nextErrors.reportType = 'Report type is required.'
  }
  if (!form.value.calculation) {
    nextErrors.calculation = 'Calculation type is required.'
  }

  errors.value = nextErrors
  return Object.keys(nextErrors).length === 0
}

const handleSave = () => {
  if (!validateForm()) {
    return
  }

  const payload = {
    ...form.value,
    details: form.value.details.map((detail) => ({ ...detail })),
  }

  if (payload.id) {
    bookModels.value = bookModels.value.map((item) =>
      item.id === payload.id ? { ...payload } : item
    )
  } else {
    const newId = Math.max(0, ...bookModels.value.map((item) => item.id)) + 1
    bookModels.value = [
      ...bookModels.value,
      { ...payload, id: newId },
    ]
  }

  handleBack()
}

const handleBack = () => {
  resetFormState()
  isEditing.value = false
  showForm.value = false
}

const addDetailRow = () => {
  form.value.details = [...form.value.details, createDetailRow()]
}

const removeDetailRow = (index) => {
  form.value.details = form.value.details.filter((_, idx) => idx !== index)
}

const removeLastDetailRow = () => {
  if (form.value.details.length === 0) return
  form.value.details = form.value.details.slice(0, -1)
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

.slide-fade-enter-active {
  transition: all 0.25s ease;
}

.slide-fade-enter-from {
  opacity: 0;
  transform: translateY(-8px);
}

.slide-fade-leave-active {
  transition: all 0.2s ease;
}

.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}
</style>

<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <h2 class="text-xl font-bold mb-4">Accounting</h2>

    <!-- Search + Card Area -->
    <div v-if="!showForm" class="bg-white rounded-lg shadow-md p-6 mb-6">
      <div class="mb-6">
        <h3 class="text-lg font-semibold mb-2">Payable</h3>

        <!-- Selection Inputs -->
        <div class="grid gap-4 mb-6 md:grid-cols-2">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Report Type*</label>
            <div class="relative">
              <input
                type="text"
                :value="selectedReportTypeLabel"
                readonly
                placeholder="Select Report Type"
                class="w-full p-3 pr-12 border border-gray-300 rounded-md shadow-sm bg-white focus:outline-none focus:ring-2 focus:ring-blue-400"
              />
              <button
                type="button"
                @click="openReportTypePicker"
                class="absolute inset-y-0 right-0 flex items-center px-3 text-[#02A2DC] hover:text-[#0191b8]"
              >
                <span class="material-icons">arrow_drop_down</span>
              </button>
            </div>
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Book Model*</label>
            <div class="relative">
              <input
                type="text"
                :value="selectedBookModelLabel"
                readonly
                :placeholder="selectedReportType ? 'Select Book Model' : 'Select Report Type first'"
                :class="[
                  'w-full p-3 pr-12 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-400',
                  selectedReportType ? 'bg-white' : 'bg-gray-100 cursor-not-allowed focus:ring-0'
                ]"
              />
              <button
                type="button"
                @click="openBookModelPicker"
                :disabled="!selectedReportType"
                class="absolute inset-y-0 right-0 flex items-center px-3"
                :class="selectedReportType ? 'text-[#02A2DC] hover:text-[#0191b8]' : 'text-gray-400'"
              >
                <span class="material-icons">arrow_drop_down</span>
              </button>
            </div>
          </div>
        </div>

        <!-- Search Input -->
        <div class="relative w-full md:w-1/3 mb-6">
          <input
            type="text"
            v-model="searchQuery"
            placeholder="Search"
            class="w-full p-3 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
          <span class="material-icons absolute right-3 top-3 text-gray-400">search</span>
        </div>

        <!-- Table View -->
        <div>
          <div class="overflow-x-auto border border-gray-200 rounded">
            <table class="min-w-[1200px] w-full text-sm table-fixed">
              <thead class="bg-[#02A2DC] text-white font-bold">
                <tr>
                  <th class="p-4 border text-center" style="width: 15%">Document<br />Number</th>
                  <th class="p-4 border text-center" style="width: 15%">Doc Ref</th>
                  <th class="p-4 border text-center" style="width: 15%">Doc Date</th>
                  <th class="p-4 border text-center" style="width: 15%">Supplier</th>
                  <th class="p-4 border text-center" style="width: 15%">Local Amount</th>
                  <th class="p-4 border text-center" style="width: 10%">Company</th>
                  <th class="p-4 border text-center" style="width: 15%">Description</th>
                  <th class="p-4 border text-center" style="width: 7%">Status</th>
                  <th class="p-4 border text-center" style="width: 10%">Action</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(item, index) in filteredTransactions"
                  :key="index"
                  class="text-center hover:bg-gray-50"
                >
                  <td class="p-4 border">{{ item.docNo }}</td>
                  <td class="p-4 border">{{ item.docRef }}</td>
                  <td class="p-4 border">{{ item.docDate }}</td>
                  <td class="p-4 border">{{ item.supplier }}</td>
                  <td class="p-4 border">{{ formatAmount(item.localAmount) }}</td>
                  <td class="p-4 border">{{ item.company }}</td>
                  <td class="p-4 border">
                    {{ item.description || 'Payroll process - MLY:KEU:GEN:2024-11' }}
                  </td>
                  <td class="p-4 border">
                    <span
                      :class="{
                        'text-black-600 font-semi': item.status === 'Paid',
                        'text-black-600 font-semi': item.status === 'Unpaid',
                      }"
                    >
                      {{ item.status }}
                    </span>
                  </td>
                  <td class="p-4 border">
                    <button
                      @click="selectTransaction(item)"
                      class="bg-[#02A2DC] hover:bg-[#0191b8] text-white p-2 rounded-full flex items-center justify-center mx-auto"
                      title="View"
                    >
                      <span class="material-icons text-sm">visibility</span>
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- PAGINATION -->
          <div class="flex justify-center items-center mt-4 space-x-2 text-sm">
            <button class="text-blue-600 font-bold">1</button>
            <button class="text-gray-400 cursor-not-allowed">2</button>
            <button class="text-gray-400 cursor-not-allowed">3</button>

            <!-- Panah dalam lingkaran biru -->
            <button class="ml-2 w-8 h-8 rounded-full bg-[#00a7e1] flex items-center justify-center">
              <svg class="w-4 h-4 text-white" fill="currentColor" viewBox="0 0 20 20">
                <path
                  fill-rule="evenodd"
                  d="M10.293 15.707a1 1 0 010-1.414L13.586 11H4a1 1 0 110-2h9.586l-3.293-3.293a1 1 0 111.414-1.414l5 5a1 1 0 010 1.414l-5 5a1 1 0 01-1.414 0z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Form View (PAYABLE) -->
    <div v-if="showForm" class="bg-white rounded-lg shadow-md p-6 mb-6">
      <h3 class="text-lg font-semibold">Payable</h3>

      <button
        @click="handleCancel"
        class="flex items-center text-[#02A2DC] font-medium gap-2 hover:underline mt-2"
      >
        <span class="w-5 h-5 flex items-center justify-center rounded-full border border-[#02A2DC]">
          <span class="material-icons text-[14px] text-[#02A2DC]">arrow_back</span>
        </span>
        View Data
      </button>

      <hr class="border border-gray-200 my-4" />

      <!-- Tombol Aksi -->
      <div class="flex justify-end gap-2 mb-6">
        <button class="btn-primary" @click="handleSave">Save</button>
        <button class="btn-outline" @click="handleCancel">Cancel</button>
        <button class="btn-outline flex items-center gap-1" @click="handlePayment">
          Payment <span class="material-icons text-sm">payments</span>
        </button>
        <button class="btn-outline flex items-center gap-1">
          Print Invoice <span class="material-icons text-sm">print</span>
        </button>
        <button class="btn-outline flex items-center gap-1">
          Print Cover <span class="material-icons text-sm">print</span>
        </button>
      </div>

      <!-- Form Grid -->
      <div class="max-w-6xl mx-auto px-1 grid grid-cols-2 gap-4">
        <div>
          <label class="block text-sm font-medium text-gray-700">Company</label>
          <input
            type="text"
            v-model="selectedTransaction.company"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700">Doc Number</label>
          <input
            type="text"
            v-model="selectedTransaction.docNo"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700">Doc Ref</label>
          <input
            type="text"
            v-model="selectedTransaction.docRef"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700">Doc Date</label>
          <input
            type="text"
            v-model="selectedTransaction.docDate"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700">Supplier</label>
          <input
            type="text"
            v-model="selectedTransaction.supplier"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700">Local Amount</label>
          <input
            type="text"
            v-model="selectedTransaction.localAmount"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
          />
        </div>

        <div class="col-span-2">
          <label class="block text-sm font-medium text-gray-700">Description</label>
          <textarea
            v-model="selectedTransaction.description"
            class="w-full border border-gray-300 rounded px-3 py-2 bg-white"
            rows="2"
          ></textarea>
        </div>
      </div>
    </div>
  </div>

  <!-- Report Type Picker -->
  <div
    v-if="showReportTypePicker"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 px-4"
  >
    <div class="w-full max-w-xl bg-white rounded-lg shadow-xl p-6">
      <h4 class="text-lg font-semibold text-gray-700 mb-4">List of Report Type</h4>

      <div class="relative mb-4">
        <input
          type="text"
          v-model="reportTypeSearch"
          placeholder="Search"
          class="w-full p-3 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-400"
        />
        <span class="material-icons absolute right-3 top-3 text-gray-400">search</span>
      </div>

      <div class="border border-gray-200 rounded overflow-hidden mb-4">
        <table class="w-full text-sm">
          <thead class="bg-[#02A2DC] text-white">
            <tr>
              <th class="p-3 border border-gray-200 text-left">Type</th>
              <th class="p-3 border border-gray-200 text-left">Description</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="item in filteredReportTypes"
              :key="item.type"
              @click="selectReportType(item)"
              @dblclick="selectReportType(item, true)"
              :class="[
                'cursor-pointer transition-colors',
                tempSelectedReportType?.type === item.type ? 'bg-[#E6F7FB]' : 'hover:bg-gray-50'
              ]"
            >
              <td class="p-3 border border-gray-200">{{ item.type }}</td>
              <td class="p-3 border border-gray-200">{{ item.description }}</td>
            </tr>
            <tr v-if="!filteredReportTypes.length">
              <td colspan="2" class="p-4 text-center text-gray-500">No report types found</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="flex items-center justify-between text-sm text-gray-600 mb-4">
        <div>Records: {{ filteredReportTypes.length }}</div>
        <div>Page: 1</div>
      </div>

      <div class="flex justify-end gap-2">
        <button class="btn-outline" type="button" @click="cancelReportTypeSelection">Cancel</button>
        <button class="btn-primary" type="button" @click="confirmReportTypeSelection">OK</button>
      </div>
    </div>
  </div>

  <!-- Book Model Picker -->
  <div
    v-if="showBookModelPicker"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 px-4"
  >
    <div class="w-full max-w-xl bg-white rounded-lg shadow-xl p-6">
      <h4 class="text-lg font-semibold text-gray-700 mb-4">List of Book Model</h4>

      <div class="relative mb-4">
        <input
          type="text"
          v-model="bookModelSearch"
          placeholder="Search"
          class="w-full p-3 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-400"
        />
        <span class="material-icons absolute right-3 top-3 text-gray-400">search</span>
      </div>

      <div class="border border-gray-200 rounded overflow-hidden mb-4">
        <table class="w-full text-sm">
          <thead class="bg-[#02A2DC] text-white">
            <tr>
              <th class="p-3 border border-gray-200 text-left">Code</th>
              <th class="p-3 border border-gray-200 text-left">Book Model</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="item in filteredBookModels"
              :key="item.code"
              @click="selectBookModel(item)"
              @dblclick="selectBookModel(item, true)"
              :class="[
                'cursor-pointer transition-colors',
                tempSelectedBookModel?.code === item.code ? 'bg-[#E6F7FB]' : 'hover:bg-gray-50'
              ]"
            >
              <td class="p-3 border border-gray-200">{{ item.code }}</td>
              <td class="p-3 border border-gray-200">{{ item.name }}</td>
            </tr>
            <tr v-if="!filteredBookModels.length">
              <td colspan="2" class="p-4 text-center text-gray-500">No book models found</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="flex items-center justify-between text-sm text-gray-600 mb-4">
        <div>Records: {{ filteredBookModels.length }}</div>
        <div>Page: 1</div>
      </div>

      <div class="flex justify-end gap-2">
        <button class="btn-outline" type="button" @click="cancelBookModelSelection">Cancel</button>
        <button class="btn-primary" type="button" @click="confirmBookModelSelection">OK</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const showForm = ref(false)
const searchQuery = ref('')
const selectedTransaction = ref({})
const showPaymentForm = ref(false)

const reportTypes = ref([
  {
    type: 'DSB',
    description: 'Dashboard Setting',
    bookModels: [
      { code: 'DSB-001', name: 'Dashboard Summary Model' },
      { code: 'DSB-002', name: 'Executive Overview Model' },
    ],
  },
  {
    type: 'F1',
    description: 'Financial Report',
    bookModels: [
      { code: 'F1-001', name: 'General Ledger Model' },
      { code: 'F1-002', name: 'Accounts Payable Model' },
      { code: 'F1-003', name: 'Accounts Receivable Model' },
    ],
  },
  {
    type: 'G1',
    description: 'Budgeting Report',
    bookModels: [
      { code: 'G1-001', name: 'Annual Budget Model' },
      { code: 'G1-002', name: 'Quarterly Budget Model' },
    ],
  },
])

const selectedReportType = ref(null)
const selectedBookModel = ref(null)

const showReportTypePicker = ref(false)
const showBookModelPicker = ref(false)

const reportTypeSearch = ref('')
const bookModelSearch = ref('')

const tempSelectedReportType = ref(null)
const tempSelectedBookModel = ref(null)

const selectedReportTypeLabel = computed(() => {
  if (!selectedReportType.value) return ''
  return `${selectedReportType.value.type} - ${selectedReportType.value.description}`
})

const selectedBookModelLabel = computed(() => {
  if (!selectedBookModel.value) return ''
  return `${selectedBookModel.value.code} - ${selectedBookModel.value.name}`
})

const filteredReportTypes = computed(() => {
  const keyword = reportTypeSearch.value.toLowerCase().trim()
  if (!keyword) return reportTypes.value

  return reportTypes.value.filter(
    (item) =>
      item.type.toLowerCase().includes(keyword) ||
      item.description.toLowerCase().includes(keyword),
  )
})

const filteredBookModels = computed(() => {
  const keyword = bookModelSearch.value.toLowerCase().trim()
  const models = selectedReportType.value?.bookModels ?? []

  if (!keyword) return models

  return models.filter(
    (item) =>
      item.code.toLowerCase().includes(keyword) ||
      item.name.toLowerCase().includes(keyword),
  )
})

function openReportTypePicker() {
  reportTypeSearch.value = ''
  tempSelectedReportType.value = selectedReportType.value
  showReportTypePicker.value = true
}

function openBookModelPicker() {
  if (!selectedReportType.value) return
  bookModelSearch.value = ''
  tempSelectedBookModel.value = selectedBookModel.value
  showBookModelPicker.value = true
}

function selectReportType(item, confirmImmediately = false) {
  tempSelectedReportType.value = item
  if (confirmImmediately) {
    confirmReportTypeSelection()
  }
}

function confirmReportTypeSelection() {
  if (!tempSelectedReportType.value) return
  selectedReportType.value = tempSelectedReportType.value
  selectedBookModel.value = null
  showReportTypePicker.value = false
  reportTypeSearch.value = ''
  tempSelectedReportType.value = null
}

function cancelReportTypeSelection() {
  showReportTypePicker.value = false
  reportTypeSearch.value = ''
  tempSelectedReportType.value = null
}

function selectBookModel(item, confirmImmediately = false) {
  tempSelectedBookModel.value = item
  if (confirmImmediately) {
    confirmBookModelSelection()
  }
}

function confirmBookModelSelection() {
  if (!tempSelectedBookModel.value) return
  selectedBookModel.value = tempSelectedBookModel.value
  showBookModelPicker.value = false
  bookModelSearch.value = ''
  tempSelectedBookModel.value = null
}

function cancelBookModelSelection() {
  showBookModelPicker.value = false
  bookModelSearch.value = ''
  tempSelectedBookModel.value = null
}

const transactions = ref([
  {
    docNo: '15000000134',
    docRef: '51000000188-KEU',
    docDate: '30-11-2024',
    supplier: 'EMP',
    localAmount: 13971636,
    company: '1000',
    description: 'Pembelian barang',
    status: 'Paid',
  },
])

const filteredTransactions = computed(() => {
  return transactions.value.filter(
    (item) =>
      item.docNo.includes(searchQuery.value) ||
      item.docRef.includes(searchQuery.value) ||
      item.supplier.includes(searchQuery.value),
  )
})

const itemsPerPage = 10
const currentPage = ref(1)

const totalPages = computed(() => {
  return Math.ceil(filteredTransactions.value.length / itemsPerPage)
})

const paginatedTransactions = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredTransactions.value.slice(start, end)
})

function nextPage() {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

function formatAmount(value) {
  if (!value && value !== 0) return '-'
  return parseFloat(value).toLocaleString('id-ID')
}

function selectTransaction(item) {
  selectedTransaction.value = { ...item }
  showForm.value = true
}

function handleSave() {
  const index = transactions.value.findIndex((t) => t.docNo === selectedTransaction.value.docNo)
  if (index !== -1) {
    transactions.value[index] = { ...selectedTransaction.value }
  } else {
    transactions.value.push({ ...selectedTransaction.value })
  }
  showForm.value = false
}

function handleCancel() {
  selectedTransaction.value = {}
  showForm.value = false
}

function handlePayment() {
  showPaymentForm.value = !showPaymentForm.value
}
</script>

<style scoped>
table {
  border-collapse: collapse;
}

th,
td {
  text-align: center;
  vertical-align: middle;
}

th {
  background-color: #02a2dc;
  color: white;
  font-weight: bold;
  text-transform: none;
}

button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

th,
td {
  padding: 12px;
}

th {
  border: 1px solid #000;
}

td {
  border: 1px solid #e1e1e1;
}

th div {
  line-height: 1.2;
}

.btn-primary {
  background-color: #02a2dc;
  color: white;
  padding: 0.5rem 1.25rem;
  font-weight: 600;
  border-radius: 0.375rem;
}

.btn-outline {
  background-color: white;
  border: 1.5px solid #02a2dc;
  color: #02a2dc;
  padding: 0.5rem 1.25rem;
  font-weight: 600;
  border-radius: 0.375rem;
}
</style>

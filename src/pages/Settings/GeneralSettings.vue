<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <div class="max-w-6xl mx-auto space-y-6">
      <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
        <div>
          <h2 class="text-2xl font-bold text-gray-800">General Setting</h2>
          <p class="text-sm text-gray-500">Kelola konfigurasi utama aplikasi sesuai kebutuhan koperasi Anda.</p>
        </div>
        <div class="flex items-center gap-3">
          <button
            class="px-4 py-2 bg-[#02A2DC] text-white rounded-lg shadow-md hover:bg-[#0191b8] transition"
            type="button"
          >
            Save
          </button>
          <button
            class="px-4 py-2 border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-50 transition"
            type="button"
          >
            Cancel
          </button>
        </div>
      </div>

      <div class="bg-white rounded-xl shadow-md overflow-hidden">
        <div class="flex flex-col lg:flex-row">
          <aside class="lg:w-60 border-b lg:border-b-0 lg:border-r border-gray-200 bg-gray-50">
            <nav class="flex lg:flex-col overflow-x-auto lg:overflow-visible">
              <button
                v-for="section in sectionList"
                :key="section.id"
                type="button"
                @click="navigateToSection(section.id)"
                class="px-4 py-3 text-sm font-medium transition whitespace-nowrap"
                :class="[
                  activeSection === section.id
                    ? 'text-[#02A2DC] bg-white lg:rounded-none lg:border-l-4 lg:border-[#02A2DC] shadow-inner'
                    : 'text-gray-600 hover:text-[#02A2DC] hover:bg-gray-100',
                ]"
              >
                {{ section.label }}
              </button>
            </nav>
          </aside>

          <main class="flex-1 p-6 space-y-6">
            <!-- Currency & Currency Rate Section -->
            <template v-if="['currency', 'currency-rate'].includes(activeSection)">
              <div class="flex flex-wrap items-center gap-3">
                <button
                  v-for="tab in currencyTabs"
                  :key="tab.id"
                  type="button"
                  @click="setCurrencyTab(tab.id)"
                  class="px-4 py-2 rounded-lg text-sm font-semibold transition"
                  :class="[
                    activeCurrencyTab === tab.id
                      ? 'bg-[#02A2DC] text-white shadow'
                      : 'bg-gray-100 text-gray-600 hover:text-[#02A2DC]',
                  ]"
                >
                  {{ tab.label }}
                </button>
              </div>

              <div v-if="activeCurrencyTab === 'currency'" class="space-y-6">
                <section class="border border-gray-200 rounded-lg overflow-hidden">
                  <header class="px-4 py-3 bg-gray-50 border-b border-gray-200">
                    <h3 class="text-base font-semibold text-gray-700">Rate Type</h3>
                  </header>
                  <div class="p-4">
                    <div class="overflow-x-auto">
                      <table class="min-w-full text-sm">
                        <thead class="bg-[#02A2DC] text-white">
                          <tr>
                            <th class="px-4 py-3 text-left font-semibold">Rate Type</th>
                            <th class="px-4 py-3 text-left font-semibold">Description</th>
                            <th class="px-4 py-3 text-left font-semibold">Status</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="type in rateTypeData"
                            :key="type.id"
                            class="border-b last:border-0 border-gray-200"
                          >
                            <td class="px-4 py-3 text-gray-700 font-medium">{{ type.code }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ type.description }}</td>
                            <td class="px-4 py-3">
                              <span
                                class="px-2 py-1 text-xs font-semibold rounded-full"
                                :class="type.status === 'Active' ? 'bg-green-100 text-green-700' : 'bg-gray-100 text-gray-500'"
                              >
                                {{ type.status }}
                              </span>
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                  </div>
                </section>

                <section class="border border-gray-200 rounded-lg overflow-hidden">
                  <header class="px-4 py-3 bg-gray-50 border-b border-gray-200 flex items-center justify-between">
                    <h3 class="text-base font-semibold text-gray-700">Currency</h3>
                    <div class="flex gap-3">
                      <button
                        type="button"
                        class="px-3 py-1.5 border border-gray-300 rounded-md text-sm text-gray-600 hover:bg-gray-100 transition"
                      >
                        Add
                      </button>
                      <button
                        type="button"
                        class="px-3 py-1.5 border border-red-300 text-red-600 rounded-md text-sm hover:bg-red-50 transition"
                      >
                        Remove
                      </button>
                    </div>
                  </header>
                  <div class="p-4">
                    <div class="overflow-x-auto">
                      <table class="min-w-full text-sm">
                        <thead class="bg-[#02A2DC] text-white">
                          <tr>
                            <th class="px-4 py-3 text-left font-semibold">Currency</th>
                            <th class="px-4 py-3 text-left font-semibold">Symbol</th>
                            <th class="px-4 py-3 text-left font-semibold">Description</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="currency in currencyList"
                            :key="currency.id"
                            class="border-b last:border-0 border-gray-200"
                          >
                            <td class="px-4 py-3 text-gray-700 font-medium">{{ currency.code }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ currency.symbol }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ currency.description }}</td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                  </div>
                </section>
              </div>

              <div v-else class="space-y-6">
                <section class="border border-gray-200 rounded-lg overflow-hidden">
                  <header class="px-4 py-3 bg-gray-50 border-b border-gray-200 flex items-center justify-between">
                    <h3 class="text-base font-semibold text-gray-700">Currency Rate</h3>
                    <button
                      type="button"
                      class="px-3 py-1.5 border border-gray-300 rounded-md text-sm text-gray-600 hover:bg-gray-100 transition"
                    >
                      Add Rate
                    </button>
                  </header>
                  <div class="p-4">
                    <div class="overflow-x-auto">
                      <table class="min-w-full text-sm">
                        <thead class="bg-[#02A2DC] text-white">
                          <tr>
                            <th class="px-4 py-3 text-left font-semibold">Currency</th>
                            <th class="px-4 py-3 text-left font-semibold">Rate Type</th>
                            <th class="px-4 py-3 text-left font-semibold">Valid From</th>
                            <th class="px-4 py-3 text-left font-semibold">Valid To</th>
                            <th class="px-4 py-3 text-left font-semibold">Rate</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="rate in currencyRateData"
                            :key="rate.id"
                            class="border-b last:border-0 border-gray-200"
                          >
                            <td class="px-4 py-3 text-gray-700 font-medium">{{ rate.currency }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ rate.rateType }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ rate.validFrom }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ rate.validTo }}</td>
                            <td class="px-4 py-3 text-gray-600">{{ rate.rate }}</td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                  </div>
                </section>
              </div>
            </template>

            <!-- Company Section -->
            <template v-else-if="activeSection === 'company'">
              <section class="border border-gray-200 rounded-lg overflow-hidden">
                <header class="px-4 py-3 bg-gray-50 border-b border-gray-200">
                  <h3 class="text-base font-semibold text-gray-700">Informasi Perusahaan</h3>
                </header>
                <div class="p-6 space-y-6">
                  <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div>
                      <label class="block text-sm font-medium text-gray-600 mb-2">Nama Perusahaan</label>
                      <input
                        v-model="companyForm.name"
                        type="text"
                        class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                      />
                    </div>
                    <div>
                      <label class="block text-sm font-medium text-gray-600 mb-2">Nomor Telepon</label>
                      <input
                        v-model="companyForm.phone"
                        type="text"
                        class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                      />
                    </div>
                    <div>
                      <label class="block text-sm font-medium text-gray-600 mb-2">Alamat Email</label>
                      <input
                        v-model="companyForm.email"
                        type="email"
                        class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                      />
                    </div>
                    <div>
                      <label class="block text-sm font-medium text-gray-600 mb-2">NPWP</label>
                      <input
                        v-model="companyForm.taxId"
                        type="text"
                        class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                      />
                    </div>
                  </div>
                  <div>
                    <label class="block text-sm font-medium text-gray-600 mb-2">Alamat</label>
                    <textarea
                      v-model="companyForm.address"
                      rows="3"
                      class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                    ></textarea>
                  </div>
                </div>
              </section>
            </template>

            <!-- Document Section -->
            <template v-else-if="activeSection === 'document'">
              <section class="border border-gray-200 rounded-lg overflow-hidden">
                <header class="px-4 py-3 bg-gray-50 border-b border-gray-200">
                  <h3 class="text-base font-semibold text-gray-700">Pengaturan Dokumen</h3>
                </header>
                <div class="p-6 space-y-4">
                  <div
                    v-for="setting in documentSettings"
                    :key="setting.id"
                    class="flex items-start justify-between border border-gray-200 rounded-lg px-4 py-3"
                  >
                    <div>
                      <p class="text-sm font-semibold text-gray-700">{{ setting.name }}</p>
                      <p class="text-xs text-gray-500">{{ setting.description }}</p>
                    </div>
                    <label class="inline-flex items-center cursor-pointer">
                      <input
                        type="checkbox"
                        class="sr-only"
                        :checked="setting.enabled"
                        @change="toggleDocumentSetting(setting.id)"
                      />
                      <span
                        class="w-11 h-6 flex items-center bg-gray-200 rounded-full p-1 duration-300 ease-in-out"
                        :class="setting.enabled ? 'bg-[#02A2DC]' : 'bg-gray-200'"
                      >
                        <span
                          class="bg-white w-4 h-4 rounded-full shadow-md transform duration-300"
                          :class="setting.enabled ? 'translate-x-5' : 'translate-x-0'"
                        ></span>
                      </span>
                    </label>
                  </div>
                </div>
              </section>
            </template>

            <!-- Notification Section -->
            <template v-else-if="activeSection === 'notification'">
              <section class="border border-gray-200 rounded-lg overflow-hidden">
                <header class="px-4 py-3 bg-gray-50 border-b border-gray-200">
                  <h3 class="text-base font-semibold text-gray-700">Preferensi Notifikasi</h3>
                </header>
                <div class="p-6 space-y-5">
                  <div
                    v-for="channel in notificationPreferences"
                    :key="channel.id"
                    class="flex items-start gap-4 border border-gray-200 rounded-lg px-4 py-3"
                  >
                    <input
                      type="checkbox"
                      class="mt-1 w-4 h-4 text-[#02A2DC] border-gray-300 rounded focus:ring-[#02A2DC]"
                      :checked="channel.enabled"
                      @change="toggleNotification(channel.id)"
                    />
                    <div>
                      <p class="text-sm font-semibold text-gray-700">{{ channel.label }}</p>
                      <p class="text-xs text-gray-500">{{ channel.description }}</p>
                    </div>
                  </div>
                </div>
              </section>
            </template>

            <!-- Language Section -->
            <template v-else-if="activeSection === 'language'">
              <section class="border border-gray-200 rounded-lg overflow-hidden">
                <header class="px-4 py-3 bg-gray-50 border-b border-gray-200">
                  <h3 class="text-base font-semibold text-gray-700">Bahasa Aplikasi</h3>
                </header>
                <div class="p-6 space-y-6">
                  <div>
                    <label class="block text-sm font-medium text-gray-600 mb-2">Bahasa Utama</label>
                    <select
                      v-model="selectedLanguage"
                      class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-[#02A2DC]"
                    >
                      <option v-for="language in availableLanguages" :key="language" :value="language">
                        {{ language }}
                      </option>
                    </select>
                  </div>
                  <div>
                    <p class="text-sm font-semibold text-gray-700 mb-3">Bahasa Tambahan</p>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                      <label
                        v-for="language in additionalLanguages"
                        :key="language.id"
                        class="flex items-center gap-3 border border-gray-200 rounded-lg px-3 py-2"
                      >
                        <input
                          type="checkbox"
                          class="w-4 h-4 text-[#02A2DC] border-gray-300 rounded focus:ring-[#02A2DC]"
                          :checked="language.enabled"
                          @change="toggleAdditionalLanguage(language.id)"
                        />
                        <span class="text-sm text-gray-600">{{ language.label }}</span>
                      </label>
                    </div>
                  </div>
                </div>
              </section>
            </template>

            <!-- Reference Section -->
            <template v-else-if="activeSection === 'reference'">
              <section class="border border-gray-200 rounded-lg overflow-hidden">
                <header class="px-4 py-3 bg-gray-50 border-b border-gray-200 flex items-center justify-between">
                  <h3 class="text-base font-semibold text-gray-700">Referensi</h3>
                  <button
                    type="button"
                    class="px-3 py-1.5 border border-gray-300 rounded-md text-sm text-gray-600 hover:bg-gray-100 transition"
                  >
                    Add Reference
                  </button>
                </header>
                <div class="p-4">
                  <div class="overflow-x-auto">
                    <table class="min-w-full text-sm">
                      <thead class="bg-[#02A2DC] text-white">
                        <tr>
                          <th class="px-4 py-3 text-left font-semibold">Code</th>
                          <th class="px-4 py-3 text-left font-semibold">Description</th>
                          <th class="px-4 py-3 text-left font-semibold">Status</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr
                          v-for="reference in referenceData"
                          :key="reference.id"
                          class="border-b last:border-0 border-gray-200"
                        >
                          <td class="px-4 py-3 text-gray-700 font-medium">{{ reference.code }}</td>
                          <td class="px-4 py-3 text-gray-600">{{ reference.description }}</td>
                          <td class="px-4 py-3">
                            <span
                              class="px-2 py-1 text-xs font-semibold rounded-full"
                              :class="reference.status === 'Active'
                                ? 'bg-green-100 text-green-700'
                                : 'bg-gray-100 text-gray-500'"
                            >
                              {{ reference.status }}
                            </span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>
              </section>
            </template>

            <!-- Fallback Section -->
            <template v-else>
              <div class="text-center py-12 text-gray-500">
                Pengaturan belum tersedia untuk bagian ini.
              </div>
            </template>
          </main>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { settingsSections } from '@/constants/settingsSections'

const router = useRouter()
const route = useRoute()

const sectionList = settingsSections

const validSections = computed(() => sectionList.map((section) => section.id))

const getInitialSection = () => {
  const sectionParam = route.params.section
  if (typeof sectionParam === 'string' && validSections.value.includes(sectionParam)) {
    return sectionParam
  }
  return 'currency'
}

const activeSection = ref(getInitialSection())

watch(
  () => route.params.section,
  (newSection) => {
    if (typeof newSection === 'string' && validSections.value.includes(newSection)) {
      activeSection.value = newSection
    } else {
      activeSection.value = 'currency'
    }
  },
  { immediate: true },
)

const navigateToSection = (sectionId) => {
  if (validSections.value.includes(sectionId)) {
    router.push({ path: `/settings/${sectionId}` })
  }
}

const currencyTabs = [
  { id: 'currency', label: 'Currency' },
  { id: 'currency-rate', label: 'Currency Rate' },
]

const activeCurrencyTab = ref(activeSection.value === 'currency-rate' ? 'currency-rate' : 'currency')

watch(
  activeSection,
  (section) => {
    if (section === 'currency-rate') {
      activeCurrencyTab.value = 'currency-rate'
    } else if (section === 'currency') {
      activeCurrencyTab.value = 'currency'
    }
  },
  { immediate: true },
)

const setCurrencyTab = (tabId) => {
  if (tabId === 'currency-rate') {
    navigateToSection('currency-rate')
  } else {
    navigateToSection('currency')
  }
}

const rateTypeData = [
  { id: 1, code: 'S', description: 'Tax Rate', status: 'Active' },
  { id: 2, code: 'M', description: 'Market Rate', status: 'Active' },
]

const currencyList = [
  { id: 1, code: 'IDR', symbol: 'Rp', description: 'Rupiah' },
  { id: 2, code: 'USD', symbol: '$', description: 'US Dollar' },
  { id: 3, code: 'JPY', symbol: '¥', description: 'Yen' },
  { id: 4, code: 'SGD', symbol: '$', description: 'Singapore Dollar' },
  { id: 5, code: 'EUR', symbol: '€', description: 'Euro' },
]

const currencyRateData = [
  { id: 1, currency: 'USD', rateType: 'Market Rate', validFrom: '01 Jan 2024', validTo: '31 Jan 2024', rate: '15,200' },
  { id: 2, currency: 'JPY', rateType: 'Market Rate', validFrom: '01 Jan 2024', validTo: '31 Jan 2024', rate: '105' },
  { id: 3, currency: 'EUR', rateType: 'Tax Rate', validFrom: '01 Jan 2024', validTo: '31 Jan 2024', rate: '16,400' },
]

const companyForm = ref({
  name: 'Koperasi Sejahtera Bersama',
  phone: '+62 812 3456 7890',
  email: 'info@koperasi.co.id',
  taxId: '01.234.567.8-123.000',
  address: 'Jl. Merdeka No. 45, Jakarta Pusat, DKI Jakarta',
})

const documentSettings = ref([
  {
    id: 'autoNumber',
    name: 'Penomoran Otomatis',
    description: 'Aktifkan penomoran otomatis untuk dokumen transaksi baru.',
    enabled: true,
  },
  {
    id: 'approvalWorkflow',
    name: 'Workflow Persetujuan',
    description: 'Minta persetujuan manager sebelum dokumen diposting.',
    enabled: false,
  },
  {
    id: 'attachmentRequired',
    name: 'Lampiran Wajib',
    description: 'Wajibkan lampiran dokumen pendukung untuk transaksi tertentu.',
    enabled: true,
  },
])

const toggleDocumentSetting = (settingId) => {
  documentSettings.value = documentSettings.value.map((setting) =>
    setting.id === settingId ? { ...setting, enabled: !setting.enabled } : setting,
  )
}

const notificationPreferences = ref([
  {
    id: 'email',
    label: 'Email Notification',
    description: 'Kirimkan update transaksi dan persetujuan melalui email.',
    enabled: true,
  },
  {
    id: 'sms',
    label: 'SMS Notification',
    description: 'Terima pemberitahuan penting melalui SMS.',
    enabled: false,
  },
  {
    id: 'inApp',
    label: 'In-App Notification',
    description: 'Tampilkan notifikasi langsung di dashboard aplikasi.',
    enabled: true,
  },
])

const toggleNotification = (channelId) => {
  notificationPreferences.value = notificationPreferences.value.map((channel) =>
    channel.id === channelId ? { ...channel, enabled: !channel.enabled } : channel,
  )
}

const availableLanguages = ['Bahasa Indonesia', 'English', '日本語 (Japanese)']
const selectedLanguage = ref('Bahasa Indonesia')

const additionalLanguages = ref([
  { id: 'english', label: 'English', enabled: true },
  { id: 'mandarin', label: '中文 (Mandarin)', enabled: false },
  { id: 'japanese', label: '日本語 (Japanese)', enabled: true },
  { id: 'arabic', label: 'العربية (Arabic)', enabled: false },
])

const toggleAdditionalLanguage = (languageId) => {
  additionalLanguages.value = additionalLanguages.value.map((language) =>
    language.id === languageId ? { ...language, enabled: !language.enabled } : language,
  )
}

const referenceData = [
  { id: 1, code: 'REF-001', description: 'Kode Referensi Penjualan', status: 'Active' },
  { id: 2, code: 'REF-002', description: 'Kode Referensi Pembelian', status: 'Active' },
  { id: 3, code: 'REF-003', description: 'Kode Referensi Simpanan', status: 'Inactive' },
]
</script>

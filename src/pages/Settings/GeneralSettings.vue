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
            @click="handleSaveSettings"
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
        <div
          v-if="showSaveSuccess"
          class="mx-6 mt-6 rounded-lg border border-green-200 bg-green-50 px-4 py-3 text-sm text-green-700"
        >
          Perubahan pengaturan berhasil disimpan.
        </div>
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
                  <header class="flex flex-wrap items-center justify-between gap-3 border-b border-gray-200 bg-gray-50 px-4 py-3">
                    <h3 class="text-base font-semibold text-gray-700">Rate Type</h3>
                    <div class="flex gap-3">
                      <button
                        type="button"
                        class="rounded-md border border-gray-300 px-3 py-1.5 text-sm font-medium text-gray-600 transition hover:bg-gray-100"
                        @click="addRateType"
                      >
                        Add
                      </button>
                      <button
                        type="button"
                        class="rounded-md border border-red-300 px-3 py-1.5 text-sm font-medium text-red-600 transition hover:bg-red-50 disabled:cursor-not-allowed disabled:opacity-50"
                        :disabled="!selectedRateTypeIds.length"
                        @click="removeSelectedRateTypes"
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
                            <th class="w-12 px-3 py-3 text-left font-semibold">
                              <input
                                type="checkbox"
                                class="h-4 w-4 rounded border-white/70 text-[#015a78] focus:ring-[#02A2DC]"
                                :checked="areAllRateTypesSelected"
                                @change="toggleAllRateTypes($event.target.checked)"
                              />
                            </th>
                            <th class="px-4 py-3 text-left font-semibold">Rate Type</th>
                            <th class="px-4 py-3 text-left font-semibold">Description</th>
                            <th class="px-4 py-3 text-left font-semibold">Status</th>
                            <th class="px-4 py-3 text-left font-semibold">Actions</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="type in rateTypes"
                            :key="type.id"
                            :class="[
                              'border-b border-gray-200 last:border-0',
                              isRateTypeSelected(type.id) ? 'bg-[#02A2DC]/5' : '',
                            ]"
                          >
                            <td class="px-3 py-3">
                              <input
                                type="checkbox"
                                class="h-4 w-4 rounded border-gray-300 text-[#015a78] focus:ring-[#02A2DC]"
                                :checked="isRateTypeSelected(type.id)"
                                @change="toggleRateTypeSelection(type.id, $event.target.checked)"
                              />
                            </td>
                            <td class="px-4 py-3 text-gray-700">
                              <template v-if="isEditingRateType(type.id)">
                                <input
                                  v-model="rateTypeDraft.code"
                                  type="text"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                  placeholder="Kode"
                                />
                              </template>
                              <template v-else>
                                <span class="font-medium">{{ type.code }}</span>
                              </template>
                            </td>
                            <td class="px-4 py-3 text-gray-600">
                              <template v-if="isEditingRateType(type.id)">
                                <input
                                  v-model="rateTypeDraft.description"
                                  type="text"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                  placeholder="Deskripsi"
                                />
                              </template>
                              <template v-else>
                                {{ type.description }}
                              </template>
                            </td>
                            <td class="px-4 py-3 text-gray-600">
                              <template v-if="isEditingRateType(type.id)">
                                <select
                                  v-model="rateTypeDraft.status"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                >
                                  <option value="Active">Active</option>
                                  <option value="Inactive">Inactive</option>
                                </select>
                              </template>
                              <template v-else>
                                <span
                                  class="rounded-full px-2 py-1 text-xs font-semibold"
                                  :class="type.status === 'Active'
                                    ? 'bg-green-100 text-green-700'
                                    : 'bg-gray-100 text-gray-500'"
                                >
                                  {{ type.status }}
                                </span>
                              </template>
                            </td>
                            <td class="px-4 py-3">
                              <div class="flex flex-wrap items-center gap-2">
                                <template v-if="isEditingRateType(type.id)">
                                  <button
                                    type="button"
                                    class="rounded-md bg-[#02A2DC] px-3 py-1.5 text-xs font-semibold text-white shadow-sm transition hover:bg-[#0191b8]"
                                    @click="saveRateTypeEdit(type.id)"
                                  >
                                    Save
                                  </button>
                                  <button
                                    type="button"
                                    class="rounded-md border border-gray-300 px-3 py-1.5 text-xs font-semibold text-gray-600 transition hover:bg-gray-100"
                                    @click="cancelRateTypeEdit(type.id)"
                                  >
                                    Cancel
                                  </button>
                                </template>
                                <template v-else>
                                  <button
                                    type="button"
                                    class="rounded-md border border-gray-300 px-3 py-1.5 text-xs font-semibold text-gray-600 transition hover:bg-gray-100"
                                    @click="startRateTypeEdit(type)"
                                  >
                                    Edit
                                  </button>
                                  <button
                                    type="button"
                                    class="rounded-md border border-red-300 px-3 py-1.5 text-xs font-semibold text-red-600 transition hover:bg-red-50"
                                    @click="removeRateType(type.id)"
                                  >
                                    Delete
                                  </button>
                                </template>
                              </div>
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                    <div class="mt-4 flex flex-wrap items-center justify-between gap-3 text-xs text-gray-500">
                      <span>Records: {{ rateTypes.length }}</span>
                      <span>Page: 1</span>
                    </div>
                  </div>
                </section>

                <section class="border border-gray-200 rounded-lg overflow-hidden">
                  <header class="flex flex-wrap items-center justify-between gap-3 border-b border-gray-200 bg-gray-50 px-4 py-3">
                    <h3 class="text-base font-semibold text-gray-700">Currency</h3>
                    <div class="flex gap-3">
                      <button
                        type="button"
                        class="rounded-md border border-gray-300 px-3 py-1.5 text-sm font-medium text-gray-600 transition hover:bg-gray-100"
                        @click="addCurrency"
                      >
                        Add
                      </button>
                      <button
                        type="button"
                        class="rounded-md border border-red-300 px-3 py-1.5 text-sm font-medium text-red-600 transition hover:bg-red-50 disabled:cursor-not-allowed disabled:opacity-50"
                        :disabled="!selectedCurrencyIds.length"
                        @click="removeSelectedCurrencies"
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
                            <th class="w-12 px-3 py-3 text-left font-semibold">
                              <input
                                type="checkbox"
                                class="h-4 w-4 rounded border-white/70 text-[#015a78] focus:ring-[#02A2DC]"
                                :checked="areAllCurrenciesSelected"
                                @change="toggleAllCurrencies($event.target.checked)"
                              />
                            </th>
                            <th class="px-4 py-3 text-left font-semibold">Currency</th>
                            <th class="px-4 py-3 text-left font-semibold">Symbol</th>
                            <th class="px-4 py-3 text-left font-semibold">Description</th>
                            <th class="px-4 py-3 text-left font-semibold">Actions</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="currency in currencies"
                            :key="currency.id"
                            :class="[
                              'border-b border-gray-200 last:border-0',
                              isCurrencySelected(currency.id) ? 'bg-[#02A2DC]/5' : '',
                            ]"
                          >
                            <td class="px-3 py-3">
                              <input
                                type="checkbox"
                                class="h-4 w-4 rounded border-gray-300 text-[#015a78] focus:ring-[#02A2DC]"
                                :checked="isCurrencySelected(currency.id)"
                                @change="toggleCurrencySelection(currency.id, $event.target.checked)"
                              />
                            </td>
                            <td class="px-4 py-3 text-gray-700">
                              <template v-if="isEditingCurrency(currency.id)">
                                <input
                                  v-model="currencyDraft.code"
                                  type="text"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                  placeholder="Kode"
                                />
                              </template>
                              <template v-else>
                                <span class="font-medium">{{ currency.code }}</span>
                              </template>
                            </td>
                            <td class="px-4 py-3 text-gray-600">
                              <template v-if="isEditingCurrency(currency.id)">
                                <input
                                  v-model="currencyDraft.symbol"
                                  type="text"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                  placeholder="Simbol"
                                />
                              </template>
                              <template v-else>
                                {{ currency.symbol }}
                              </template>
                            </td>
                            <td class="px-4 py-3 text-gray-600">
                              <template v-if="isEditingCurrency(currency.id)">
                                <input
                                  v-model="currencyDraft.description"
                                  type="text"
                                  class="w-full rounded-md border border-gray-300 px-3 py-1.5 text-sm focus:border-[#02A2DC] focus:outline-none focus:ring-1 focus:ring-[#02A2DC]"
                                  placeholder="Deskripsi"
                                />
                              </template>
                              <template v-else>
                                {{ currency.description }}
                              </template>
                            </td>
                            <td class="px-4 py-3">
                              <div class="flex flex-wrap items-center gap-2">
                                <template v-if="isEditingCurrency(currency.id)">
                                  <button
                                    type="button"
                                    class="rounded-md bg-[#02A2DC] px-3 py-1.5 text-xs font-semibold text-white shadow-sm transition hover:bg-[#0191b8]"
                                    @click="saveCurrencyEdit(currency.id)"
                                  >
                                    Save
                                  </button>
                                  <button
                                    type="button"
                                    class="rounded-md border border-gray-300 px-3 py-1.5 text-xs font-semibold text-gray-600 transition hover:bg-gray-100"
                                    @click="cancelCurrencyEdit(currency.id)"
                                  >
                                    Cancel
                                  </button>
                                </template>
                                <template v-else>
                                  <button
                                    type="button"
                                    class="rounded-md border border-gray-300 px-3 py-1.5 text-xs font-semibold text-gray-600 transition hover:bg-gray-100"
                                    @click="startCurrencyEdit(currency)"
                                  >
                                    Edit
                                  </button>
                                  <button
                                    type="button"
                                    class="rounded-md border border-red-300 px-3 py-1.5 text-xs font-semibold text-red-600 transition hover:bg-red-50"
                                    @click="removeCurrency(currency.id)"
                                  >
                                    Delete
                                  </button>
                                </template>
                              </div>
                            </td>
                          </tr>
                        </tbody>
                      </table>
                    </div>
                    <div class="mt-4 flex flex-wrap items-center justify-between gap-3 text-xs text-gray-500">
                      <span>Records: {{ currencies.length }}</span>
                      <span>Page: 1</span>
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
import { computed, onBeforeUnmount, ref, watch } from 'vue'
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
  activeCurrencyTab.value = tabId
  if (tabId === 'currency-rate') {
    navigateToSection('currency-rate')
  } else {
    navigateToSection('currency')
  }
}

const showSaveSuccess = ref(false)
const saveMessageTimeout = ref()

const handleSaveSettings = () => {
  showSaveSuccess.value = true
  if (saveMessageTimeout.value) {
    clearTimeout(saveMessageTimeout.value)
  }
  saveMessageTimeout.value = setTimeout(() => {
    showSaveSuccess.value = false
    saveMessageTimeout.value = undefined
  }, 3000)
}

onBeforeUnmount(() => {
  if (saveMessageTimeout.value) {
    clearTimeout(saveMessageTimeout.value)
  }
})

const rateTypes = ref([
  { id: 1, code: 'S', description: 'Tax Rate', status: 'Active' },
  { id: 2, code: 'M', description: 'Market Rate', status: 'Active' },
])

const rateTypeIdCounter = ref(rateTypes.value.length + 1)
const selectedRateTypeIds = ref([])
const newRateTypeIds = ref([])
const editingRateTypeId = ref(null)
const rateTypeDraft = ref({})

const areAllRateTypesSelected = computed(
  () => rateTypes.value.length > 0 && selectedRateTypeIds.value.length === rateTypes.value.length,
)

const isRateTypeSelected = (id) => selectedRateTypeIds.value.includes(id)

const toggleRateTypeSelection = (id, checked) => {
  if (checked) {
    if (!isRateTypeSelected(id)) {
      selectedRateTypeIds.value = [...selectedRateTypeIds.value, id]
    }
  } else {
    selectedRateTypeIds.value = selectedRateTypeIds.value.filter((selectedId) => selectedId !== id)
  }
}

const toggleAllRateTypes = (checked) => {
  if (checked) {
    selectedRateTypeIds.value = rateTypes.value.map((type) => type.id)
  } else {
    selectedRateTypeIds.value = []
  }
}

const isNewRateType = (id) => newRateTypeIds.value.includes(id)

const addRateType = () => {
  if (editingRateTypeId.value !== null) {
    cancelRateTypeEdit(editingRateTypeId.value)
  }

  const newId = rateTypeIdCounter.value++
  const newRateType = { id: newId, code: '', description: '', status: 'Active' }
  rateTypes.value = [...rateTypes.value, newRateType]
  newRateTypeIds.value = [...newRateTypeIds.value, newId]
  selectedRateTypeIds.value = [...selectedRateTypeIds.value, newId]
  editingRateTypeId.value = newId
  rateTypeDraft.value = { ...newRateType }
}

const startRateTypeEdit = (rateType) => {
  editingRateTypeId.value = rateType.id
  rateTypeDraft.value = { ...rateType }
}

const saveRateTypeEdit = (id) => {
  const index = rateTypes.value.findIndex((item) => item.id === id)
  if (index !== -1) {
    rateTypes.value.splice(index, 1, { ...rateTypeDraft.value, id })
  }
  newRateTypeIds.value = newRateTypeIds.value.filter((newId) => newId !== id)
  editingRateTypeId.value = null
  rateTypeDraft.value = {}
}

const cancelRateTypeEdit = (id) => {
  if (isNewRateType(id)) {
    rateTypes.value = rateTypes.value.filter((type) => type.id !== id)
    newRateTypeIds.value = newRateTypeIds.value.filter((newId) => newId !== id)
    selectedRateTypeIds.value = selectedRateTypeIds.value.filter((selectedId) => selectedId !== id)
  }
  editingRateTypeId.value = null
  rateTypeDraft.value = {}
}

const removeRateType = (id) => {
  rateTypes.value = rateTypes.value.filter((type) => type.id !== id)
  newRateTypeIds.value = newRateTypeIds.value.filter((newId) => newId !== id)
  selectedRateTypeIds.value = selectedRateTypeIds.value.filter((selectedId) => selectedId !== id)
  if (editingRateTypeId.value === id) {
    editingRateTypeId.value = null
    rateTypeDraft.value = {}
  }
}

const removeSelectedRateTypes = () => {
  if (!selectedRateTypeIds.value.length) {
    return
  }
  const idsToRemove = new Set(selectedRateTypeIds.value)
  rateTypes.value = rateTypes.value.filter((type) => !idsToRemove.has(type.id))
  newRateTypeIds.value = newRateTypeIds.value.filter((newId) => !idsToRemove.has(newId))
  if (editingRateTypeId.value && idsToRemove.has(editingRateTypeId.value)) {
    editingRateTypeId.value = null
    rateTypeDraft.value = {}
  }
  selectedRateTypeIds.value = []
}

const isEditingRateType = (id) => editingRateTypeId.value === id

const currencies = ref([
  { id: 1, code: 'IDR', symbol: 'Rp', description: 'Rupiah' },
  { id: 2, code: 'USD', symbol: '$', description: 'US Dollar' },
  { id: 3, code: 'JPY', symbol: '¥', description: 'Yen' },
  { id: 4, code: 'SGD', symbol: '$', description: 'Singapore Dollar' },
  { id: 5, code: 'EUR', symbol: '€', description: 'Euro' },
])

const currencyIdCounter = ref(currencies.value.length + 1)
const selectedCurrencyIds = ref([])
const newCurrencyIds = ref([])
const editingCurrencyId = ref(null)
const currencyDraft = ref({})

const areAllCurrenciesSelected = computed(
  () => currencies.value.length > 0 && selectedCurrencyIds.value.length === currencies.value.length,
)

const isCurrencySelected = (id) => selectedCurrencyIds.value.includes(id)

const toggleCurrencySelection = (id, checked) => {
  if (checked) {
    if (!isCurrencySelected(id)) {
      selectedCurrencyIds.value = [...selectedCurrencyIds.value, id]
    }
  } else {
    selectedCurrencyIds.value = selectedCurrencyIds.value.filter((selectedId) => selectedId !== id)
  }
}

const toggleAllCurrencies = (checked) => {
  if (checked) {
    selectedCurrencyIds.value = currencies.value.map((currency) => currency.id)
  } else {
    selectedCurrencyIds.value = []
  }
}

const isNewCurrency = (id) => newCurrencyIds.value.includes(id)

const addCurrency = () => {
  if (editingCurrencyId.value !== null) {
    cancelCurrencyEdit(editingCurrencyId.value)
  }

  const newId = currencyIdCounter.value++
  const newCurrency = { id: newId, code: '', symbol: '', description: '' }
  currencies.value = [...currencies.value, newCurrency]
  newCurrencyIds.value = [...newCurrencyIds.value, newId]
  selectedCurrencyIds.value = [...selectedCurrencyIds.value, newId]
  editingCurrencyId.value = newId
  currencyDraft.value = { ...newCurrency }
}

const startCurrencyEdit = (currency) => {
  editingCurrencyId.value = currency.id
  currencyDraft.value = { ...currency }
}

const saveCurrencyEdit = (id) => {
  const index = currencies.value.findIndex((item) => item.id === id)
  if (index !== -1) {
    currencies.value.splice(index, 1, { ...currencyDraft.value, id })
  }
  newCurrencyIds.value = newCurrencyIds.value.filter((newId) => newId !== id)
  editingCurrencyId.value = null
  currencyDraft.value = {}
}

const cancelCurrencyEdit = (id) => {
  if (isNewCurrency(id)) {
    currencies.value = currencies.value.filter((currency) => currency.id !== id)
    newCurrencyIds.value = newCurrencyIds.value.filter((newId) => newId !== id)
    selectedCurrencyIds.value = selectedCurrencyIds.value.filter((selectedId) => selectedId !== id)
  }
  editingCurrencyId.value = null
  currencyDraft.value = {}
}

const removeCurrency = (id) => {
  currencies.value = currencies.value.filter((currency) => currency.id !== id)
  newCurrencyIds.value = newCurrencyIds.value.filter((newId) => newId !== id)
  selectedCurrencyIds.value = selectedCurrencyIds.value.filter((selectedId) => selectedId !== id)
  if (editingCurrencyId.value === id) {
    editingCurrencyId.value = null
    currencyDraft.value = {}
  }
}

const removeSelectedCurrencies = () => {
  if (!selectedCurrencyIds.value.length) {
    return
  }
  const idsToRemove = new Set(selectedCurrencyIds.value)
  currencies.value = currencies.value.filter((currency) => !idsToRemove.has(currency.id))
  newCurrencyIds.value = newCurrencyIds.value.filter((newId) => !idsToRemove.has(newId))
  if (editingCurrencyId.value && idsToRemove.has(editingCurrencyId.value)) {
    editingCurrencyId.value = null
    currencyDraft.value = {}
  }
  selectedCurrencyIds.value = []
}

const isEditingCurrency = (id) => editingCurrencyId.value === id

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

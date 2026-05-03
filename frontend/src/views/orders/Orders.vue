<script setup>
import { ref, computed, onMounted } from 'vue'
import api from '../../services/api'

// State
const orders = ref([])
const customers = ref([])
const products = ref([])
const showModal = ref(false)
const loading = ref(false)
const isFetching = ref(true)

const selectedCustomer = ref('')
const items = ref([])

// Fetch data
const fetchData = async () => {
  isFetching.value = true
  try {
    const [o, c, p] = await Promise.all([
      api.get('/orders'),
      api.get('/customers'),
      api.get('/products')
    ])

    orders.value = o.data.data || o.data
    customers.value = c.data.data || c.data
    products.value = p.data.data || p.data
  } finally {
    isFetching.value = false
  }
}

// Order Logic
const openModal = () => {
  selectedCustomer.value = ''
  items.value = [{ product_id: '', quantity: 1 }]
  showModal.value = true
}

const addItem = () => {
  items.value.push({ product_id: '', quantity: 1 })
}

const removeItem = (index) => {
  if (items.value.length > 1) items.value.splice(index, 1)
}

const getProduct = (id) => products.value.find(p => p.id === id)

const isDuplicate = (id, index) => {
  if (!id) return false
  return items.value.some((item, i) => item.product_id === id && i !== index)
}

const formatCurrency = (val) => {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(val ?? 0)
}

// validation
const isInvalid = computed(() => {
  if (!selectedCustomer.value) return true
  if (items.value.length === 0) return true

  return items.value.some((item, index) => {
    if (!item.product_id) return true
    if (!Number.isFinite(Number(item.quantity))) return true
    if (item.quantity <= 0) return true
    if (isDuplicate(item.product_id, index)) return true

    const product = getProduct(item.product_id)
    if (!product) return true

    // stock validation
    if (item.quantity > product.quantity) return true
    return false
  })
})

// Totals
const total = computed(() => {
  return items.value.reduce((sum, item) => {
    const product = getProduct(item.product_id)
    const line = product ? product.price * item.quantity : 0
    return sum + line
  }, 0)
})

const submit = async () => {
  if (!selectedCustomer.value || items.value.some(i => !i.product_id)) {
    alert('Please select a customer and products for all lines.')
    return
  }

  loading.value = true
  try {
    await api.post('/orders', {
      customer_id: selectedCustomer.value,
      items: items.value
    })

    showModal.value = false
    await fetchData()
  } catch (err) {
    alert(err.response?.data?.message || 'Error creating order')
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
</script>

<template>
  <div class="max-w-6xl mx-auto p-4 sm:p-8">
    <!-- Header -->
    <div class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between mb-8">
      <div>
        <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-blue-50 text-blue-700 border border-blue-100">
          <span class="inline-block w-2 h-2 rounded-full bg-blue-600"></span>
          <span class="text-xs font-extrabold uppercase tracking-wider">Dashboard</span>
        </div>

        <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-900 tracking-tight mt-3">
          Sales Orders
        </h1>
        <p class="text-gray-500 mt-2">
          Review transaction history and generate new invoices.
        </p>
      </div>

      <div class="flex items-center gap-3">
        <div v-if="isFetching"
          class="hidden sm:flex items-center gap-2 px-4 py-2 rounded-xl bg-gray-50 border border-gray-100">
          <span class="w-2.5 h-2.5 rounded-full bg-gray-400 animate-pulse"></span>
          <span class="text-sm font-semibold text-gray-600">Loading…</span>
        </div>

        <button
          @click="openModal"
          class="inline-flex items-center justify-center px-6 py-3 bg-blue-600 text-white font-bold rounded-xl hover:bg-blue-700 transition-all shadow-lg shadow-blue-100 active:scale-[0.99]"
        >
          <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
          </svg>
          New Order
        </button>
      </div>
    </div>

    <!-- Stats -->
    <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 sm:gap-6 mb-8">
      <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
        <div class="flex items-center justify-between">
          <p class="text-sm font-medium text-gray-500 uppercase tracking-wider">Total Orders</p>
          <div class="w-10 h-10 rounded-xl bg-blue-50 flex items-center justify-center border border-blue-100">
            <svg class="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5a3 3 0 006 0" />
            </svg>
          </div>
        </div>
        <p class="text-2xl font-extrabold text-gray-900 mt-3">{{ orders.length }}</p>
      </div>

      <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm">
        <div class="flex items-center justify-between">
          <p class="text-sm font-medium text-gray-500 uppercase tracking-wider">Active Customers</p>
          <div class="w-10 h-10 rounded-xl bg-emerald-50 flex items-center justify-center border border-emerald-100">
            <svg class="w-5 h-5 text-emerald-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5V4H2v16h5" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 20v-4h10v4" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 10h10" />
            </svg>
          </div>
        </div>
        <p class="text-2xl font-extrabold text-gray-900 mt-3">{{ customers.length }}</p>
      </div>

      <div class="bg-gradient-to-br from-gray-50 to-white p-6 rounded-2xl border border-gray-100 shadow-sm">
        <div class="flex items-center justify-between">
          <p class="text-sm font-medium text-gray-500 uppercase tracking-wider">Catalog Items</p>
          <div class="w-10 h-10 rounded-xl bg-gray-50 flex items-center justify-center border border-gray-100">
            <svg class="w-5 h-5 text-gray-700" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 10c0 6-9 13-9 13S3 16 3 10a9 9 0 0118 0z" />
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10l4-2" />
            </svg>
          </div>
        </div>
        <p class="text-2xl font-extrabold text-gray-900 mt-3">{{ products.length }}</p>
      </div>
    </div>

    <!-- Table Card -->
    <div class="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
      <div class="flex items-center justify-between px-6 py-4 border-b border-gray-100 bg-gray-50/40">
        <div>
          <p class="text-sm font-extrabold text-gray-900">Order List</p>
          <p class="text-xs text-gray-500 mt-1">Latest transactions and their totals</p>
        </div>

        <div class="hidden md:flex items-center gap-2 text-xs font-bold text-gray-500">
          <span class="px-3 py-1 rounded-full bg-white border border-gray-200">Auto-formatted</span>
          <span class="px-3 py-1 rounded-full bg-white border border-gray-200">Responsive</span>
        </div>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left min-w-[820px]">
          <thead class="bg-gray-50 border-b border-gray-200">
            <tr>
              <th class="px-6 py-4 text-xs font-extrabold text-gray-500 uppercase tracking-widest">Order ID</th>
              <th class="px-6 py-4 text-xs font-extrabold text-gray-500 uppercase tracking-widest">Customer</th>
              <th class="px-6 py-4 text-xs font-extrabold text-gray-500 uppercase tracking-widest">Amount</th>
              <th class="px-6 py-4 text-xs font-extrabold text-gray-500 uppercase tracking-widest">Date</th>
              <th class="px-6 py-4 text-xs font-extrabold text-gray-500 uppercase tracking-widest text-right">Status</th>
            </tr>
          </thead>

          <tbody class="divide-y divide-gray-100">
            <tr v-if="isFetching">
              <td colspan="5" class="px-6 py-12 text-center text-gray-400">
                Loading orders...
              </td>
            </tr>

            <tr
              v-for="o in orders"
              :key="o.id"
              class="hover:bg-gray-50 transition-colors"
            >
              <td class="px-6 py-4 font-mono text-sm text-blue-700 font-extrabold">#ORD-{{ o.id }}</td>

              <td class="px-6 py-4">
                <div class="font-extrabold text-gray-900">{{ o.customer?.name }}</div>
                <div class="text-xs text-gray-500">{{ o.customer?.email }}</div>
              </td>

              <td class="px-6 py-4 font-extrabold text-gray-900">
                {{ formatCurrency(o.total) }}
              </td>

              <td class="px-6 py-4 text-sm text-gray-600">
                {{ o.created_at ? new Date(o.created_at).toLocaleDateString() : '—' }}
              </td>

              <td class="px-6 py-4 text-right">
                <span class="px-3 py-1 inline-flex items-center rounded-full text-xs font-extrabold bg-emerald-100 text-emerald-700 border border-emerald-200">
                  <span class="w-2 h-2 rounded-full bg-emerald-600 mr-2"></span>
                  Completed
                </span>
              </td>
            </tr>

            <tr v-if="!isFetching && orders.length === 0">
              <td colspan="5" class="px-6 py-12 text-center text-gray-500">
                No orders found.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modal -->
    <Transition name="modal">
      <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4">
        <div class="absolute inset-0 bg-gray-900/60 backdrop-blur-md" @click="showModal = false"></div>

        <div class="relative bg-white w-full max-w-3xl rounded-3xl shadow-2xl overflow-hidden flex flex-col max-h-[90vh]">
          <!-- Modal Header -->
          <div class="p-6 border-b border-gray-100 flex justify-between items-center bg-gradient-to-r from-gray-50 to-white">
            <div>
              <h2 class="text-xl font-extrabold text-gray-900 tracking-tight uppercase">
                Create New Sales Order
              </h2>
              <p class="text-xs text-gray-500 mt-1">
                Select a customer, add line items, and confirm.
              </p>
            </div>

            <button
              @click="showModal = false"
              class="w-10 h-10 rounded-xl text-gray-400 hover:text-gray-700 hover:bg-gray-50 transition-all flex items-center justify-center"
              aria-label="Close"
            >
              ✕
            </button>
          </div>

          <!-- Modal Body -->
          <div class="p-6 sm:p-8 overflow-y-auto space-y-8">
            <!-- Customer -->
            <div class="bg-gray-50/50 border border-gray-100 rounded-2xl p-5">
              <label class="block text-xs font-extrabold text-gray-500 uppercase tracking-widest mb-2 ml-1">
                Bill To Customer
              </label>

              <div class="relative">
                <select
                  v-model="selectedCustomer"
                  class="w-full px-4 py-3 rounded-xl border border-gray-200 bg-white focus:bg-white focus:ring-4 focus:ring-blue-500/10 focus:border-blue-500 outline-none transition-all appearance-none cursor-pointer font-bold"
                >
                  <option value="">Select a customer...</option>
                  <option v-for="c in customers" :key="c.id" :value="c.id">{{ c.name }}</option>
                </select>

                <div class="pointer-events-none absolute inset-y-0 right-3 flex items-center text-gray-400">
                  <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                    <path d="M5.23 7.21a.75.75 0 011.06.02L10 10.94l3.71-3.71a.75.75 0 011.08 1.04l-4.25 4.25a.75.75 0 01-1.06 0L5.21 8.27a.75.75 0 01.02-1.06z" />
                  </svg>
                </div>
              </div>
            </div>

            <!-- Line Items -->
            <div class="space-y-4">
              <div class="flex items-end justify-between">
                <div>
                  <p class="text-xs font-extrabold text-gray-500 uppercase tracking-widest">
                    Line Items
                  </p>
                  <p class="text-sm text-gray-500 mt-1">Products, quantities, pricing & subtotal</p>
                </div>

                <div class="text-xs font-extrabold text-gray-400">
                  { { items.length } } lines
                </div>
              </div>

              <div class="bg-white rounded-2xl border border-gray-100 p-3 sm:p-4">
                <div class="grid grid-cols-12 gap-4 px-2 py-2 text-[10px] font-extrabold text-gray-400 uppercase tracking-widest">
                  <div class="col-span-5">Product</div>
                  <div class="col-span-2 text-center">Qty</div>
                  <div class="col-span-2 text-right">Price</div>
                  <div class="col-span-2 text-right">Subtotal</div>
                  <div class="col-span-1"></div>
                </div>

                <div class="space-y-3 mt-2">
                  <div
                    v-for="(item, i) in items"
                    :key="i"
                    class="grid grid-cols-12 gap-4 items-center p-3 rounded-xl border transition-all"
                    :class="{
                      'border-red-200 bg-red-50/40': isDuplicate(item.product_id, i)
                    }"
                  >
                    <!-- Product -->
                    <div class="col-span-5">
                      <select
                        v-model="item.product_id"
                        class="w-full bg-white font-extrabold text-sm outline-none focus:text-blue-700 cursor-pointer border border-gray-200 rounded-lg px-3 py-2 transition-all focus:ring-4 focus:ring-blue-500/10"
                      >
                        <option value="">Select Product</option>
                        <option v-for="p in products" :key="p.id" :value="p.id">{{ p.name }}</option>
                      </select>

                      <p v-if="isDuplicate(item.product_id, i)" class="text-[10px] text-red-600 font-extrabold mt-1 uppercase">
                        Duplicate Item
                      </p>

                      <p
                        v-else
                        class="text-[10px] text-gray-400 font-extrabold mt-1 uppercase"
                      >
                        In stock: {{ getProduct(item.product_id)?.quantity ?? '—' }}
                      </p>
                    </div>

                    <!-- Quantity -->
                    <div class="col-span-2">
                      <input
                        type="number"
                        v-model.number="item.quantity"
                        min="1"
                        class="w-full bg-white border border-gray-200 rounded-lg py-2 px-2 text-center text-sm font-extrabold shadow-sm outline-none focus:ring-4 focus:ring-blue-500/10"
                      />
                    </div>

                    <!-- Price -->
                    <div class="col-span-2 text-right text-sm text-gray-600 font-extrabold">
                      {{ formatCurrency(getProduct(item.product_id)?.price || 0) }}
                    </div>

                    <!-- Subtotal -->
                    <div class="col-span-2 text-right text-sm font-extrabold text-gray-900">
                      {{ formatCurrency((getProduct(item.product_id)?.price || 0) * item.quantity) }}
                    </div>

                    <!-- Remove -->
                    <div class="col-span-1 text-right">
                      <button
                        @click="removeItem(i)"
                        class="w-10 h-10 rounded-xl text-gray-300 hover:text-red-600 hover:bg-red-50 transition-all border border-gray-100 inline-flex items-center justify-center"
                        :disabled="items.length <= 1"
                        :class="{ 'opacity-50 cursor-not-allowed': items.length <= 1 }"
                        aria-label="Remove line item"
                      >
                        <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                          <path d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" />
                        </svg>
                      </button>
                    </div>
                  </div>
                </div>

                <div class="mt-4 flex items-center justify-start">
                  <button
                    @click="addItem"
                    class="inline-flex items-center text-sm font-extrabold text-blue-700 hover:text-blue-900 transition-colors px-2 py-2 rounded-xl hover:bg-blue-50"
                  >
                    <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M12 4v16m8-8H4" />
                    </svg>
                    Add Line Item
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- Modal Footer -->
          <div class="p-6 sm:p-8 bg-gray-900 text-white flex flex-col sm:flex-row justify-between items-center gap-5">
            <div>
              <p class="text-xs font-extrabold text-gray-400 uppercase tracking-[0.2em] mb-1">
                Total Order Amount
              </p>
              <h3 class="text-4xl font-extrabold text-white">
                {{ formatCurrency(total) }}
              </h3>
            </div>

            <div class="flex gap-3 w-full sm:w-auto">
              <button
                @click="showModal = false"
                class="flex-1 sm:flex-none px-6 py-3 font-extrabold text-gray-300 hover:text-white transition-colors bg-white/0 hover:bg-white/10 rounded-xl border border-white/10"
              >
                Cancel
              </button>

              <button
                @click="submit"
                :disabled="loading || isInvalid"
                class="flex-1 sm:flex-none px-8 py-3 font-extrabold rounded-xl transition-all
                       hover:bg-blue-400 disabled:opacity-50 disabled:cursor-not-allowed
                       bg-blue-500 hover:bg-blue-400 text-white shadow-lg shadow-blue-400/20"
              >
                {{ loading ? 'Processing…' : 'Confirm Order' }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.modal-enter-active,
.modal-leave-active { transition: all 0.25s ease; }
.modal-enter-from,
.modal-leave-to { opacity: 0; transform: translateY(16px) scale(0.98); }
</style>
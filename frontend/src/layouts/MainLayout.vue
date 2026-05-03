<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const router = useRouter()
const route = useRoute()

const menu = [
  {
    section: 'Main Menu',
    items: [
      { name: 'Dashboard', path: '/dashboard', icon: 'dashboard' },
      { name: 'Products',  path: '/products',  icon: 'products', badge: '48' },
      { name: 'Customers', path: '/customers', icon: 'customers', badge: '1.2k' },
      { name: 'Orders',   path: '/orders',    icon: 'orders',   badge: '7' },
    ],
  },
  {
    section: 'Finance',
    items: [
      { name: 'Payments', path: '/payments', icon: 'payments' },
    ],
  },
]

const go = (path) => {
  if (route.path !== path) router.push(path)
}

const isActive = (path) => route.path === path

/* ── Dark mode ── */
const isDark = ref(false)

const applyTheme = (dark) => {
  document.documentElement.classList.toggle('dark', dark)
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  isDark.value =
    saved === 'dark' ? true
    : saved === 'light' ? false
    : (window.matchMedia?.('(prefers-color-scheme: dark)').matches ?? false)
  applyTheme(isDark.value)
})

watch(isDark, (val) => {
  applyTheme(val)
  localStorage.setItem('theme', val ? 'dark' : 'light')
})

const toggleTheme = () => { isDark.value = !isDark.value }
</script>

<template>
  <!-- Layout wrapper -->
  <div class="flex h-screen bg-gray-50 dark:bg-gray-900">
    
    <!-- ═══════════════════════════════════════════
         Sidebar shell
    ════════════════════════════════════════════ -->
    <aside
      class="flex flex-col flex-shrink-0 w-64 h-screen
             bg-white dark:bg-gray-950
             border-r border-gray-100 dark:border-gray-800
             select-none"
      aria-label="Sidebar navigation"
    >

      <!-- ── Brand ── -->
      <div class="flex items-center gap-3 px-5 py-4 border-b border-gray-100 dark:border-gray-800">
        <!-- Logo mark -->
        <div class="flex-shrink-0 w-9 h-9 rounded-xl bg-blue-600 flex items-center justify-center">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none" aria-hidden="true">
            <rect x="2"  y="2"  width="6" height="6" rx="1.5" fill="white" opacity="0.95"/>
            <rect x="10" y="2"  width="6" height="6" rx="1.5" fill="white" opacity="0.65"/>
            <rect x="2"  y="10" width="6" height="6" rx="1.5" fill="white" opacity="0.65"/>
            <rect x="10" y="10" width="6" height="6" rx="1.5" fill="white" opacity="0.40"/>
          </svg>
        </div>

        <div class="leading-tight">
          <p class="text-[11px] tracking-widest text-gray-400 dark:text-gray-500 uppercase">
            E-Commerce
          </p>
          <p class="text-sm font-semibold text-gray-900 dark:text-white">
            Admin Panel
          </p>
        </div>
      </div>

      <!-- ── Navigation ── -->
      <nav class="flex-1 overflow-y-auto px-3 py-4 space-y-5">
        <div v-for="group in menu" :key="group.section">

          <!-- Section label -->
          <p class="mb-1.5 px-2 text-[10px] font-medium tracking-[0.08em] uppercase
                    text-gray-400 dark:text-gray-500">
            {{ group.section }}
          </p>

          <!-- Nav items -->
          <div class="space-y-0.5">
            <button
              v-for="item in group.items"
              :key="item.name"
              type="button"
              @click="go(item.path)"
              :aria-current="isActive(item.path) ? 'page' : undefined"
              class="group w-full flex items-center gap-3 px-2.5 py-2 rounded-lg
                     text-sm font-medium transition-colors duration-150
                     focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500
                     focus-visible:ring-offset-2 focus-visible:ring-offset-white
                     dark:focus-visible:ring-offset-gray-950"
              :class="isActive(item.path)
                ? 'bg-blue-50 dark:bg-blue-950 text-blue-700 dark:text-blue-300'
                : 'text-gray-600 dark:text-gray-400 hover:bg-gray-50 dark:hover:bg-gray-900 hover:text-gray-900 dark:hover:text-gray-100'"
            >
              <!-- Icon tile -->
              <span
                class="flex-shrink-0 flex items-center justify-center w-7 h-7 rounded-md transition-colors duration-150"
                :class="isActive(item.path)
                  ? 'bg-blue-600 text-white'
                  : 'bg-gray-100 dark:bg-gray-800 text-gray-500 dark:text-gray-400 group-hover:bg-gray-200 dark:group-hover:bg-gray-700 group-hover:text-gray-700 dark:group-hover:text-gray-200'"
                aria-hidden="true"
              >
                <!-- Dashboard -->
                <svg v-if="item.icon === 'dashboard'" width="13" height="13" viewBox="0 0 13 13" fill="none">
                  <rect x="1" y="1" width="4.5" height="4.5" rx="1" fill="currentColor" opacity="0.9"/>
                  <rect x="7.5" y="1" width="4.5" height="4.5" rx="1" fill="currentColor" opacity="0.6"/>
                  <rect x="1" y="7.5" width="4.5" height="4.5" rx="1" fill="currentColor" opacity="0.6"/>
                  <rect x="7.5" y="7.5" width="4.5" height="4.5" rx="1" fill="currentColor" opacity="0.4"/>
                </svg>

                <!-- Products -->
                <svg v-else-if="item.icon === 'products'" width="13" height="13" viewBox="0 0 13 13" fill="none">
                  <rect x="1.5" y="3" width="10" height="8.5" rx="1.5" stroke="currentColor" stroke-width="1.2" fill="none"/>
                  <path d="M4 3V2.5a2.5 2.5 0 015 0V3" stroke="currentColor" stroke-width="1.2" stroke-linecap="round"/>
                  <path d="M4 6.5h5M4 9h3" stroke="currentColor" stroke-width="1" stroke-linecap="round"/>
                </svg>

                <!-- Customers -->
                <svg v-else-if="item.icon === 'customers'" width="13" height="13" viewBox="0 0 13 13" fill="none">
                  <circle cx="6.5" cy="4" r="2.5" stroke="currentColor" stroke-width="1.2" fill="none"/>
                  <path d="M1 12c0-2.485 2.462-4.5 5.5-4.5S12 9.515 12 12" stroke="currentColor" stroke-width="1.2" stroke-linecap="round"/>
                </svg>

                <!-- Orders -->
                <svg v-else-if="item.icon === 'orders'" width="13" height="13" viewBox="0 0 13 13" fill="none">
                  <rect x="1.5" y="2" width="10" height="9" rx="1.5" stroke="currentColor" stroke-width="1.2" fill="none"/>
                  <path d="M4 5h5M4 7.5h3" stroke="currentColor" stroke-width="1" stroke-linecap="round"/>
                </svg>

                <!-- Payments -->
                <svg v-else-if="item.icon === 'payments'" width="13" height="13" viewBox="0 0 13 13" fill="none">
                  <rect x="1" y="3.5" width="11" height="7.5" rx="1.5" stroke="currentColor" stroke-width="1.2" fill="none"/>
                  <path d="M1 6.5h11" stroke="currentColor" stroke-width="1.2"/>
                  <circle cx="4" cy="8.75" r="0.75" fill="currentColor"/>
                  <circle cx="6.5" cy="8.75" r="0.75" fill="currentColor"/>
                </svg>
              </span>

              <!-- Label -->
              <span class="flex-1 text-left">{{ item.name }}</span>

              <!-- Badge -->
              <span
                v-if="item.badge"
                class="text-[10px] font-semibold px-1.5 py-0.5 rounded-full leading-none"
                :class="isActive(item.path)
                  ? 'bg-blue-100 dark:bg-blue-900 text-blue-700 dark:text-blue-300'
                  : 'bg-gray-100 dark:bg-gray-800 text-gray-500 dark:text-gray-400'"
              >
                {{ item.badge }}
              </span>

              <!-- Active indicator dot -->
              <span
                v-if="isActive(item.path)"
                class="w-1.5 h-1.5 rounded-full bg-blue-600 dark:bg-blue-400 flex-shrink-0"
                aria-hidden="true"
              />
            </button>
          </div>

        </div>
      </nav>

      <!-- ── Footer ── -->
      <div class="px-4 py-3 border-t border-gray-100 dark:border-gray-800">
        <div class="flex items-center gap-3">

          <!-- Avatar -->
          <div
            class="flex-shrink-0 w-8 h-8 rounded-full flex items-center justify-center
                   bg-blue-50 dark:bg-blue-950 border border-blue-100 dark:border-blue-900
                   text-blue-700 dark:text-blue-300 text-xs font-semibold"
          >
            A
          </div>

          <!-- User info -->
          <div class="flex-1 min-w-0">
            <p class="text-sm font-medium text-gray-900 dark:text-gray-100 truncate">Admin</p>
            <p class="text-[11px] text-gray-400 dark:text-gray-500 truncate">Administrator</p>
          </div>

          <!-- Theme toggle -->
          <button
            type="button"
            @click="toggleTheme"
            class="flex-shrink-0 w-7 h-7 rounded-md flex items-center justify-center
                   border border-gray-200 dark:border-gray-700
                   text-gray-400 dark:text-gray-500
                   hover:bg-gray-50 dark:hover:bg-gray-800
                   hover:text-gray-700 dark:hover:text-gray-200
                   transition-colors duration-150 focus:outline-none
                   focus-visible:ring-2 focus-visible:ring-blue-500"
            :aria-label="isDark ? 'Switch to light mode' : 'Switch to dark mode'"
          >
            <!-- Sun icon (light mode) -->
            <svg v-if="!isDark" width="13" height="13" viewBox="0 0 13 13" fill="none" aria-hidden="true">
              <circle cx="6.5" cy="6.5" r="2.5" stroke="currentColor" stroke-width="1.1"/>
              <path d="M6.5 1v1M6.5 11v1M1 6.5h1M11 6.5h1M2.55 2.55l.7.7M9.75 9.75l.7.7M9.75 2.55l-.7.7M2.55 9.75l.7-.7" stroke="currentColor" stroke-width="1.1" stroke-linecap="round"/>
            </svg>
            <!-- Moon icon (dark mode) -->
            <svg v-else width="13" height="13" viewBox="0 0 13 13" fill="none" aria-hidden="true">
              <path d="M10.5 7.5A4.5 4.5 0 115.5 2.5a5.5 5.5 0 005 5z" fill="currentColor"/>
            </svg>
          </button>
        </div>
      </div>
    </aside>

    <!-- ═══════════════════════════════════════════
         Main content area
    ════════════════════════════════════════════ -->
    <main class="flex-1 p-6 overflow-y-auto">
      <router-view />
    </main>

  </div>
</template>
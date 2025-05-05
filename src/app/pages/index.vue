<script setup lang="ts">
import { h, onMounted, ref, shallowRef, computed } from 'vue'
import { getPaginationRowModel, type Table } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'

useHead({ title: 'Список продуктів' })

type Product = {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const data = shallowRef<Product[]>([])
const loading = ref(true)
const pagination = ref({ pageIndex: 0, pageSize: 10 })
const table = ref<{ tableApi: Table<Product> } | null>(null)

// Creates a sortable column header with arrows for sort direction
const createSortableHeader = (title: string) => ({ column }: { column: any }) => h('div', {
  class: 'cursor-pointer select-none',
  onClick: () => column.toggleSorting()
}, [
  title,
  column.getIsSorted() === 'asc' ? ' 🔼' : column.getIsSorted() === 'desc' ? ' 🔽' : ''
])

async function fetchProducts() {
  loading.value = true
  
  try {
    const response = await fetch('https://dummyjson.com/products?limit=150')
    const result = await response.json()
    data.value = result.products
  } catch (error) {
    console.error('Error fetching products:', error)
  } finally {
    loading.value = false
  }
}

onMounted(fetchProducts)

// Memoized table columns definition
const columns = computed<TableColumn<Product>[]>(() => [{
  accessorKey: 'thumbnail',
  header: 'Фото',
  cell: ({ row }) => h('img', {
    src: row.getValue('thumbnail'),
    alt: row.getValue('title'),
    class: 'w-24 h-24 object-cover rounded',
    loading: 'lazy',
  }),
  enableSorting: false,
}, {
  accessorKey: 'title',
  header: createSortableHeader('Назва'),
  cell: ({ row }) => h('div', { class: 'font-medium' }, row.getValue('title')),
  enableSorting: true,
}, {
  accessorKey: 'description',
  header: 'Опис',
  cell: ({ row }) => h('div', { class: 'max-w-md truncate' }, row.getValue('description')),
  enableSorting: false,
}, {
  accessorKey: 'price',
  header: createSortableHeader('Ціна'),
  cell: ({ row }) => {
    const price = Number.parseFloat(row.getValue('price'))
    const formatted = new Intl.NumberFormat('uk-UA', {
      style: 'currency',
      currency: 'USD'
    }).format(price)
    return h('div', { class: 'font-medium' }, formatted)
  },
  enableSorting: true,
}, {
  accessorKey: 'rating',
  header: createSortableHeader('Оцінка'),
  cell: ({ row }) => {
    const rating = Number(row.getValue('rating'))
    return h('div', { 
      class: `font-medium ${rating >= 4.5 ? 'text-green-500' : 'text-red-500'}` 
    }, rating.toFixed(1))
  },
  enableSorting: true,
}, {
  accessorKey: 'brand',
  header: 'Бренд',
  cell: ({ row }) => h('div', {}, row.getValue('brand')),
  enableSorting: true, 
}, {
  accessorKey: 'category',
  header: 'Категорія',
  cell: ({ row }) => h('UBadge', { 
    class: 'capitalize', 
    variant: 'subtle', 
    color: 'info' 
  }, { default: () => row.getValue('category') }),
  enableSorting: true, 
}])

// Computed table statistics for display
const tableStats = computed(() => ({
  selectedRowsCount: table.value?.tableApi?.getFilteredSelectedRowModel().rows.length || 0,
  totalFilteredRows: table.value?.tableApi?.getFilteredRowModel().rows.length || 0,
  currentPage: (table.value?.tableApi?.getState().pagination.pageIndex || 0) + 1,
  pageSize: table.value?.tableApi?.getState().pagination.pageSize,
}))
</script>

<template>
  <div class="flex-1 divide-y divide-(--ui-border-accented) w-full">
    <div class="bg-blue-500 text-white py-4 px-6 rounded-b-lg shadow-md">
      <h1 class="text-2xl font-bold">Список продуктів</h1>
      <p class="text-sm text-blue-100">Переглядайте, сортуйте та керуйте продуктами</p>
    </div>

    <div class="flex items-center gap-2 px-4 py-3.5 overflow-x-auto">
      <UInput
        :model-value="(table?.tableApi?.getColumn('title')?.getFilterValue() as string)"
        class="max-w-sm min-w-[12ch]"
        placeholder="Пошук за назвою..."
        @update:model-value="table?.tableApi?.getColumn('title')?.setFilterValue($event)"
      />
      <UButton color="info" label="Оновити" icon="i-lucide-refresh-cw" @click="fetchProducts" />
    </div>

    <!-- Loading skeleton -->
    <div v-if="loading" class="h-96 w-full flex items-center justify-center">
      <div class="text-center">
        <ULoading size="lg" class="mb-4" />
        <p class="text-sm text-gray-600 dark:text-gray-300">Завантаження даних...</p>
      </div>
    </div>

    <!-- Data table -->
    <UTable
      v-else
      ref="table"
      v-model:pagination="pagination"
      :data="data"
      :columns="columns"
      :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
      sticky
      class="h-96 border border-gray-200 dark:border-gray-700 rounded-lg shadow-sm"
    />

    <!-- Pagination -->
    <div v-if="!loading" class="flex justify-center border-t border-default pt-4 px-4 py-3">
      <UPagination
        :default-page="tableStats.currentPage"
        :items-per-page="tableStats.pageSize"
        :total="tableStats.totalFilteredRows"
        @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>

    <!-- Selected rows statistics -->
    <div v-if="!loading" class="px-4 py-3.5 text-sm text-(--ui-text-muted)">
      {{ tableStats.totalFilteredRows }} рядків завантажено.
    </div>
  </div>
</template>

<style scoped>
button {
  transition: background-color 0.3s;
}

tr { transition: background-color 0.3s; }
</style>
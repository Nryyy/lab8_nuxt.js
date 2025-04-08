<script setup lang="ts">
import { h, resolveComponent, onMounted } from 'vue'
import { upperFirst } from 'scule'
import type { TableColumn } from '@nuxt/ui'

useHead({
  title: 'Список продуктів'
})

const UButton = resolveComponent('UButton')
const UCheckbox = resolveComponent('UCheckbox')
const UBadge = resolveComponent('UBadge')
const UDropdownMenu = resolveComponent('UDropdownMenu')

const toast = useToast()

type Product = {
  id: number
  title: string
  description: string
  price: number
  discountPercentage: number
  rating: number
  stock: number
  brand: string
  category: string
  thumbnail: string
  images: string[]
}

const data = ref<Product[]>([])
const loading = ref(true)
const progress = ref(0)
const progressInterval = ref<NodeJS.Timeout | null>(null)

// Fetch products from DummyJSON API
async function fetchProducts() {
  loading.value = true
  progress.value = 0

  // Start progress animation
  progressInterval.value = setInterval(() => {
    progress.value += Math.random() * 15
    if (progress.value >= 95) {
      progress.value = 95
      if (progressInterval.value) {
        clearInterval(progressInterval.value)
      }
    }
  }, 200)

  try {
    const response = await fetch('https://dummyjson.com/products?limit=100')
    const result = await response.json()
    data.value = result.products

    // Complete progress animation
    progress.value = 100
    setTimeout(() => {
      loading.value = false
    }, 500) // Small delay to show completed progress

  } catch (error) {
    console.error('Error fetching products:', error)
    toast.add({
      title: 'Error fetching products',
      color: 'error',
      icon: 'i-lucide-alert-circle'
    })
    loading.value = false
  } finally {
    if (progressInterval.value) {
      clearInterval(progressInterval.value)
    }
  }
}

onMounted(() => {
  fetchProducts()
})

const columns: TableColumn<Product>[] = [{
  id: 'select',
  header: ({ table }) => h(UCheckbox, {
    'modelValue': table.getIsSomePageRowsSelected() ? 'indeterminate' : table.getIsAllPageRowsSelected(),
    'onUpdate:modelValue': (value: boolean | 'indeterminate') => table.toggleAllPageRowsSelected(!!value),
    'aria-label': 'Select all'
  }),
  cell: ({ row }) => h(UCheckbox, {
    'modelValue': row.getIsSelected(),
    'onUpdate:modelValue': (value: boolean | 'indeterminate') => row.toggleSelected(!!value),
    'aria-label': 'Select row'
  }),
  enableSorting: false,
  enableHiding: false
}, {
  accessorKey: 'thumbnail',
  header: 'Фото',
  cell: ({ row }) => h('img', {
    src: row.getValue('thumbnail'),
    alt: row.getValue('title'),
    class: 'w-24 h-24 object-cover rounded',
    style: 'width: 100px; height: 100px;'
  }),
  enableSorting: false,
}, {
  accessorKey: 'title',
  header: 'Назва',
  cell: ({ row }) => h('div', { class: 'font-medium' }, row.getValue('title'))
}, {
  accessorKey: 'description',
  header: 'Опис',
  cell: ({ row }) => h('div', { class: 'max-w-md truncate' }, row.getValue('description')),
}, {
  accessorKey: 'price',
  header: 'Ціна',
  cell: ({ row }) => {
    const price = Number.parseFloat(row.getValue('price'))

    const formatted = new Intl.NumberFormat('uk-UA', {
      style: 'currency',
      currency: 'USD'
    }).format(price)

    return h('div', { class: 'font-medium' }, formatted)
  }
}, {
  accessorKey: 'rating',
  header: 'Оцінка',
  cell: ({ row }) => {
    const rating = Number(row.getValue('rating'))
    const textColor = rating >= 4.5 ? 'text-green-500' : 'text-red-500'

    return h('div', { class: `font-medium ${textColor}` }, rating.toFixed(1))
  }
}, {
  accessorKey: 'brand',
  header: 'Бренд',
  cell: ({ row }) => h('div', {}, row.getValue('brand'))
}, {
  accessorKey: 'category',
  header: 'Категорія',
  cell: ({ row }) => {
    return h(UBadge, { class: 'capitalize', variant: 'subtle', color: 'info' }, () => row.getValue('category'))
  }
}, {
  id: 'actions',
  enableHiding: false,
  cell: ({ row }) => {
    const items = [{
      type: 'label',
      label: 'Дії'
    }, {
      label: 'Копіювати ID',
      onSelect() {
        navigator.clipboard.writeText(row.original.id.toString())

        toast.add({
          title: 'ID скопійовано!',
          color: 'success',
          icon: 'i-lucide-circle-check'
        })
      }
    }, {
      label: row.getIsExpanded() ? 'Згорнути' : 'Розгорнути',
      onSelect() {
        row.toggleExpanded()
      }
    }, {
      type: 'separator'
    }, {
      label: 'Переглянути деталі'
    }, {
      label: 'Додати в кошик'
    }]

    return h('div', { class: 'text-right' }, h(UDropdownMenu, {
      'content': {
        align: 'end'
      },
      items,
      'aria-label': 'Actions dropdown'
    }, () => h(UButton, {
      'icon': 'i-lucide-ellipsis-vertical',
      'color': 'neutral',
      'variant': 'ghost',
      'class': 'ml-auto',
      'aria-label': 'Actions dropdown'
    })))
  }
}]

const table = useTemplateRef('table')

function randomize() {
  data.value = [...data.value].sort(() => Math.random() - 0.5)
}

// Function to reload data
function reloadData() {
  fetchProducts()
}

// Create skeleton rows for loading state
const skeletonRows = Array(10).fill(0).map((_, i) => i)
</script>

<template>
  <div class="flex-1 divide-y divide-(--ui-border-accented) w-full">
    <!-- Progress bar for loading state -->
    <div v-if="loading" class="w-full h-1 bg-gray-200 dark:bg-gray-700 fixed top-0 left-0 z-50">
      <div class="h-1 bg-blue-500 transition-all duration-300 ease-out" :style="{ width: `${progress}%` }"></div>
    </div>

    <div class="flex items-center gap-2 px-4 py-3.5 overflow-x-auto">
      <UInput
          :model-value="(table?.tableApi?.getColumn('title')?.getFilterValue() as string)"
          class="max-w-sm min-w-[12ch]"
          placeholder="Пошук за назвою..."
          @update:model-value="table?.tableApi?.getColumn('title')?.setFilterValue($event)"
      />

      <UButton color="neutral" label="Випадково" @click="randomize" />
      <UButton color="info" label="Оновити" icon="i-lucide-refresh-cw" @click="reloadData" />

      <UDropdownMenu
          :items="table?.tableApi?.getAllColumns().filter(column => column.getCanHide()).map(column => ({
          label: upperFirst(column.id),
          type: 'checkbox' as const,
          checked: column.getIsVisible(),
          onUpdateChecked(checked: boolean) {
            table?.tableApi?.getColumn(column.id)?.toggleVisibility(!!checked)
          },
          onSelect(e?: Event) {
            e?.preventDefault()
          }
        }))"
          :content="{ align: 'end' }"
      >
        <UButton
            label="Колонки"
            color="neutral"
            variant="outline"
            trailing-icon="i-lucide-chevron-down"
            class="ml-auto"
            aria-label="Columns select dropdown"
        />
      </UDropdownMenu>
    </div>

    <div v-if="loading" class="relative h-96 w-full">
      <!-- Loading overlay -->
      <div class="absolute inset-0 flex flex-col items-center justify-center bg-white dark:bg-gray-800 bg-opacity-80 dark:bg-opacity-80 z-10">
        <ULoading size="lg" class="mb-4" />
        <p class="text-sm text-gray-600 dark:text-gray-300">Завантаження даних про продукти...</p>
        <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">{{ Math.round(progress) }}% завершено</p>
      </div>

      <!-- Skeleton table for better loading UX -->
      <table class="w-full table-auto">
        <thead>
        <tr class="border-b border-gray-200 dark:border-gray-700">
          <th class="p-3 text-left">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-32 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-20 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-left">
            <div class="h-5 w-20 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </th>
          <th class="p-3 text-right">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded ml-auto"></div>
          </th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="index in skeletonRows" :key="index" class="border-b border-gray-200 dark:border-gray-700">
          <td class="p-3">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-24 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-32 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-48 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-16 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-10 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3">
            <div class="h-5 w-24 bg-gray-200 dark:bg-gray-700 rounded"></div>
          </td>
          <td class="p-3 text-right">
            <div class="h-5 w-5 bg-gray-200 dark:bg-gray-700 rounded ml-auto"></div>
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <UTable
        v-else
        ref="table"
        :data="data"
        :columns="columns"
        sticky
        class="h-96"
    >
      <template #expanded="{ row }">
        <div class="p-4 bg-gray-50 dark:bg-gray-800">
          <div class="flex gap-4">
            <div class="flex space-x-2">
              <img v-for="(image, idx) in row.original.images.slice(0, 3)" :key="idx"
                   :src="image" :alt="`${row.original.title} - image ${idx + 1}`"
                   class="w-32 h-32 object-cover rounded"
              />
            </div>
            <div>
              <h3 class="text-lg font-semibold">{{ row.original.title }}</h3>
              <p class="text-sm text-gray-600 dark:text-gray-300 mt-1">{{ row.original.description }}</p>
              <div class="mt-2 flex flex-wrap gap-2">
                <div>
                  <span class="text-sm font-medium">Знижка: </span>
                  <UBadge color="success">{{ row.original.discountPercentage }}% off</UBadge>
                </div>
                <div>
                  <span class="text-sm font-medium">Залишок: </span>
                  <UBadge :color="row.original.stock > 50 ? 'success' : 'warning'">{{ row.original.stock }} шт.</UBadge>
                </div>
              </div>
            </div>
          </div>
        </div>
      </template>
    </UTable>

    <div class="px-4 py-3.5 text-sm text-(--ui-text-muted)">
      {{ table?.tableApi?.getFilteredSelectedRowModel().rows.length || 0 }} з
      {{ table?.tableApi?.getFilteredRowModel().rows.length || 0 }} рядків вибрано.
    </div>
  </div>
</template>
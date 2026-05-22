<template>
  <div class="relative flex h-full min-h-0 w-full flex-1 flex-col overflow-x-auto !px-[1rem]">
    <div
      class="flex h-full min-h-0 w-max min-w-full flex-col overflow-y-hidden rounded-[16px] border border-solid border-[#E7E7E7] bg-[#FFFFFF] dark:!border-[#656565] dark:!bg-[#232830]"
      style="border: 1px solid #e7e7e7"
      :class="$attrs.class"
    >
      <slot v-bind="{ showGroupedRows, selectable }">
        <ListHeader />
        <template v-if="props.rows.length">
          <ListGroups v-if="showGroupedRows" />
          <ListRows v-else />
        </template>
        <ListEmptyState v-else />
        <ListSelectBanner v-if="selectable" />
      </slot>
    </div>
  </div>
</template>
<script setup>
import ListEmptyState from './ListEmptyState.vue'
import ListHeader from './ListHeader.vue'
import ListRows from './ListRows.vue'
import ListGroups from './ListGroups.vue'
import ListSelectBanner from './ListSelectBanner.vue'
import { reactive, computed, provide, watch, useSlots, ref } from 'vue'

defineOptions({
  inheritAttrs: false,
})

const props = defineProps({
  columns: {
    type: Array,
    default: [],
  },
  rows: {
    type: Array,
    default: [],
  },
  rowKey: {
    type: String,
    required: true,
  },
  options: {
    type: Object,
    default: () => ({
      getRowRoute: null,
      onRowClick: null,
      showTooltip: true,
      selectable: true,
      resizeColumn: false,
      rowHeight: 40,
      emptyState: {
        title: 'No Data',
        description: 'No data available',
      },
    }),
  },
})

const slots = useSlots()

let selections = reactive(new Set())
const lastSelectedRow = ref(null)

const emit = defineEmits(['update:selections'])

watch(selections, (value) => {
  emit('update:selections', value)
})

let _options = computed(() => {
  function defaultTrue(value) {
    return value === undefined ? true : value
  }

  function defaultFalse(value) {
    return value === undefined ? false : value
  }

  return {
    getRowRoute: props.options.getRowRoute || null,
    onRowClick: props.options.onRowClick || null,
    showTooltip: defaultTrue(props.options.showTooltip),
    selectable: defaultTrue(props.options.selectable),
    resizeColumn: defaultFalse(props.options.resizeColumn),
    rowHeight: props.options.rowHeight || 40,
    emptyState: props.options.emptyState,
  }
})

const allRowsSelected = computed(() => {
  if (!props.rows.length) return false
  if (showGroupedRows.value) {
    return (
      selections.size ===
      props.rows.reduce((acc, row) => acc + row.rows.length, 0)
    )
  }
  return selections.size === props.rows.length
})

const selectable = computed(() => {
  return _options.value.selectable
})

let showGroupedRows = computed(() => {
  return props.rows.every(
    (row) => row.group && row.rows && Array.isArray(row.rows)
  )
})

function getFlatRowKeys() {
  if (showGroupedRows.value) {
    return props.rows.flatMap((row) => row.rows.map((r) => r[props.rowKey]))
  }
  return props.rows.map((row) => row[props.rowKey])
}

function toggleRow(row, options = {}) {
  const { shiftKey = false } = options

  if (shiftKey && lastSelectedRow.value !== null) {
    const flatRowKeys = getFlatRowKeys()
    const start = flatRowKeys.indexOf(lastSelectedRow.value)
    const end = flatRowKeys.indexOf(row)

    if (start !== -1 && end !== -1) {
      const targetSelectedState = !selections.has(row)
      const rangeStart = Math.min(start, end)
      const rangeEnd = Math.max(start, end)

      for (let i = rangeStart; i <= rangeEnd; i++) {
        const rowKey = flatRowKeys[i]
        if (targetSelectedState) {
          selections.add(rowKey)
        } else {
          selections.delete(rowKey)
        }
      }

      lastSelectedRow.value = row
      return
    }
  }
  if (!selections.delete(row)) {
    selections.add(row)
  }

  lastSelectedRow.value = row
}

function toggleAllRows(select) {
  if (!select || allRowsSelected.value) {
    selections.clear()
    lastSelectedRow.value = null
    return
  }
  if (showGroupedRows.value) {
    props.rows.forEach((row) => {
      row.rows.forEach((r) => selections.add(r[props.rowKey]))
    })
    lastSelectedRow.value = null
    return
  }
  props.rows.forEach((row) => selections.add(row[props.rowKey]))
  lastSelectedRow.value = null
}

provide(
  'list',
  computed(() => ({
    rowKey: props.rowKey,
    rows: props.rows,
    columns: props.columns,
    options: _options.value,
    selections: selections,
    allRowsSelected: allRowsSelected.value,
    slots: slots,
    toggleRow,
    toggleAllRows,
  }))
)
</script>

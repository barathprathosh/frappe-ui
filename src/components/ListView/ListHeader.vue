<template>
  <div
    class="grid !h-[48px] items-center space-x-4 rounded border-b border-transparent !bg-[#E6F3F1] px-3 py-2 !text-black hover:!bg-gray-300 dark:!border-[#656565] dark:!bg-[#E6F3F1]"
    :style="{
      gridTemplateColumns: getGridTemplateColumns(
        list.columns,
        list.options.selectable
      ),
    }"
  >
    <Checkbox
      v-if="list.options.selectable"
      size="md"
      class="cursor-pointer duration-300 dark:!bg-[#232830]"
      :modelValue="list.allRowsSelected"
      @click.stop="list.toggleAllRows"
    />
    <slot>
      <ListHeaderItem
        v-for="column in list.columns"
        :key="column.key"
        :item="column"
        @columnWidthUpdated="emit('columnWidthUpdated', column)"
      />
    </slot>
  </div>
</template>

<script setup>
import Checkbox from '../Checkbox.vue'
import ListHeaderItem from './ListHeaderItem.vue'
import { getGridTemplateColumns } from './utils'
import { inject } from 'vue'

const emit = defineEmits(['columnWidthUpdated'])

const list = inject('list')
</script>

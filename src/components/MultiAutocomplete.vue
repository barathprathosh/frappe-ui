<template>
  <div class="w-full">
    <Popover class="w-full" v-model:show="showOptions">
      <template #target="{ togglePopover }">
        <button
          type="button"
          class="flex min-h-[32px] w-full items-center justify-between rounded border border-gray-300 bg-white px-2 py-1.5 text-left text-base text-gray-800 focus:outline-none"
          :class="{ 'cursor-not-allowed opacity-60': disabled }"
          :disabled="disabled"
          @click="togglePopover()"
        >
          <div class="flex flex-1 flex-wrap gap-1">
            <span
              v-if="selectedOptions.length === 0"
              class="text-gray-500"
            >
              {{ placeholder }}
            </span>
            <span
              v-for="option in selectedOptions"
              :key="option.value"
              class="inline-flex items-center gap-1 rounded bg-gray-100 px-2 py-0.5 text-sm"
            >
              <span>{{ option.label }}</span>
              <button
                type="button"
                class="text-gray-500 hover:text-gray-700"
                :disabled="disabled"
                @click.stop="removeValue(option.value)"
              >
                <FeatherIcon name="x" class="h-3 w-3" />
              </button>
            </span>
          </div>
          <FeatherIcon
            name="chevron-down"
            class="ml-2 h-4 w-4 shrink-0 text-gray-600"
          />
        </button>
      </template>

      <template #body="{ isOpen }">
        <div
          v-show="isOpen"
          class="mt-1 rounded-lg border border-gray-200 bg-white py-1 text-base shadow-2xl"
        >
          <div class="px-1.5 pt-0.5">
            <input
              ref="search"
              v-model="query"
              class="form-input w-full border bg-white"
              type="text"
              autocomplete="off"
              placeholder="Search"
            />
          </div>

          <div class="my-1 max-h-[12rem] overflow-y-auto px-1.5">
            <button
              v-for="option in filteredOptions"
              :key="option.value"
              type="button"
              class="flex w-full items-center justify-between rounded px-2.5 py-1.5 text-left text-base"
              :class="
                isSelected(option.value)
                  ? 'bg-gray-100 text-gray-900'
                  : 'hover:bg-gray-100'
              "
              @click="toggleOption(option.value)"
            >
              <span>{{ option.label }}</span>
              <FeatherIcon
                v-if="isSelected(option.value)"
                name="check"
                class="h-4 w-4"
              />
            </button>

            <div
              v-if="filteredOptions.length === 0"
              class="rounded px-2.5 py-1.5 text-base text-gray-600"
            >
              No results found
            </div>
          </div>
        </div>
      </template>
    </Popover>
  </div>
</template>

<script setup>
import { computed, nextTick, ref, watch } from 'vue'
import Popover from './Popover.vue'
import FeatherIcon from './FeatherIcon.vue'

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [],
  },
  options: {
    type: Array,
    default: () => [],
  },
  placeholder: {
    type: String,
    default: '',
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  multiple: {
    type: Boolean,
    default: true,
  },
})

const emit = defineEmits(['update:modelValue', 'change'])

const showOptions = ref(false)
const query = ref('')
const search = ref(null)

const normalizedOptions = computed(() =>
  Array.isArray(props.options) ? props.options : []
)

const normalizedValue = computed(() =>
  Array.isArray(props.modelValue) ? props.modelValue : []
)

const filteredOptions = computed(() => {
  if (!query.value) return normalizedOptions.value

  const searchText = query.value.toLowerCase()
  return normalizedOptions.value.filter((option) => {
    const label = (option?.label || '').toString().toLowerCase()
    const value = (option?.value || '').toString().toLowerCase()
    return label.includes(searchText) || value.includes(searchText)
  })
})

const selectedOptions = computed(() => {
  const selected = new Set(normalizedValue.value)
  return normalizedOptions.value.filter((option) => selected.has(option.value))
})

function isSelected(value) {
  return normalizedValue.value.includes(value)
}

function updateValue(value) {
  emit('update:modelValue', value)
  emit('change', value)
}

function toggleOption(value) {
  const current = [...normalizedValue.value]
  const index = current.indexOf(value)

  if (index >= 0) {
    current.splice(index, 1)
  } else if (props.multiple) {
    current.push(value)
  } else {
    current.splice(0, current.length, value)
  }

  updateValue(current)
}

function removeValue(value) {
  updateValue(normalizedValue.value.filter((item) => item !== value))
}

watch(showOptions, (value) => {
  if (!value) {
    query.value = ''
    return
  }

  nextTick(() => {
    search.value?.focus()
  })
})
</script>

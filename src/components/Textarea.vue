<template>
  <textarea
    :placeholder="placeholder"
    :class="inputClasses"
    :disabled="disabled"
    :id="id"
    :value="modelValue"
    :rows="rows"
    @input="handleChange"
    @change="handleChange"
    v-bind="attrs"
  />
</template>

<script setup lang="ts">
import { computed, useAttrs } from 'vue'
import type { TextInputTypes } from './types/TextInput'
import debounce from '../utils/debounce'

interface TextareaProps {
  size?: 'sm' | 'md' | 'lg' | 'xl'
  variant?: 'subtle' | 'outline'
  placeholder?: string
  disabled?: boolean
  id?: string
  modelValue?: string
  debounce?: number
  rows?: number
}

const props = withDefaults(defineProps<TextareaProps>(), {
  type: 'text',
  size: 'sm',
  variant: 'subtle',
  rows: 3,
})

const emit = defineEmits(['update:modelValue'])
const attrs = useAttrs()

const inputClasses = computed(() => {
  let sizeClasses = {
    sm: 'text-base rounded',
    md: 'text-base rounded',
    lg: 'text-lg rounded-md',
    xl: 'text-xl rounded-md',
  }[props.size]

  let paddingClasses = {
    sm: ['py-1.5 px-2'],
    md: ['py-1.5 px-2.5'],
    lg: ['py-1.5 px-3'],
    xl: ['py-1.5 px-3'],
  }[props.size]

  let variant = props.disabled ? 'disabled' : props.variant
  let variantClasses = {
    subtle:
      'border border-gray-100 bg-gray-100 placeholder-gray-500 hover:border-gray-200 hover:bg-gray-200 focus:bg-white focus:border-gray-500 focus:shadow-sm focus:ring-0 focus-visible:ring-2 focus-visible:ring-gray-400',
    outline:
      'outline-border border-[#898989 !important] !h-[100px] !rounded-[8px] dark:!bg-[#101213] dark:!text-[white] dark:!border-[#898989!important] bg-surface-white placeholder-ink-gray-4 hover:border-outline-gray-3 hover:shadow-sm focus:bg-surface-white focus:border-outline-gray-4 focus:shadow-sm focus:ring-0 focus-visible:ring-2 focus-visible:ring-outline-gray-3',
    disabled: [
      'border bg-gray-50 placeholder-gray-400',
      props.variant === 'outline' ? 'border-gray-300' : 'border-transparent',
    ],
  }[variant]

  return [
    sizeClasses,
    paddingClasses,
    variantClasses,
    props.disabled ? 'text-gray-600' : 'text-gray-800',
    'transition-colors w-full block',
  ]
})

let emitChange = (value: string) => {
  emit('update:modelValue', value)
}
if (props.debounce) {
  emitChange = debounce(emitChange, props.debounce)
}

let handleChange = (e: Event) => {
  emitChange((e.target as HTMLInputElement).value)
}
</script>
<style>
.outline-border {
  border: 1px solid #898989;
}
</style>

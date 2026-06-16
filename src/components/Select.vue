<template>
  <div class="relative flex items-center">
    <div
      :class="[
        'absolute inset-y-0 left-0 flex items-center',
        textColor,
        prefixClasses,
      ]"
      v-if="$slots.prefix"
    >
      <slot name="prefix"> </slot>
    </div>
    <div
      v-if="placeholder"
      v-show="!modelValue"
      class="pointer-events-none absolute text-gray-500"
      :class="[fontSizeClasses, paddingClasses]"
    >
      {{ placeholder }}
    </div>
    <select
      :class="selectClasses"
      :disabled="disabled"
      :id="id"
      :multiple="multiple"
      :value="multiple ? null : activeValue"
      @change="handleChange"
      v-bind="attrs"
    >
      <option
        v-for="option in selectOptions"
        :key="option.value"
        :value="option.value"
        :disabled="option.disabled || false"
        :selected="
          multiple
            ? Array.isArray(activeValue) && activeValue.includes(option.value)
            : activeValue === option.value
        "
      >
        {{ option.label }}
      </option>
    </select>
  </div>
</template>

<script setup lang="ts">
import { computed, useSlots, useAttrs } from 'vue'

defineOptions({
  inheritAttrs: false,
})

type SelectOption =
  | string
  | {
      label: string
      value: string
      disabled?: boolean
    }

interface SelectProps {
  size?: 'sm' | 'md' | 'lg'
  variant?: 'subtle' | 'outline' | 'ghost'
  placeholder?: string
  disabled?: boolean
  id?: string
  modelValue?: string | number | Array<string | number>
  options?: SelectOption[]
  multiple?: boolean
}

const props = withDefaults(defineProps<SelectProps>(), {
  size: 'sm',
  variant: 'subtle',
  multiple: false,
})

const emit = defineEmits(['update:modelValue'])
const slots = useSlots()
const attrs = useAttrs()

function handleChange(e: Event) {
  const target = e.target as HTMLSelectElement
  if (props.multiple) {
    const selectedValues = Array.from(target.selectedOptions).map(
      (option) => option.value
    )
    emit('update:modelValue', selectedValues)
  } else {
    emit('update:modelValue', target.value)
  }
}

const activeValue = computed(() => {
  if (props.multiple) {
    if (Array.isArray(props.modelValue)) {
      return props.modelValue
    }
    if (attrs.value) {
      return Array.isArray(attrs.value)
        ? attrs.value
        : String(attrs.value).split(',').map((v) => v.trim())
    }
    return []
  }
  if (props.modelValue !== undefined && props.modelValue !== null) {
    return props.modelValue
  }
  return attrs.value
})

const selectOptions = computed(() => {
  // Handle Frappe's newline-separated options string (e.g. from SectionFields)
  let rawOptions: SelectOption[] | undefined = props.options
  if (typeof rawOptions === 'string') {
    rawOptions = (rawOptions as string).split('\n').filter(Boolean) as SelectOption[]
  }

  if (!rawOptions || rawOptions.length === 0) return []

  const mapped = rawOptions
    .map((option) => {
      if (typeof option === 'string') {
        return { label: option, value: option }
      }
      return option
    })
    .filter(Boolean) as { label: string; value: string; disabled?: boolean }[]

  // Keep leading empty/placeholder option pinned at top; sort everything else
  if (
    mapped.length > 1 &&
    (mapped[0].value === '' ||
      mapped[0].value === null ||
      mapped[0].value === undefined)
  ) {
    const [first, ...rest] = mapped
    return [
      first,
      ...rest.sort((a, b) => String(a.label).localeCompare(String(b.label))),
    ]
  }
  return [...mapped].sort((a, b) => String(a.label).localeCompare(String(b.label)))
})

const textColor = computed(() => {
  return props.disabled ? 'text-gray-500' : 'text-gray-800'
})

const fontSizeClasses = computed(() => {
  return {
    sm: 'text-base',
    md: 'text-base',
    lg: 'text-lg',
    xl: 'text-xl',
  }[props.size]
})

const paddingClasses = computed(() => {
  return {
    sm: 'px-2',
    md: 'px-2.5',
    lg: 'px-3',
    xl: 'px-3',
  }[props.size]
})

const selectClasses = computed(() => {
  let sizeClasses = {
    sm: 'rounded h-7',
    md: 'rounded h-8',
    lg: 'rounded-md h-10',
    xl: 'rounded-md h-10',
  }[props.size]

  let variant = props.disabled ? 'disabled' : props.variant
  let variantClasses = {
    subtle:
      'outline-border !h-[45px] !rounded-[8px] bg-surface-white hover:border-outline-gray-3 focus:border-outline-gray-4 focus:ring-0 focus-visible:ring-2 focus-visible:ring-outline-gray-3  dark:!bg-[#101213] dark:!text-[white] dark:!border-[#898989!important]',
    outline:
      'outline-border !h-[45px] !rounded-[8px] bg-surface-white hover:border-outline-gray-3 focus:border-outline-gray-4 focus:ring-0 focus-visible:ring-2 focus-visible:ring-outline-gray-3  dark:!bg-[#101213] dark:!text-[white] dark:!border-[#898989!important]',
    ghost:
      'bg-transparent border-transparent hover:bg-gray-200 focus:bg-gray-200 focus:border-gray-500 focus:ring-0 focus-visible:ring-2 focus-visible:ring-gray-400',
    disabled: [
      'border',
      props.variant !== 'ghost' ? 'bg-gray-50' : '',
      props.variant === 'outline'
        ? 'outline-border !h-[45px] !rounded-[8px] bg-surface-white hover:border-outline-gray-3 focus:border-outline-gray-4 focus:ring-0 focus-visible:ring-2 focus-visible:ring-outline-gray-3  dark:!bg-[#101213] dark:!text-[white] dark:!border-[#898989!important]'
        : 'border-transparent',
    ],
  }[variant]

  return [
    sizeClasses,
    fontSizeClasses.value,
    paddingClasses.value,
    variantClasses,
    textColor.value,
    'transition-colors w-full py-0',
  ]
})

let prefixClasses = computed(() => {
  return {
    sm: 'pl-2',
    md: 'pl-2.5',
    lg: 'pl-3',
    xl: 'pl-3',
  }[props.size]
})
</script>
<style>
.outline-border {
  border: 1px solid #898989;
}

select option {
  border-bottom: 1px solid #dcdcdc;
  font-size: 20px !important;
  font-family: 'Montserrat', sans-serif;
}

:root.dark select option {
  background-color: #101213;
  border-bottom: 1px solid #555;
  color: white;
}
</style>

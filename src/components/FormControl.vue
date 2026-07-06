<template>
  <div v-if="type != 'checkbox'" :class="['space-y-1.5', attrs.class]">
    <label class="block" :class="labelClasses" v-if="label" :for="id">
      {{ label }}
    </label>
    <Select
      v-if="type === 'select'"
      :id="id"
      :options="sortedOptions"
      v-bind="{ ...controlAttrs, size }"
    >
      <template #prefix v-if="$slots.prefix">
        <slot name="prefix" />
      </template>
    </Select>
    <Autocomplete
      v-else-if="type === 'autocomplete'"
      :options="sortedOptions"
      v-bind="{ ...controlAttrs }"
    >
      <template #prefix v-if="$slots.prefix">
        <slot name="prefix" />
      </template>
      <template #item-prefix="itemPrefixProps" v-if="$slots['item-prefix']">
        <slot name="item-prefix" v-bind="itemPrefixProps" />
      </template>
    </Autocomplete>
    <Textarea
      v-else-if="type === 'textarea'"
      :id="id"
      v-bind="{ ...controlAttrs, size }"
    />
    <TextInput v-else :id="id" v-bind="{ ...controlAttrs, type, size }">
      <template #prefix v-if="$slots.prefix">
        <slot name="prefix" />
      </template>
      <template #suffix v-if="$slots.suffix">
        <slot name="suffix" />
      </template>
    </TextInput>
    <slot name="description">
      <p v-if="description" :class="descriptionClasses">{{ description }}</p>
    </slot>
  </div>
  <Checkbox
    v-else
    :id="id"
    v-bind="{ ...controlAttrs, label, size, class: attrs.class }"
  />
</template>
<script setup lang="ts">
import { useAttrs, computed } from 'vue'
import { useId } from '../utils/useId'
import TextInput from './TextInput.vue'
import type { TextInputTypes } from './types/TextInput'
import Select from './Select.vue'
import Textarea from './Textarea.vue'
import Checkbox from './Checkbox.vue'
import Autocomplete from './Autocomplete.vue'

type RawOption = string | { label: string; value: string; disabled?: boolean }

interface FormControlProps {
  label?: string
  description?: string
  type?: TextInputTypes | 'textarea' | 'select' | 'checkbox' | 'autocomplete'
  size?: 'sm' | 'md'
  options?: string | RawOption[]
}

const id = useId()
const props = withDefaults(defineProps<FormControlProps>(), {
  type: 'text',
  size: 'sm',
})

const attrs = useAttrs()
const controlAttrs = computed(() => {
  // pass everything except class, style, and options (handled separately)
  let _attrs: typeof attrs = {}
  for (let key in attrs) {
    if (key !== 'class' && key !== 'style' && key !== 'options') {
      _attrs[key] = attrs[key]
    }
  }
  return _attrs
})

const sortedOptions = computed(() => {
  let raw: RawOption[] = []
  if (typeof props.options === 'string') {
    raw = props.options.split('\n').filter(Boolean) as RawOption[]
  } else if (Array.isArray(props.options)) {
    raw = props.options
  } else {
    return []
  }

  const mapped = raw
    .map((o) => (typeof o === 'string' ? { label: o, value: o } : o))
    .filter(Boolean) as { label: string; value: string; disabled?: boolean }[]

  if (
    mapped.length > 1 &&
    (mapped[0].value === '' || mapped[0].value === null || mapped[0].value === undefined)
  ) {
    const [first, ...rest] = mapped
    return [first, ...rest.sort((a, b) => String(a.label).localeCompare(String(b.label)))]
  }
  return [...mapped].sort((a, b) => String(a.label).localeCompare(String(b.label)))
})

const labelClasses = computed(() => {
  return [
    {
      sm: 'text-xs',
      md: 'text-base',
    }[props.size],
    'text-gray-600',
  ]
})

const descriptionClasses = computed(() => {
  return [
    {
      sm: 'text-xs',
      md: 'text-base',
    }[props.size],
    'text-gray-600',
  ]
})
</script>
<script lang="ts">
export default {
  inheritAttrs: false,
}
</script>

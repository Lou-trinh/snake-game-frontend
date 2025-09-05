<script setup lang="ts">
  import { computed } from 'vue'

  const props = defineProps({
    title: {
      type: String,
      default: '',
    },
    modelValue: {
      type: String,
      default: 'Anonymous',
    },
  })

  const emit = defineEmits(['update:modelValue'])

  const name = computed({
    get: () => props.modelValue,
    set: (val: string) => {
      emit('update:modelValue', val)
    },
  })
</script>

<template>
  <div class="w-full relative mt-6">
    <label class="block mb-2 text-sm font-medium text-slate-700 dark:text-slate-300">
      {{ title }}
    </label>

    <div class="relative flex items-center">
      <!-- Input -->
      <input
        type="text"
        required
        v-model="name"
        class="w-full bg-white dark:bg-slate-900 text-slate-800 dark:text-slate-100
               border border-slate-300 dark:border-slate-600 rounded-lg
               px-3 pr-20 py-2 text-sm shadow-sm
               focus:outline-none focus:ring-2 focus:ring-purple-500 dark:focus:ring-purple-400"
      />
    </div>
  </div>
</template>

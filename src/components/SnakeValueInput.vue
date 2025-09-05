<script setup lang="ts">
  import { computed } from 'vue'

  const props = defineProps({
    title: {
      type: String,
      default: '',
    },
    modelValue: {
      type: Number,
      default: 10,
    },
    min: {
      type: Number,
      default: 10,
    },
    max: {
      type: Number,
      default: 10,
    },
  })

  const emit = defineEmits(['update:modelValue'])

  const amount = computed({
    get: () => props.modelValue,
    set: (val: number) => {
      const clamped = Math.min(props.max, Math.max(props.min, val))
      emit('update:modelValue', clamped)
    },
  })

  const increase = () => {
    amount.value = amount.value + 1
  }

  const decrease = () => {
    amount.value = amount.value - 1
  }
</script>

<template>
  <div class="w-full relative mt-6">
    <label class="block mb-2 text-sm font-medium text-slate-700 dark:text-slate-300">
      {{ title }}
    </label>

    <div class="relative flex items-center">
      <!-- Decrease Button -->
      <button
        @click="decrease"
        class="absolute right-10 rounded-lg bg-slate-900 dark:bg-slate-700 p-2 text-white
               shadow hover:bg-slate-800 dark:hover:bg-slate-600 transition disabled:opacity-40"
        type="button"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
          <path d="M3.75 7.25a.75.75 0 0 0 0 1.5h8.5a.75.75 0 0 0 0-1.5h-8.5Z" />
        </svg>
      </button>

      <!-- Input -->
      <input
        type="number"
        v-model="amount"
        class="w-full bg-white dark:bg-slate-900 text-slate-800 dark:text-slate-100
               border border-slate-300 dark:border-slate-600 rounded-lg
               px-3 pr-20 py-2 text-sm shadow-sm
               focus:outline-none focus:ring-2 focus:ring-purple-500 dark:focus:ring-purple-400"
      />

      <!-- Increase Button -->
      <button
        @click="increase"
        class="absolute right-1 rounded-lg bg-slate-900 dark:bg-slate-700 p-2 text-white
               shadow hover:bg-slate-800 dark:hover:bg-slate-600 transition disabled:opacity-40"
        type="button"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
          <path d="M8.75 3.75a.75.75 0 0 0-1.5 0v3.5h-3.5a.75.75 0 0 0 0 1.5h3.5v3.5a.75.75 0 0 0 1.5 0v-3.5h3.5a.75.75 0 0 0 0-1.5h-3.5v-3.5Z" />
        </svg>
      </button>
    </div>
  </div>
</template>

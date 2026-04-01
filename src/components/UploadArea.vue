<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  isLoading: boolean
  files: File[]
}>()

const emit = defineEmits<{
  (e: 'update:files', files: File[]): void
  (e: 'remove-file', index: number): void
  (e: 'process'): void
}>()

const dragActive = ref(false)
const fileInputRef = ref<HTMLInputElement | null>(null)

const triggerFileInput = () => fileInputRef.value?.click()

const handleDrop = (e: DragEvent) => {
  dragActive.value = false
  if (e.dataTransfer?.files?.length) {
    emit('update:files', [e.dataTransfer.files[0]])
  }
}

const handleFileSelect = (e: Event) => {
  const selected = (e.target as HTMLInputElement).files
  if (selected?.length) {
    emit('update:files', [selected[0]])
  }
}
</script>

<template>
  <div 
    @dragover.prevent="dragActive = true" 
    @dragleave.prevent="dragActive = false" 
    @drop.prevent="handleDrop"
    :class="['relative group overflow-hidden rounded-3xl border transition-all duration-500 max-w-2xl mx-auto', dragActive ? 'border-purple-500 bg-purple-500/10' : 'border-white/10 bg-white/5 backdrop-blur-xl']"
  >
    <div class="p-12 text-center space-y-6">
      <div class="relative flex justify-center" @click="triggerFileInput">
        <div class="p-6 rounded-2xl bg-white/5 border border-white/10 group-hover:scale-105 cursor-pointer transition-transform">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-12 h-12 text-purple-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
          </svg>
        </div>
      </div>
      <div class="space-y-2">
        <h3 class="text-2xl font-semibold text-white">Drop your file here</h3>
        <p class="text-gray-400/60">PDF, JPG, PNG (Max 10MB)</p>
      </div>
      <button 
        @click="triggerFileInput" 
        class="px-8 py-3 bg-white text-black font-semibold rounded-xl hover:bg-gray-200 transition-colors"
      >
        Select File
      </button>
      <input 
        type="file" 
        ref="fileInputRef" 
        class="hidden" 
        @change="handleFileSelect" 
        accept="image/*,application/pdf"
      >
    </div>

    <!-- File selection list -->
    <div v-if="files.length > 0" class="border-t border-white/10 p-6 bg-black/40">
      <div v-for="(file, idx) in files" :key="idx" class="flex items-center gap-3 p-3 rounded-lg bg-white/5 border border-white/10 mb-4 transition-all animate-in fade-in slide-in-from-bottom">
        <div class="flex-1 min-w-0 text-left">
          <p class="text-sm font-medium truncate text-gray-200">{{ file.name }}</p>
        </div>
        <button 
          @click="emit('remove-file', idx)" 
          class="text-gray-500 hover:text-red-400 text-sm font-medium transition-colors"
        >
          Remove
        </button>
      </div>

      <button 
        @click="emit('process')" 
        :disabled="isLoading"
        class="w-full py-4 bg-gradient-to-r from-purple-500 to-blue-500 rounded-xl font-bold text-white shadow-lg disabled:opacity-50 hover:brightness-110 active:scale-[0.98] transition-all"
      >
        <span v-if="isLoading" class="flex items-center justify-center gap-2">
          <div class="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin"></div>
          Processing OCR... (Please wait)
        </span>
        <span v-else>Start Processing OCR</span>
      </button>
    </div>
  </div>
</template>

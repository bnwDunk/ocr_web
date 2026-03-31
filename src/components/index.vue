<template>
  <div class="min-h-screen bg-[#07050d] text-gray-100 selection:bg-purple-500/30 overflow-x-hidden relative">
    <div class="absolute top-[-10%] left-[-10%] w-[40%] h-[40%] bg-purple-600/20 blur-[120px] rounded-full animate-pulse-slow"></div>
    <div class="absolute bottom-[-10%] right-[-10%] w-[35%] h-[35%] bg-blue-600/20 blur-[100px] rounded-full animate-pulse"></div>

    <div class="relative z-10 max-w-4xl mx-auto px-6 py-16">
      <header class="text-center mb-16 space-y-4">
        <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-purple-500/10 border border-purple-500/20 text-purple-400 text-sm font-medium mb-4">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10"/></svg>
          Secure & Internal OCR
        </div>
        <h1 class="text-4xl md:text-6xl font-bold tracking-tight bg-gradient-to-r from-white to-gray-500 bg-clip-text text-transparent">Extract Text Instantly</h1>
      </header>

      <div 
        @dragover.prevent="dragActive = true" @dragleave.prevent="dragActive = false" @drop.prevent="handleDrop"
        :class="['relative group overflow-hidden rounded-3xl border transition-all duration-500', dragActive ? 'border-purple-500 bg-purple-500/10' : 'border-white/10 bg-white/5 backdrop-blur-xl']"
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
          <button @click="triggerFileInput" class="px-8 py-3 bg-white text-black font-semibold rounded-xl hover:bg-gray-200 transition-colors">Select File</button>
          <input type="file" ref="fileInputRef" class="hidden" @change="handleFileSelect" accept="image/*,application/pdf">
        </div>

        <div v-if="files.length > 0" class="border-t border-white/10 p-6 bg-black/40 animate-slide-up">
          <div v-for="(file, idx) in files" :key="idx" class="flex items-center gap-3 p-3 rounded-lg bg-white/5 border border-white/10 mb-4">
            <div class="flex-1 min-w-0">
              <p class="text-sm font-medium truncate text-gray-200">{{ file.name }}</p>
            </div>
            <button @click="removeFile(idx)" class="text-gray-500 hover:text-red-400">Remove</button>
          </div>

          <button 
            @click="uploadAndProcess" :disabled="isUploading"
            class="w-full py-4 bg-gradient-to-r from-purple-500 to-blue-500 rounded-xl font-bold text-white shadow-lg disabled:opacity-50"
          >
            <span v-if="isUploading">Processing OCR... (Please wait)</span>
            <span v-else>Start Processing OCR</span>
          </button>
        </div>

        <div v-if="result" class="p-6 bg-emerald-500/10 border-t border-emerald-500/20 animate-fade-in flex justify-between items-center">
          <span class="text-emerald-400 text-sm">✓ Processed {{ result.total_rows }} rows</span>
          <button @click="downloadFile" class="px-4 py-2 bg-emerald-500 text-white text-sm font-bold rounded-lg hover:bg-emerald-600">Download Excel</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const API_BASE = import.meta.env.VITE_API_URL || "http://localhost:8000"
const dragActive = ref(false)
const files = ref<File[]>([])
const fileInputRef = ref<HTMLInputElement | null>(null)
const isUploading = ref(false)
const result = ref<any>(null)

const triggerFileInput = () => fileInputRef.value?.click()
const handleDrop = (e: DragEvent) => {
  dragActive.value = false
  if (e.dataTransfer?.files) files.value = [e.dataTransfer.files[0]]
}
const handleFileSelect = (e: Event) => {
  const selected = (e.target as HTMLInputElement).files
  if (selected) files.value = [selected[0]]
}
const removeFile = (idx: number) => { files.value.splice(idx, 1); result.value = null }

const uploadAndProcess = async () => {
  if (files.value.length === 0) return
  isUploading.value = true
  result.value = null

  const formData = new FormData()
  formData.append('file', files.value[0])

  try {
    const response = await fetch(`${API_BASE}/ocr`, { method: 'POST', body: formData })
    result.value = await response.json()
  } catch (err) {
    alert("Error connecting to server")
  } finally {
    isUploading.value = false
  }
}

const downloadFile = () => {
  if (result.value?.excel_download) {
    window.open(`${API_BASE}${result.value.excel_download}`, '_blank')
  }
}
</script>
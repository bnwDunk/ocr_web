<script setup lang="ts">
import { ref } from 'vue'
import BackgroundDecor from '../components/ui/BackgroundDecor.vue'
import OcrHeader from '../components/ui/OcrHeader.vue'
import UploadArea from '../components/UploadArea.vue'
import ResultPanel from '../components/ResultPanel.vue'

const API_BASE = import.meta.env.VITE_API_URL || "http://localhost:8000"

const files = ref<File[]>([])
const isUploading = ref(false)
const elapsedTime = ref(0)
const finalTime = ref<number | null>(null)
const result = ref<any>(null)

const handleFilesUpdate = (newFiles: File[]) => {
  files.value = newFiles
  result.value = null // Clear result when file changes
  finalTime.value = null
}

const handleRemoveFile = (idx: number) => {
  files.value.splice(idx, 1)
  result.value = null
  finalTime.value = null
}

const uploadAndProcess = async () => {
  if (files.value.length === 0) return
  isUploading.value = true
  result.value = null
  elapsedTime.value = 0
  finalTime.value = null

  const startTime = Date.now()
  const timerInterval = setInterval(() => {
    elapsedTime.value = Math.floor((Date.now() - startTime) / 1000)
  }, 1000)

  const formData = new FormData()
  formData.append('file', files.value[0])

  try {
    const response = await fetch(`${API_BASE}/ocr`, { 
      method: 'POST', 
      body: formData 
    })
    
    if (!response.ok) throw new Error("OCR Failed")
    
    result.value = await response.json()
    finalTime.value = Math.floor((Date.now() - startTime) / 1000)
  } catch (err) {
    console.error(err)
    alert("Error processing document. Please check if the backend is running.")
  } finally {
    clearInterval(timerInterval)
    isUploading.value = false
  }
}

const handleSaveSelected = async (selectedData: Record<string, string[][]>) => {
  try {
    const response = await fetch(`${API_BASE}/save-excel`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        filename: result.value.filename,
        selected_data: selectedData
      })
    })

    if (!response.ok) throw new Error("Save Failed")
    
    const saveResult = await response.json()
    if (saveResult.excel_download) {
      window.open(`${API_BASE}${saveResult.excel_download}`, '_blank')
    }
  } catch (err) {
    console.error(err)
    alert("Error generating Excel file.")
  }
}
</script>

<template>
  <div 
    class="min-h-screen bg-[#07050d] text-gray-100 selection:bg-purple-500/30 overflow-x-hidden relative"
  >
    <BackgroundDecor />

    <div class="relative z-10 max-w-4xl mx-auto px-6 py-16">
      <OcrHeader />

      <main class="space-y-4">
        <!-- Main upload logic -->
        <UploadArea 
          :files="files" 
          :isLoading="isUploading"
          @update:files="handleFilesUpdate"
          @remove-file="handleRemoveFile"
          @process="uploadAndProcess"
        />

        <!-- Loading & Timer -->
        <div v-if="isUploading" class="max-w-2xl mx-auto px-4 py-8 flex flex-col items-center gap-4 animate-pulse">
          <div class="p-4 rounded-full bg-purple-500/10 border border-purple-500/20">
            <svg class="w-10 h-10 text-purple-400 animate-spin" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path> 
            </svg>
          </div>
          <div class="flex flex-col items-center gap-1">
            <span class="text-sm font-medium text-purple-400 tracking-wide">Processing Document...</span>
            <span class="text-xs font-mono text-gray-500 bg-white/5 px-2 py-0.5 rounded border border-white/10 uppercase">
              Elapsed: {{ elapsedTime }}s
            </span>
          </div>
        </div>

        <!-- Result display logic -->
        <div v-if="result" class="animate-in fade-in slide-in-from-bottom duration-500">
          <div v-if="finalTime !== null" class="text-center mb-4 text-xs text-gray-500 font-medium">
            ⏱ Total processing time: {{ finalTime }} seconds
          </div>
          <ResultPanel 
            :result="result" 
            @save-selected="handleSaveSelected"
          />
        </div>
      </main>

      <footer class="mt-20 text-center text-gray-500/40 text-sm">
        <p>© 2026 Internal OCR Tool</p>
      </footer>
    </div>
  </div>
</template>

<style>
/* Global-like animations that might be used across components */
.animate-in {
  animation-duration: 0.5s;
  animation-fill-mode: both;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
.fade-in {
  animation-name: fadeIn;
}

@keyframes slideInFromBottom {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
.slide-in-from-bottom {
  animation-name: slideInFromBottom;
}
</style>

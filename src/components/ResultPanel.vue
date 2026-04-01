<script setup lang="ts">
const props = defineProps<{
  result: {
    total_rows: number
    excel_download: string
  }
}>()

const emit = defineEmits<{
  (e: 'download'): void
}>()
</script>

<template>
  <div 
    class="mt-8 p-6 bg-emerald-500/10 border border-emerald-500/20 rounded-2xl animate-in fade-in slide-in-from-bottom duration-500 flex flex-col sm:flex-row justify-between items-center gap-4"
  >
    <div class="flex items-center gap-3">
      <div class="p-2 rounded-full bg-emerald-500/20 text-emerald-400">
        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>
      </div>
      <div>
        <h4 class="text-emerald-400 font-semibold">Processing Complete</h4>
        <p class="text-emerald-400/60 text-sm">✓ Processed {{ result.total_rows }} rows</p>
      </div>
    </div>
    <button 
      @click="emit('download')" 
      class="px-6 py-2 bg-emerald-500 text-white font-bold rounded-xl hover:bg-emerald-600 active:scale-95 transition-all shadow-lg shadow-emerald-500/20"
    >
      Download Excel
    </button>
  </div>
</template>

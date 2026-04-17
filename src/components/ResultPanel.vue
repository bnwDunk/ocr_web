<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const props = defineProps<{
  result: {
    filename: string
    total_rows: number
    data: Record<string, string[][]>
  }
}>()

const emit = defineEmits<{
  (e: 'save-selected', selectedData: Record<string, string[][]>): void
}>()

// State for selected rows: Record<page_name, Record<row_index, boolean>>
const selection = ref<Record<string, boolean[]>>({})
const isSaving = ref(false)

// Initialize selection state
onMounted(() => {
  const initial: Record<string, boolean[]> = {}
  Object.keys(props.result.data).forEach(page => {
    initial[page] = props.result.data[page].map(() => true) // Default to all selected
  })
  selection.value = initial
})

const toggleAllRows = (page: string, value: boolean) => {
  if (selection.value[page]) {
    selection.value[page] = selection.value[page].map(() => value)
  }
}

const handleSave = async () => {
  const selectedData: Record<string, string[][]> = {}
  
  Object.keys(props.result.data).forEach(page => {
    const selectedRows = props.result.data[page].filter((_, idx) => selection.value[page][idx])
    if (selectedRows.length > 0) {
      selectedData[page] = selectedRows
    }
  })

  if (Object.keys(selectedData).length === 0) {
    alert("Please select at least one row.")
    return
  }

  isSaving.value = true
  await emit('save-selected', selectedData)
  isSaving.value = false
}
</script>

<template>
  <div class="mt-8 space-y-6">
    <!-- Summary Header -->
    <div 
      class="p-6 bg-emerald-500/10 border border-emerald-500/20 rounded-2xl flex flex-col sm:flex-row justify-between items-center gap-4"
    >
      <div class="flex items-center gap-3">
        <div class="p-2 rounded-full bg-emerald-500/20 text-emerald-400">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>
        </div>
        <div>
          <h4 class="text-emerald-400 font-semibold text-lg">Scan Results</h4>
          <p class="text-emerald-400/60 text-sm">Review and select rows to export to Excel</p>
        </div>
      </div>
      <button 
        @click="handleSave"
        :disabled="isSaving"
        class="px-8 py-3 bg-emerald-500 text-white font-bold rounded-xl hover:bg-emerald-600 active:scale-95 transition-all shadow-lg shadow-emerald-500/20 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2"
      >
        <span v-if="isSaving">Saving...</span>
        <template v-else>
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>
          Download Selected (.xlsx)
        </template>
      </button>
    </div>

    <!-- Data Tables -->
    <div class="space-y-8">
      <div v-for="(rows, pageName) in result.data" :key="pageName" class="bg-white/5 border border-white/10 rounded-2xl overflow-hidden">
        <div class="px-6 py-4 bg-white/5 border-b border-white/10 flex justify-between items-center">
          <h5 class="text-purple-400 font-medium flex items-center gap-2">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>
            {{ pageName }}
          </h5>
          <div class="flex items-center gap-2">
            <button 
              @click="toggleAllRows(pageName, true)"
              class="text-xs text-gray-400 hover:text-white transition-colors"
            >Select All</button>
            <span class="text-gray-700">|</span>
            <button 
              @click="toggleAllRows(pageName, false)"
              class="text-xs text-gray-400 hover:text-white transition-colors"
            >Clear All</button>
          </div>
        </div>

        <div class="overflow-x-auto">
          <table class="w-full text-left text-sm">
            <thead>
              <tr class="border-b border-white/5 bg-white/5">
                <th class="px-6 py-3 w-10 text-center">#</th>
                <th class="px-6 py-3">Content</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="(row, idx) in rows" :key="idx"
                class="border-b border-white/5 hover:bg-white/10 transition-colors group"
                :class="{'bg-purple-500/5': selection[pageName] && selection[pageName][idx]}"
              >
                <td class="px-6 py-3 text-center">
                  <input 
                    type="checkbox" 
                    v-if="selection[pageName]"
                    v-model="selection[pageName][idx]"
                    class="rounded border-gray-600 bg-gray-800 text-purple-500 focus:ring-purple-500/50"
                  />
                </td>
                <td class="px-6 py-3">
                  <div class="flex flex-wrap gap-2">
                    <span 
                      v-for="(col, cIdx) in row" :key="cIdx"
                      class="px-2 py-0.5 bg-white/10 rounded border border-white/10 text-gray-300 group-hover:border-purple-500/30 group-hover:text-white transition-colors"
                    >
                      {{ col }}
                    </span>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

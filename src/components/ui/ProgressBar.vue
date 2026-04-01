<script setup lang="ts">
/**
 * ProgressBar.vue
 * A sleek, high-end progress bar component built with Tailwind CSS.
 * Features a shimmer effect and smooth transitions.
 */
defineProps<{
  progress: number // 0 to 100
  label?: string
}>()
</script>

<template>
  <div class="w-full space-y-2 animate-in fade-in slide-in-from-top duration-300">
    <!-- Header: Label and Percentage -->
    <div class="flex justify-between items-end">
      <span class="text-xs font-semibold text-purple-400/80 uppercase tracking-wider">
        {{ label || 'Processing OCR' }}
      </span>
      <span class="text-xs font-mono text-purple-400 font-bold bg-purple-500/10 px-2 py-0.5 rounded border border-purple-500/20">
        {{ Math.round(progress) }}%
      </span>
    </div>
    
    <!-- Progress Track -->
    <div class="h-2 w-full bg-white/5 rounded-full overflow-hidden border border-white/5 shadow-inner relative">
      <!-- Active Progress Bar -->
      <div 
        class="h-full bg-gradient-to-r from-purple-600 via-blue-500 to-purple-500 transition-all duration-300 ease-out shadow-[0_0_15px_rgba(168,85,247,0.4)] relative"
        :style="{ width: `${progress}%` }"
      >
        <!-- Dynamic Shimmer/Light effect -->
        <div class="absolute inset-0 w-full h-full">
          <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/30 to-transparent -translate-x-[200%] animate-shimmer"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}
.animate-shimmer {
  animation: shimmer 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
</style>

<template>
  <div class="fixed inset-0 pointer-events-none overflow-hidden">
    <!-- Purple Pulse -->
    <div 
      class="absolute top-[-10%] left-[-10%] w-[40%] h-[40%] bg-purple-600/20 blur-[120px] rounded-full animate-pulse-slow"
    ></div>
    <!-- Blue Pulse -->
    <div 
      class="absolute bottom-[-10%] right-[-10%] w-[35%] h-[35%] bg-blue-600/20 blur-[100px] rounded-full animate-pulse"
    ></div>
  </div>
</template>

<style scoped>
@keyframes pulse-slow {
  0%, 100% { opacity: 0.3; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.1); }
}
.animate-pulse-slow {
  animation: pulse-slow 8s ease-in-out infinite;
}
</style>

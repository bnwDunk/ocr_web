<template>
  <header class="text-center mb-16 space-y-4">
    <div 
      class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-purple-500/10 border border-purple-500/20 text-purple-400 text-sm font-medium mb-4"
    >
      <slot name="badge">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10"/>
        </svg>
        Secure & Internal OCR
      </slot>
    </div>
    <h1 
      class="text-4xl md:text-6xl font-bold tracking-tight bg-gradient-to-r from-white to-gray-500 bg-clip-text text-transparent"
    >
      <slot name="title">Extract Text Instantly</slot>
    </h1>
  </header>
</template>

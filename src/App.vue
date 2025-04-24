<script setup lang="ts">
import ChatBot from './components/ChatBot.vue'
</script>

<template>
  <main class="min-h-screen w-full bg-gradient-to-br from-slate-50 to-slate-100 overflow-hidden">
    <div class="max-w-[1440px] mx-auto px-4 sm:px-6">
      <ChatBot />
    </div>
  </main>
</template>

<style>
@import './assets/main.css';

:root {
  --color-primary: #3b82f6;
  --color-primary-dark: #2563eb;
  --color-secondary: #64748b;
  --color-text: #334155;
  --color-heading: #0f172a;
  --color-background: #ffffff;
  --color-background-alt: #f8fafc;
  --color-border: #e2e8f0;
  --color-success: #10b981;
  --color-error: #ef4444;
  --color-warning: #f59e0b;
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-md: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --section-gap: 160px;
}

html,
body {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

body {
  min-height: 100vh;
  color: var(--color-text);
  background: var(--color-background-alt);
  font-family:
    'Inter',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Fira Sans',
    'Droid Sans',
    'Helvetica Neue',
    sans-serif;
  font-size: 15px;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}

button {
  transition: all 0.2s ease;
}

*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
</style>

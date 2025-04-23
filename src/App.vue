<script setup lang="ts">
import ChatBot from './components/ChatBot.vue'
</script>

<template>
  <main class="min-h-screen w-full bg-gray-50 overflow-hidden">
    <ChatBot />
  </main>
</template>

<style>
@import './assets/main.css';

:root {
  --color-text: #213547;
  --color-heading: #0f172a;
  --color-background: #ffffff;
  --color-border: #e5e7eb;
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
  background: var(--color-background);
  font-family:
    Inter,
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
</style>

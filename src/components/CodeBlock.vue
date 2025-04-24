<script setup lang="ts">
import { ref, computed, watchEffect } from 'vue'
import { marked } from 'marked'

const props = defineProps<{
  content: string
  isThinkingModel?: boolean
}>()

const showCodeBlocks = ref<Record<number, boolean>>({})
const copySuccess = ref<Record<number, boolean>>({})
const showThinking = ref(false)

// Check if the response has a thinking section (for claude-3.7-sonnet-thinking)
const hasThinkingSection = computed(() => {
  return (
    props.isThinkingModel &&
    props.content.includes('Thinking:') &&
    props.content.includes('Answer:')
  )
})

// Split content into thinking and answer sections
const { thinking, answer } = computed(() => {
  if (!hasThinkingSection.value) {
    return { thinking: '', answer: props.content }
  }

  const parts = props.content.split('Answer:')
  if (parts.length < 2) {
    return { thinking: '', answer: props.content }
  }

  const thinkingPart = parts[0].replace('Thinking:', '').trim()
  const answerPart = parts.slice(1).join('Answer:').trim()

  return { thinking: thinkingPart, answer: answerPart }
}).value

// Process the content to identify and format code blocks
const processContent = (content: string) => {
  const lines = content.split('\n')
  const result = []

  let inCodeBlock = false
  let currentBlock = []
  let blockIndex = 0
  let language = ''

  for (let i = 0; i < lines.length; i++) {
    const line = lines[i]

    if (line.includes('```')) {
      if (!inCodeBlock) {
        // Start of code block
        inCodeBlock = true
        currentBlock = []
        blockIndex++
        // Extract language if specified (e.g., ```javascript)
        language = line.replace('```', '').trim()
      } else {
        // End of code block
        inCodeBlock = false
        // Add the block to the result
        result.push({
          type: 'code',
          content: currentBlock.join('\n'),
          index: blockIndex,
          language,
        })
        language = ''
      }
    } else if (inCodeBlock) {
      // Add line to current code block
      currentBlock.push(line)
    } else {
      // Regular text - process markdown
      result.push({
        type: 'markdown',
        content: line,
      })
    }
  }

  return { result, blockCount: blockIndex }
}

const formattedAnswer = computed(() => processContent(answer))
const formattedThinking = computed(() => processContent(thinking))

// Combine both for block index tracking
const totalBlockCount = computed(() => {
  return formattedThinking.value.blockCount + formattedAnswer.value.blockCount
})

// Initialize showCodeBlocks when content changes
watchEffect(() => {
  const total = totalBlockCount.value
  for (let i = 1; i <= total; i++) {
    if (showCodeBlocks.value[i] === undefined) {
      showCodeBlocks.value[i] = false
    }
    copySuccess.value[i] = false
  }
})

// Toggle code block visibility
function toggleCodeBlock(index: number | undefined) {
  if (index !== undefined) {
    showCodeBlocks.value[index] = !showCodeBlocks.value[index]
  }
}

// Toggle thinking visibility
function toggleThinking() {
  showThinking.value = !showThinking.value
}

// Copy code to clipboard
function copyToClipboard(text: string, index: number | undefined) {
  if (index === undefined) return

  navigator.clipboard
    .writeText(text)
    .then(() => {
      copySuccess.value[index] = true
      setTimeout(() => {
        copySuccess.value[index] = false
      }, 2000)
    })
    .catch((err) => {
      console.error('Failed to copy code: ', err)
    })
}

// Render markdown content
function renderMarkdown(text: string) {
  return marked(text)
}
</script>

<template>
  <div class="code-wrapper">
    <!-- Thinking & Answer accordion for claude-3.7-sonnet-thinking model -->
    <div v-if="hasThinkingSection" class="thinking-section">
      <div class="thinking-toggle" @click="toggleThinking">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="thinking-icon"
          :class="{ 'rotate-180': showThinking }"
          viewBox="0 0 20 20"
          fill="currentColor"
        >
          <path
            fill-rule="evenodd"
            d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z"
            clip-rule="evenodd"
          />
        </svg>
        <span>{{ showThinking ? 'Hide' : 'Show' }} Thinking Process</span>
      </div>

      <div v-if="showThinking" class="thinking-content">
        <template v-for="(part, index) in formattedThinking.result" :key="'thinking-' + index">
          <!-- Markdown text -->
          <div
            v-if="part.type === 'markdown'"
            class="thinking-text"
            v-html="renderMarkdown(part.content)"
          ></div>

          <!-- Code block -->
          <div v-else class="code-block-container">
            <div class="code-header">
              <div class="code-header-left">
                <button @click="toggleCodeBlock(part.index)" class="code-toggle">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="toggle-icon"
                    :class="{ 'rotate-90': part.index !== undefined && showCodeBlocks[part.index] }"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                  >
                    <path
                      fill-rule="evenodd"
                      d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                      clip-rule="evenodd"
                    />
                  </svg>
                  {{ part.index !== undefined && showCodeBlocks[part.index] ? 'Hide' : 'Show' }}
                  Code
                </button>
                <span v-if="part.language" class="language-tag">{{ part.language }}</span>
              </div>
              <button @click="copyToClipboard(part.content, part.index)" class="copy-button">
                <svg
                  v-if="part.index !== undefined && copySuccess[part.index]"
                  xmlns="http://www.w3.org/2000/svg"
                  class="check-icon"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path
                    fill-rule="evenodd"
                    d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                    clip-rule="evenodd"
                  />
                </svg>
                <svg
                  v-else
                  xmlns="http://www.w3.org/2000/svg"
                  class="copy-icon"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path d="M8 2a1 1 0 000 2h2a1 1 0 100-2H8z" />
                  <path
                    d="M3 5a2 2 0 012-2 3 3 0 003 3h2a3 3 0 003-3 2 2 0 012 2v6h-4.586l1.293-1.293a1 1 0 00-1.414-1.414l-3 3a1 1 0 000 1.414l3 3a1 1 0 001.414-1.414L10.414 13H15v3a2 2 0 01-2 2H5a2 2 0 01-2-2V5zM15 11h2a1 1 0 110 2h-2v-2z"
                  />
                </svg>
                {{ part.index !== undefined && copySuccess[part.index] ? 'Copied!' : 'Copy' }}
              </button>
            </div>
            <pre
              v-if="part.index !== undefined && showCodeBlocks[part.index]"
              class="code-content"
            ><code>{{ part.content }}</code></pre>
          </div>
        </template>
      </div>
    </div>

    <!-- Answer section (always visible) -->
    <div class="answer-section">
      <template v-for="(part, index) in formattedAnswer.result" :key="'answer-' + index">
        <!-- Markdown text -->
        <div
          v-if="part.type === 'markdown'"
          class="markdown-content"
          v-html="renderMarkdown(part.content)"
        ></div>

        <!-- Code block -->
        <div v-else class="code-block-container">
          <div class="code-header">
            <div class="code-header-left">
              <button @click="toggleCodeBlock(part.index)" class="code-toggle">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="toggle-icon"
                  :class="{ 'rotate-90': part.index !== undefined && showCodeBlocks[part.index] }"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path
                    fill-rule="evenodd"
                    d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                    clip-rule="evenodd"
                  />
                </svg>
                {{ part.index !== undefined && showCodeBlocks[part.index] ? 'Hide' : 'Show' }} Code
              </button>
              <span v-if="part.language" class="language-tag">{{ part.language }}</span>
            </div>
            <button @click="copyToClipboard(part.content, part.index)" class="copy-button">
              <svg
                v-if="part.index !== undefined && copySuccess[part.index]"
                xmlns="http://www.w3.org/2000/svg"
                class="check-icon"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                  clip-rule="evenodd"
                />
              </svg>
              <svg
                v-else
                xmlns="http://www.w3.org/2000/svg"
                class="copy-icon"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path d="M8 2a1 1 0 000 2h2a1 1 0 100-2H8z" />
                <path
                  d="M3 5a2 2 0 012-2 3 3 0 003 3h2a3 3 0 003-3 2 2 0 012 2v6h-4.586l1.293-1.293a1 1 0 00-1.414-1.414l-3 3a1 1 0 000 1.414l3 3a1 1 0 001.414-1.414L10.414 13H15v3a2 2 0 01-2 2H5a2 2 0 01-2-2V5zM15 11h2a1 1 0 110 2h-2v-2z"
                />
              </svg>
              {{ part.index !== undefined && copySuccess[part.index] ? 'Copied!' : 'Copy' }}
            </button>
          </div>
          <pre
            v-if="part.index !== undefined && showCodeBlocks[part.index]"
            class="code-content"
          ><code>{{ part.content }}</code></pre>
        </div>
      </template>
    </div>
  </div>
</template>

<style scoped>
.code-wrapper {
  color: rgb(51, 65, 85);
  width: 100%;
  display: block;
  background-color: transparent;
}

.thinking-section {
  margin-bottom: 1rem;
  border: 1px solid rgb(226, 232, 240);
  border-radius: 0.5rem;
  overflow: hidden;
  width: 100%;
  display: block;
}

.thinking-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem;
  background-color: rgb(248, 250, 252);
  color: rgb(71, 85, 105);
  font-weight: 500;
  cursor: pointer;
  transition-property: background-color;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 150ms;
}

.thinking-toggle:hover {
  background-color: rgb(241, 245, 249);
}

.thinking-icon {
  height: 1.25rem;
  width: 1.25rem;
  color: rgb(100, 116, 139);
  transition-property: transform;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

.thinking-icon.rotate-180 {
  transform: rotate(180deg);
}

.thinking-content {
  padding: 1rem;
  background-color: white;
  border-top: 1px solid rgb(226, 232, 240);
  display: block;
  width: 100%;
}

.thinking-text {
  margin-bottom: 0.75rem;
  display: block;
  width: 100%;
}

.code-block-container {
  margin-top: 1rem;
  margin-bottom: 1rem;
  border-radius: 0.5rem;
  border: 1px solid rgb(226, 232, 240);
  overflow: hidden;
  background-color: white;
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  width: 100%;
  display: block;
}

.code-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: rgb(248, 250, 252);
  padding-left: 0.75rem;
  padding-right: 0.75rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid rgb(226, 232, 240);
  width: 100%;
}

.code-header-left {
  display: flex;
  align-items: center;
}

.code-toggle {
  font-size: 0.875rem;
  line-height: 1.25rem;
  font-weight: 500;
  color: rgb(37, 99, 235);
  display: flex;
  align-items: center;
}

.code-toggle:hover {
  color: rgb(29, 78, 216);
}

.toggle-icon {
  height: 1rem;
  width: 1rem;
  margin-right: 0.25rem;
  transition-property: transform;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

.toggle-icon.rotate-90 {
  transform: rotate(90deg);
}

.language-tag {
  margin-left: 0.5rem;
  padding-left: 0.5rem;
  padding-right: 0.5rem;
  padding-top: 0.125rem;
  padding-bottom: 0.125rem;
  background-color: rgb(219, 234, 254);
  color: rgb(29, 78, 216);
  font-size: 0.75rem;
  line-height: 1rem;
  border-radius: 0.375rem;
}

.copy-button {
  font-size: 0.75rem;
  line-height: 1rem;
  display: flex;
  align-items: center;
  gap: 0.25rem;
  color: rgb(71, 85, 105);
  padding-left: 0.5rem;
  padding-right: 0.5rem;
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
  border-radius: 0.25rem;
  transition-property: background-color, color;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 150ms;
}

.copy-button:hover {
  color: rgb(15, 23, 42);
  background-color: rgb(241, 245, 249);
}

.copy-icon,
.check-icon {
  height: 0.875rem;
  width: 0.875rem;
}

.check-icon {
  color: rgb(34, 197, 94);
}

.code-content {
  padding: 1rem;
  overflow-x: auto;
  font-size: 0.875rem;
  line-height: 1.25rem;
  color: rgb(30, 41, 59);
  background-color: rgb(248, 250, 252);
  font-family:
    ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New',
    monospace;
  display: block;
  width: 100%;
  white-space: pre-wrap;
}

.markdown-content {
  margin-bottom: 0.75rem;
  width: 100%;
  display: block;
  white-space: normal;
  word-break: break-word;
}

.markdown-content :deep(a) {
  color: rgb(37, 99, 235);
}

.markdown-content :deep(a:hover) {
  text-decoration: underline;
}

.markdown-content :deep(h1) {
  font-size: 1.5rem;
  line-height: 2rem;
  font-weight: 700;
  margin-top: 1.5rem;
  margin-bottom: 1rem;
  display: block;
}

.markdown-content :deep(h2) {
  font-size: 1.25rem;
  line-height: 1.75rem;
  font-weight: 700;
  margin-top: 1.25rem;
  margin-bottom: 0.75rem;
  display: block;
}

.markdown-content :deep(h3) {
  font-size: 1.125rem;
  line-height: 1.75rem;
  font-weight: 600;
  margin-top: 1rem;
  margin-bottom: 0.5rem;
  display: block;
}

.markdown-content :deep(ul) {
  list-style-type: disc;
  padding-left: 1.25rem;
  margin-top: 0.75rem;
  margin-bottom: 0.75rem;
  display: block;
}

.markdown-content :deep(ol) {
  list-style-type: decimal;
  padding-left: 1.25rem;
  margin-top: 0.75rem;
  margin-bottom: 0.75rem;
  display: block;
}

.markdown-content :deep(blockquote) {
  padding-left: 1rem;
  border-left-width: 4px;
  border-color: rgb(226, 232, 240);
  font-style: italic;
  color: rgb(71, 85, 105);
  margin-top: 0.75rem;
  margin-bottom: 0.75rem;
  display: block;
}

.markdown-content :deep(p) {
  margin-bottom: 0.75rem;
  display: block;
}

.answer-section {
  color: rgb(51, 65, 85);
  width: 100%;
  display: block;
}
</style>

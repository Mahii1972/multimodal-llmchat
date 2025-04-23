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
function toggleCodeBlock(index: number) {
  showCodeBlocks.value[index] = !showCodeBlocks.value[index]
}

// Toggle thinking visibility
function toggleThinking() {
  showThinking.value = !showThinking.value
}

// Copy code to clipboard
function copyToClipboard(text: string, index: number) {
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
  <div class="whitespace-pre-wrap">
    <!-- Thinking & Answer accordion for claude-3.7-sonnet-thinking model -->
    <div v-if="hasThinkingSection" class="mb-4">
      <!-- Answer section (always visible) -->
      <div class="mb-2">
        <template v-for="(part, index) in formattedAnswer.result" :key="'answer-' + index">
          <!-- Markdown text -->
          <div
            v-if="part.type === 'markdown'"
            class="mb-2"
            v-html="renderMarkdown(part.content)"
          ></div>

          <!-- Code block -->
          <div
            v-else
            class="my-4 rounded-lg border border-blue-200 overflow-hidden bg-white shadow-sm"
          >
            <div
              class="flex justify-between items-center bg-gradient-to-r from-blue-50 to-blue-100 px-3 py-2 border-b border-blue-200"
            >
              <div class="flex items-center">
                <button
                  @click="toggleCodeBlock(part.index)"
                  class="text-sm font-medium text-blue-700 hover:text-blue-800 hover:underline flex items-center"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4 mr-1"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      v-if="showCodeBlocks[part.index]"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M19 9l-7 7-7-7"
                    />
                    <path
                      v-else
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M9 5l7 7-7 7"
                    />
                  </svg>
                  {{ showCodeBlocks[part.index] ? 'Hide' : 'Show' }} Code Block
                </button>

                <span
                  v-if="part.language"
                  class="ml-2 px-2 py-0.5 bg-blue-200 text-blue-800 text-xs rounded-md"
                >
                  {{ part.language }}
                </span>
              </div>

              <button
                @click="copyToClipboard(part.content, part.index)"
                class="text-sm font-medium hover:text-blue-800 flex items-center gap-1 transition-colors duration-200"
                :class="copySuccess[part.index] ? 'text-green-600' : 'text-blue-700'"
              >
                <svg
                  v-if="copySuccess[part.index]"
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-4 w-4"
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
                  class="h-4 w-4"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="currentColor"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
                  />
                </svg>
                {{ copySuccess[part.index] ? 'Copied!' : 'Copy' }}
              </button>
            </div>

            <div
              v-if="showCodeBlocks[part.index]"
              class="bg-gray-900 text-yellow-300 p-4 overflow-x-auto font-mono text-sm"
            >
              {{ part.content }}
            </div>
          </div>
        </template>
      </div>

      <!-- Thinking accordion -->
      <div class="border border-gray-200 rounded-lg mt-4">
        <button
          @click="toggleThinking"
          class="w-full px-4 py-3 text-left flex justify-between items-center bg-gray-50 hover:bg-gray-100 transition-colors duration-200 rounded-lg"
        >
          <span class="font-medium text-gray-700">Thinking Process</span>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5 text-gray-500 transition-transform duration-200"
            :class="showThinking ? 'rotate-180' : ''"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M19 9l-7 7-7-7"
            />
          </svg>
        </button>
        <div v-if="showThinking" class="p-4 bg-gray-50 rounded-b-lg">
          <template v-for="(part, index) in formattedThinking.result" :key="'thinking-' + index">
            <!-- Markdown text -->
            <div
              v-if="part.type === 'markdown'"
              class="mb-2"
              v-html="renderMarkdown(part.content)"
            ></div>

            <!-- Code block -->
            <div
              v-else
              class="my-4 rounded-lg border border-blue-200 overflow-hidden bg-white shadow-sm"
            >
              <div
                class="flex justify-between items-center bg-gradient-to-r from-blue-50 to-blue-100 px-3 py-2 border-b border-blue-200"
              >
                <div class="flex items-center">
                  <button
                    @click="toggleCodeBlock(formattedAnswer.blockCount + part.index)"
                    class="text-sm font-medium text-blue-700 hover:text-blue-800 hover:underline flex items-center"
                  >
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      class="h-4 w-4 mr-1"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        v-if="showCodeBlocks[formattedAnswer.blockCount + part.index]"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M19 9l-7 7-7-7"
                      />
                      <path
                        v-else
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M9 5l7 7-7 7"
                      />
                    </svg>
                    {{
                      showCodeBlocks[formattedAnswer.blockCount + part.index] ? 'Hide' : 'Show'
                    }}
                    Code Block
                  </button>

                  <span
                    v-if="part.language"
                    class="ml-2 px-2 py-0.5 bg-blue-200 text-blue-800 text-xs rounded-md"
                  >
                    {{ part.language }}
                  </span>
                </div>

                <button
                  @click="copyToClipboard(part.content, formattedAnswer.blockCount + part.index)"
                  class="text-sm font-medium hover:text-blue-800 flex items-center gap-1 transition-colors duration-200"
                  :class="
                    copySuccess[formattedAnswer.blockCount + part.index]
                      ? 'text-green-600'
                      : 'text-blue-700'
                  "
                >
                  <svg
                    v-if="copySuccess[formattedAnswer.blockCount + part.index]"
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4"
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
                    class="h-4 w-4"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
                    />
                  </svg>
                  {{ copySuccess[formattedAnswer.blockCount + part.index] ? 'Copied!' : 'Copy' }}
                </button>
              </div>

              <div
                v-if="showCodeBlocks[formattedAnswer.blockCount + part.index]"
                class="bg-gray-900 text-yellow-300 p-4 overflow-x-auto font-mono text-sm"
              >
                {{ part.content }}
              </div>
            </div>
          </template>
        </div>
      </div>
    </div>

    <!-- Regular content (no thinking section) -->
    <template v-else v-for="(part, index) in formattedAnswer.result" :key="index">
      <!-- Markdown text -->
      <div v-if="part.type === 'markdown'" class="mb-2" v-html="renderMarkdown(part.content)"></div>

      <!-- Code block -->
      <div v-else class="my-4 rounded-lg border border-blue-200 overflow-hidden bg-white shadow-sm">
        <div
          class="flex justify-between items-center bg-gradient-to-r from-blue-50 to-blue-100 px-3 py-2 border-b border-blue-200"
        >
          <div class="flex items-center">
            <button
              @click="toggleCodeBlock(part.index)"
              class="text-sm font-medium text-blue-700 hover:text-blue-800 hover:underline flex items-center"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4 mr-1"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  v-if="showCodeBlocks[part.index]"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 9l-7 7-7-7"
                />
                <path
                  v-else
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M9 5l7 7-7 7"
                />
              </svg>
              {{ showCodeBlocks[part.index] ? 'Hide' : 'Show' }} Code Block
            </button>

            <span
              v-if="part.language"
              class="ml-2 px-2 py-0.5 bg-blue-200 text-blue-800 text-xs rounded-md"
            >
              {{ part.language }}
            </span>
          </div>

          <button
            @click="copyToClipboard(part.content, part.index)"
            class="text-sm font-medium hover:text-blue-800 flex items-center gap-1 transition-colors duration-200"
            :class="copySuccess[part.index] ? 'text-green-600' : 'text-blue-700'"
          >
            <svg
              v-if="copySuccess[part.index]"
              xmlns="http://www.w3.org/2000/svg"
              class="h-4 w-4"
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
              class="h-4 w-4"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3"
              />
            </svg>
            {{ copySuccess[part.index] ? 'Copied!' : 'Copy' }}
          </button>
        </div>

        <div
          v-if="showCodeBlocks[part.index]"
          class="bg-gray-900 text-yellow-300 p-4 overflow-x-auto font-mono text-sm"
        >
          {{ part.content }}
        </div>
      </div>
    </template>
  </div>
</template>

<style scoped>
:deep(a) {
  color: #3b82f6;
  text-decoration: underline;
}

:deep(h1) {
  font-size: 1.5rem;
  font-weight: 600;
  margin-top: 1rem;
  margin-bottom: 0.5rem;
}

:deep(h2) {
  font-size: 1.25rem;
  font-weight: 600;
  margin-top: 0.75rem;
  margin-bottom: 0.5rem;
}

:deep(h3) {
  font-size: 1.125rem;
  font-weight: 600;
  margin-top: 0.75rem;
  margin-bottom: 0.5rem;
}

:deep(ul),
:deep(ol) {
  padding-left: 1.5rem;
  margin: 0.5rem 0;
}

:deep(ul) {
  list-style-type: disc;
}

:deep(ol) {
  list-style-type: decimal;
}

:deep(li) {
  margin: 0.25rem 0;
}

:deep(p) {
  margin: 0.5rem 0;
}

:deep(blockquote) {
  border-left: 4px solid #e5e7eb;
  padding-left: 1rem;
  margin: 0.5rem 0;
  color: #4b5563;
}

:deep(hr) {
  border: 0;
  border-top: 1px solid #e5e7eb;
  margin: 1rem 0;
}

:deep(code:not(pre code)) {
  background-color: #f3f4f6;
  padding: 0.125rem 0.25rem;
  border-radius: 0.25rem;
  font-family: monospace;
  font-size: 0.875em;
}

:deep(table) {
  width: 100%;
  border-collapse: collapse;
  margin: 0.5rem 0;
}

:deep(th),
:deep(td) {
  border: 1px solid #e5e7eb;
  padding: 0.5rem;
  text-align: left;
}

:deep(th) {
  background-color: #f9fafb;
  font-weight: 600;
}
</style>

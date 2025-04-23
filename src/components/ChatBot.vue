<script setup lang="ts">
import { ref, computed } from 'vue'
import CodeBlock from './CodeBlock.vue'

interface Message {
  role: 'user' | 'assistant'
  content: string
}

const selectedModel = ref<string>('')
const messages = ref<Message[]>([])
const inputMessage = ref<string>('')
const isLoading = ref<boolean>(false)
const sessionActive = ref<boolean>(false)
const showModelDropdown = ref<boolean>(false)

const models = [
  { id: 'claude-3.5-sonnet', name: 'Claude-3.5-Sonnet' },
  { id: 'o3-mini', name: 'O3-Mini' },
  { id: 'claude-3.7-sonnet', name: 'Claude-3.7-Sonnet' },
  { id: 'claude-3.7-sonnet-thinking', name: 'Claude-3.7-Sonnet-Thinking' },
]

// Check if the selected model is the thinking model
const isThinkingModel = computed(() => {
  return selectedModel.value === 'claude-3.7-sonnet-thinking'
})

// Get the webhook URL from environment variables
const webhookUrl = import.meta.env.VITE_WEBHOOK_URL

function startSession() {
  sessionActive.value = true
  messages.value = []
}

function endSession() {
  sessionActive.value = false
  messages.value = []
  selectedModel.value = ''
  showModelDropdown.value = false
}

function toggleModelDropdown() {
  showModelDropdown.value = !showModelDropdown.value
}

function changeModel(modelId: string) {
  selectedModel.value = modelId
  showModelDropdown.value = false
}

async function sendMessage() {
  if (!inputMessage.value.trim() || !selectedModel.value || !sessionActive.value) return

  const userMessage = inputMessage.value
  messages.value.push({ role: 'user', content: userMessage })
  inputMessage.value = ''
  isLoading.value = true

  try {
    const response = await fetch(webhookUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        question: userMessage,
        context: messages.value,
        model: selectedModel.value,
      }),
    })

    if (!response.ok) {
      throw new Error('Failed to get response')
    }

    const responseText = await response.text()
    messages.value.push({ role: 'assistant', content: responseText })
  } catch (error) {
    console.error('Error sending message:', error)
    messages.value.push({
      role: 'assistant',
      content: 'Sorry, there was an error processing your request.',
    })
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="w-full h-screen flex flex-col p-0 m-0 overflow-hidden">
    <div class="flex justify-center py-4 bg-white border-b">
      <h1 class="text-3xl font-bold text-blue-600 flex items-center">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-8 w-8 mr-2"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z"
          />
        </svg>
        Multi-Modal LLM Chat
      </h1>
    </div>

    <div v-if="!sessionActive" class="flex-1 flex items-center justify-center p-4">
      <div class="bg-white rounded-lg shadow-lg p-6 border border-gray-200 w-full max-w-xl">
        <h2 class="text-xl font-semibold mb-4 text-gray-800">Select a model to start chatting</h2>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
          <button
            v-for="model in models"
            :key="model.id"
            @click="selectedModel = model.id"
            class="p-4 border rounded-md transition-all duration-200 flex items-center"
            :class="
              selectedModel === model.id
                ? 'bg-blue-100 border-blue-500 text-blue-700 shadow-sm'
                : 'hover:bg-gray-50 border-gray-200'
            "
          >
            <svg
              v-if="selectedModel === model.id"
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5 mr-2 text-blue-500"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                fill-rule="evenodd"
                d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
                clip-rule="evenodd"
              />
            </svg>
            <span>{{ model.name }}</span>
          </button>
        </div>
        <button
          @click="startSession"
          :disabled="!selectedModel"
          class="w-full py-3 bg-blue-600 text-white rounded-md font-medium transition-colors duration-200 disabled:bg-gray-300 hover:bg-blue-700"
        >
          Start Chat Session
        </button>
      </div>
    </div>

    <div v-else class="flex-1 flex flex-col h-full overflow-hidden">
      <div
        class="flex justify-between items-center px-4 py-2 bg-white shadow-sm border-b border-gray-200 z-10"
      >
        <div class="text-sm font-medium flex items-center relative">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5 mr-1 text-blue-500"
            viewBox="0 0 20 20"
            fill="currentColor"
          >
            <path
              fill-rule="evenodd"
              d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z"
              clip-rule="evenodd"
            />
          </svg>
          Using model:
          <button
            @click="toggleModelDropdown"
            class="text-blue-600 ml-1 flex items-center gap-1 hover:underline focus:outline-none"
          >
            {{ models.find((m) => m.id === selectedModel)?.name || selectedModel }}
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-4 w-4"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
              :class="{ 'rotate-180': showModelDropdown }"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M19 9l-7 7-7-7"
              />
            </svg>
          </button>

          <!-- Dropdown menu -->
          <div
            v-if="showModelDropdown"
            class="absolute top-full left-0 mt-1 bg-white border border-gray-200 rounded-md shadow-lg z-20 w-64"
          >
            <ul>
              <li
                v-for="model in models"
                :key="model.id"
                @click="changeModel(model.id)"
                class="px-4 py-2 cursor-pointer hover:bg-blue-50 flex items-center"
                :class="{ 'bg-blue-50': model.id === selectedModel }"
              >
                <svg
                  v-if="model.id === selectedModel"
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-4 w-4 mr-2 text-blue-600"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path
                    fill-rule="evenodd"
                    d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                    clip-rule="evenodd"
                  />
                </svg>
                <span v-else class="w-4 mr-2"></span>
                {{ model.name }}
              </li>
            </ul>
          </div>
        </div>

        <button
          @click="endSession"
          class="px-4 py-1.5 bg-red-500 text-white text-sm rounded-md hover:bg-red-600 transition-colors duration-200 flex items-center"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-4 w-4 mr-1"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
          End Session
        </button>
      </div>

      <div class="flex-1 overflow-y-auto p-4 bg-gray-50">
        <div
          v-if="messages.length === 0"
          class="flex flex-col items-center justify-center h-full text-gray-500 py-4"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-12 w-12 mb-2 text-gray-400"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="1.5"
              d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"
            />
          </svg>
          <p>Type a message to start the conversation</p>
        </div>

        <div v-for="(message, index) in messages" :key="index" class="mb-4">
          <div
            class="flex items-start gap-2"
            :class="message.role === 'user' ? 'justify-end' : 'justify-start'"
          >
            <div
              v-if="message.role === 'assistant'"
              class="flex-shrink-0 w-8 h-8 bg-blue-100 rounded-full flex items-center justify-center"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-5 w-5 text-blue-600"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V5a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
                />
              </svg>
            </div>

            <div
              class="rounded-lg p-3 max-w-[85%] shadow-sm"
              :class="
                message.role === 'user'
                  ? 'bg-blue-500 text-white'
                  : 'bg-white text-gray-800 border border-gray-200'
              "
            >
              <CodeBlock
                v-if="message.role === 'assistant'"
                :content="message.content"
                :is-thinking-model="isThinkingModel"
              />
              <div v-else class="whitespace-pre-wrap">{{ message.content }}</div>
            </div>

            <div
              v-if="message.role === 'user'"
              class="flex-shrink-0 w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-5 w-5 text-white"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
                />
              </svg>
            </div>
          </div>
        </div>

        <div
          v-if="isLoading"
          class="flex items-center gap-2 text-blue-500 p-3 bg-blue-50 rounded-lg mb-4"
        >
          <div class="loading-spinner"></div>
          <span>Thinking...</span>
        </div>
      </div>

      <div class="p-4 bg-white border-t border-gray-200">
        <div class="relative">
          <textarea
            v-model="inputMessage"
            @keydown.ctrl.enter="sendMessage"
            placeholder="Type your message here... (Ctrl+Enter to send)"
            class="w-full p-4 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"
            rows="3"
          ></textarea>
          <button
            @click="sendMessage"
            class="absolute right-3 bottom-3 px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors duration-200 flex items-center"
            :disabled="isLoading || !inputMessage.trim()"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-4 w-4 mr-1"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13 5l7 7-7 7M5 5l7 7-7 7"
              />
            </svg>
            Send
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.loading-spinner {
  display: inline-block;
  width: 1rem;
  height: 1rem;
  border: 2px solid rgba(59, 130, 246, 0.3);
  border-radius: 50%;
  border-top-color: rgba(59, 130, 246, 1);
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>

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
  <div class="chat-container">
    <div class="header">
      <h1 class="title">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="icon"
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

    <div v-if="!sessionActive" class="welcome-container">
      <div class="model-selector">
        <h2 class="subtitle">Select a model to start chatting</h2>
        <div class="model-grid">
          <button
            v-for="model in models"
            :key="model.id"
            @click="selectedModel = model.id"
            class="model-button"
            :class="{ selected: selectedModel === model.id }"
          >
            <svg
              v-if="selectedModel === model.id"
              xmlns="http://www.w3.org/2000/svg"
              class="check-icon"
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
          class="start-button"
          :class="{ disabled: !selectedModel }"
        >
          Start Chat Session
        </button>
      </div>
    </div>

    <div v-else class="chat-session">
      <div class="model-header">
        <div class="model-info">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="info-icon"
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
          <button @click="toggleModelDropdown" class="model-selector-button">
            {{ models.find((m) => m.id === selectedModel)?.name || selectedModel }}
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="dropdown-icon"
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
          <div v-if="showModelDropdown" class="model-dropdown">
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
                  class="dropdown-check-icon"
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

        <button @click="endSession" class="end-button">End Session</button>
      </div>

      <div class="messages-container">
        <div v-if="messages.length === 0" class="empty-state">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="empty-icon"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"
            />
          </svg>
          <p class="empty-text">Start the conversation by sending a message</p>
        </div>

        <div v-else class="messages-list">
          <div
            v-for="(message, index) in messages"
            :key="index"
            class="message"
            :class="message.role"
          >
            <div class="message-content">
              <div v-if="message.role === 'user'" class="avatar user-avatar">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
                  <path
                    fill-rule="evenodd"
                    d="M7.5 6a4.5 4.5 0 119 0 4.5 4.5 0 01-9 0zM3.751 20.105a8.25 8.25 0 0116.498 0 .75.75 0 01-.437.695A18.683 18.683 0 0112 22.5c-2.786 0-5.433-.608-7.812-1.7a.75.75 0 01-.437-.695z"
                    clip-rule="evenodd"
                  />
                </svg>
              </div>
              <div v-else class="avatar assistant-avatar">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M16.5 7.5h-9v9h9v-9z" />
                  <path
                    fill-rule="evenodd"
                    d="M8.25 2.25A.75.75 0 019 3v.75h2.25V3a.75.75 0 011.5 0v.75H15V3a.75.75 0 011.5 0v.75h.75a3 3 0 013 3v.75H21A.75.75 0 0121 9h-.75v2.25H21a.75.75 0 010 1.5h-.75V15H21a.75.75 0 010 1.5h-.75v.75a3 3 0 01-3 3h-.75V21a.75.75 0 01-1.5 0v-.75h-2.25V21a.75.75 0 01-1.5 0v-.75H9V21a.75.75 0 01-1.5 0v-.75h-.75a3 3 0 01-3-3v-.75H3A.75.75 0 013 15h.75v-2.25H3a.75.75 0 010-1.5h.75V9H3a.75.75 0 010-1.5h.75v-.75a3 3 0 013-3h.75V3a.75.75 0 01.75-.75zM6 6.75A.75.75 0 016.75 6h10.5a.75.75 0 01.75.75v10.5a.75.75 0 01-.75.75H6.75a.75.75 0 01-.75-.75V6.75z"
                    clip-rule="evenodd"
                  />
                </svg>
              </div>
              <div class="message-body">
                <div class="message-header">
                  <span class="role-label">{{
                    message.role === 'user' ? 'You' : 'Assistant'
                  }}</span>
                </div>
                <div v-if="message.role === 'assistant'" class="assistant-content">
                  <CodeBlock :content="message.content" :is-thinking-model="isThinkingModel" />
                </div>
                <div v-else class="whitespace-pre-wrap">{{ message.content }}</div>
              </div>
            </div>
          </div>

          <!-- Loading indicator bubble -->
          <div v-if="isLoading" class="message assistant">
            <div class="message-content">
              <div class="avatar assistant-avatar">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M16.5 7.5h-9v9h9v-9z" />
                  <path
                    fill-rule="evenodd"
                    d="M8.25 2.25A.75.75 0 019 3v.75h2.25V3a.75.75 0 011.5 0v.75H15V3a.75.75 0 011.5 0v.75h.75a3 3 0 013 3v.75H21A.75.75 0 0121 9h-.75v2.25H21a.75.75 0 010 1.5h-.75V15H21a.75.75 0 010 1.5h-.75v.75a3 3 0 01-3 3h-.75V21a.75.75 0 01-1.5 0v-.75h-2.25V21a.75.75 0 01-1.5 0v-.75H9V21a.75.75 0 01-1.5 0v-.75h-.75a3 3 0 01-3-3v-.75H3A.75.75 0 013 15h.75v-2.25H3a.75.75 0 010-1.5h.75V9H3a.75.75 0 010-1.5h.75v-.75a3 3 0 013-3h.75V3a.75.75 0 01.75-.75zM6 6.75A.75.75 0 016.75 6h10.5a.75.75 0 01.75.75v10.5a.75.75 0 01-.75.75H6.75a.75.75 0 01-.75-.75V6.75z"
                    clip-rule="evenodd"
                  />
                </svg>
              </div>
              <div class="message-body">
                <div class="message-header">
                  <span class="role-label">Assistant</span>
                </div>
                <div class="assistant-content loading-content">
                  <div class="loading-dots">
                    <span></span>
                    <span></span>
                    <span></span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="input-container">
        <div class="input-wrapper">
          <textarea
            v-model="inputMessage"
            @keydown.enter="
              (event) => {
                if (!event.shiftKey) {
                  event.preventDefault()
                  sendMessage()
                }
              }
            "
            placeholder="Type your message..."
            class="message-input"
            :disabled="isLoading"
          ></textarea>
          <button
            @click="sendMessage"
            class="send-button"
            :disabled="!inputMessage.trim() || isLoading"
          >
            <svg
              v-if="isLoading"
              class="animate-spin loading-icon"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
              ></path>
            </svg>
            <svg
              v-else
              xmlns="http://www.w3.org/2000/svg"
              class="send-icon"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z"
              />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0;
  overflow: hidden;
}

.header {
  display: flex;
  justify-content: center;
  padding-top: 1rem;
  padding-bottom: 1rem;
  background-color: white;
  border-bottom: 1px solid rgb(226, 232, 240);
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
}

.title {
  font-size: 1.875rem;
  line-height: 2.25rem;
  font-weight: 700;
  color: rgb(59, 130, 246);
  display: flex;
  align-items: center;
}

.icon {
  height: 2rem;
  width: 2rem;
  margin-right: 0.5rem;
}

.welcome-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.model-selector {
  background-color: white;
  border-radius: 0.75rem;
  box-shadow:
    0 4px 6px -1px rgb(0 0 0 / 0.1),
    0 2px 4px -2px rgb(0 0 0 / 0.1);
  padding: 2rem;
  border: 1px solid rgb(226, 232, 240);
  width: 100%;
  max-width: 36rem;
}

.subtitle {
  font-size: 1.25rem;
  line-height: 1.75rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
  color: rgb(30, 41, 59);
}

.model-grid {
  display: grid;
  grid-template-columns: repeat(1, minmax(0, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

@media (min-width: 768px) {
  .model-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

.model-button {
  padding: 1rem;
  border: 1px solid rgb(226, 232, 240);
  border-radius: 0.5rem;
  transition-property: all;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
  display: flex;
  align-items: center;
}

.model-button:hover {
  background-color: rgb(248, 250, 252);
}

.model-button.selected {
  background-color: rgb(239, 246, 255);
  border-color: rgb(59, 130, 246);
  color: rgb(29, 78, 216);
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
}

.check-icon {
  height: 1.25rem;
  width: 1.25rem;
  margin-right: 0.5rem;
  color: rgb(59, 130, 246);
}

.start-button {
  width: 100%;
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
  background-color: rgb(59, 130, 246);
  color: white;
  border-radius: 0.5rem;
  font-weight: 500;
  transition-property: color, background-color, border-color;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

.start-button:hover {
  background-color: rgb(37, 99, 235);
}

.start-button.disabled {
  background-color: rgb(203, 213, 225);
  cursor: not-allowed;
}

.start-button.disabled:hover {
  background-color: rgb(203, 213, 225);
}

.chat-session {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100%;
  overflow: hidden;
}

.model-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 1.5rem;
  padding-right: 1.5rem;
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
  background-color: white;
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  border-bottom: 1px solid rgb(226, 232, 240);
  z-index: 10;
}

.model-info {
  font-size: 0.875rem;
  line-height: 1.25rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  position: relative;
}

.info-icon {
  height: 1.25rem;
  width: 1.25rem;
  margin-right: 0.25rem;
  color: rgb(59, 130, 246);
}

.model-selector-button {
  color: rgb(59, 130, 246);
  margin-left: 0.25rem;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.model-selector-button:hover {
  color: rgb(37, 99, 235);
}

.model-selector-button:focus {
  outline: none;
}

.dropdown-icon {
  height: 1rem;
  width: 1rem;
  transition-property: transform;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

.dropdown-icon.rotate-180 {
  transform: rotate(180deg);
}

.model-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 0.25rem;
  background-color: white;
  border: 1px solid rgb(226, 232, 240);
  border-radius: 0.375rem;
  box-shadow:
    0 10px 15px -3px rgb(0 0 0 / 0.1),
    0 4px 6px -4px rgb(0 0 0 / 0.1);
  z-index: 20;
  width: 16rem;
}

.end-button {
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  font-size: 0.875rem;
  line-height: 1.25rem;
  background-color: rgb(241, 245, 249);
  color: rgb(51, 65, 85);
  border-radius: 0.5rem;
  font-weight: 500;
}

.end-button:hover {
  background-color: rgb(226, 232, 240);
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 1rem;
  padding-bottom: 1rem;
  background-color: rgb(248, 250, 252);
}

@media (min-width: 768px) {
  .messages-container {
    padding-left: 1.5rem;
    padding-right: 1.5rem;
  }
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: rgb(148, 163, 184);
}

.empty-icon {
  width: 4rem;
  height: 4rem;
  margin-bottom: 1rem;
}

.empty-text {
  font-size: 1.125rem;
  line-height: 1.75rem;
}

.messages-list {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  padding-bottom: 1rem;
}

.message {
  position: relative;
}

.message.user {
  display: flex;
  justify-content: flex-end;
}

.message.assistant {
  display: flex;
  justify-content: flex-start;
}

.message-content {
  display: flex;
  max-width: 80%;
  width: 100%;
}

.avatar {
  width: 2.5rem;
  height: 2.5rem;
  border-radius: 9999px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-top: 0.25rem;
}

.user-avatar {
  background-color: rgb(219, 234, 254);
  color: rgb(37, 99, 235);
  order: 9999;
  margin-left: 0.5rem;
}

.assistant-avatar {
  background-color: rgb(241, 245, 249);
  color: rgb(71, 85, 105);
  margin-right: 0.5rem;
}

.message-body {
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
  border-radius: 0.75rem;
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  width: 100%;
  min-width: 300px;
  overflow: visible;
}

.message.user .message-body {
  background-color: rgb(59, 130, 246);
  color: white;
  border-top-right-radius: 0;
}

.message.assistant .message-body {
  background-color: white;
  color: rgb(51, 65, 85);
  border-top-left-radius: 0;
  min-width: 300px;
  border: 1px solid rgb(226, 232, 240);
}

.assistant-content {
  width: 100%;
  min-width: 300px;
  display: block;
  overflow: visible;
}

.message-header {
  display: flex;
  align-items: center;
  margin-bottom: 0.25rem;
  font-size: 0.875rem;
  line-height: 1.25rem;
}

.role-label {
  font-weight: 500;
}

.input-container {
  padding: 1rem;
  background-color: white;
  border-top: 1px solid rgb(226, 232, 240);
}

.input-wrapper {
  position: relative;
  display: flex;
  max-width: 56rem;
  margin-left: auto;
  margin-right: auto;
}

.message-input {
  width: 100%;
  padding: 0.75rem 1rem;
  padding-right: 3rem;
  background-color: rgb(248, 250, 252);
  border: 1px solid rgb(203, 213, 225);
  border-radius: 0.5rem;
  resize: none;
  min-height: 58px;
  max-height: 200px;
  color: rgb(51, 65, 85);
}

.message-input::placeholder {
  color: rgb(148, 163, 184);
}

.message-input:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgb(59, 130, 246);
  border-color: transparent;
}

.loading-content {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  min-height: 36px;
}

.loading-dots {
  display: flex;
  align-items: center;
}

.loading-dots span {
  width: 8px;
  height: 8px;
  margin: 0 3px;
  background-color: #64748b;
  border-radius: 50%;
  display: inline-block;
  animation: bounce 1.4s infinite ease-in-out both;
}

.loading-dots span:nth-child(1) {
  animation-delay: -0.32s;
}

.loading-dots span:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes bounce {
  0%,
  80%,
  100% {
    transform: scale(0);
  }
  40% {
    transform: scale(1);
  }
}

.send-button {
  position: absolute;
  right: 0.75rem;
  bottom: 0.75rem;
  border-radius: 9999px;
  padding: 0.5rem;
  color: rgb(59, 130, 246);
}

.send-button:hover {
  background-color: rgb(239, 246, 255);
}

.send-button:disabled {
  color: rgb(148, 163, 184);
}

.send-button:disabled:hover {
  background-color: transparent;
  cursor: not-allowed;
}

.loading-icon,
.send-icon {
  height: 1.25rem;
  width: 1.25rem;
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.dropdown-check-icon {
  width: 1rem;
  height: 1rem;
  margin-right: 0.5rem;
  color: rgb(37, 99, 235);
}
</style>

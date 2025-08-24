<template>
  <div class="chat-window">
    <div class="chat-header">
      <h2>Chat Client</h2>
      <span class="status" :class="{ connected: isConnected }">
        {{ isConnected ? 'Connected' : 'Disconnected' }}
      </span>
    </div>
    
    <div class="messages-container" ref="messagesContainer">
      <div v-if="messages.length === 0" class="welcome-message">
        Connected to ch.at. Type your message to begin.
      </div>
      <div
        v-for="(message, index) in messages"
        :key="index"
        class="message"
        :class="message.type"
      >
        <div class="message-content">{{ message.content }}</div>
        <div class="message-time">{{ formatTime(message.timestamp) }}</div>
      </div>
      <div v-if="isLoading" class="message bot loading">
        <div class="typing-indicator">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
    </div>
    
    <div class="input-container">
      <input
        v-model="inputMessage"
        @keyup.enter="sendMessage"
        type="text"
        placeholder="Type your message..."
        class="message-input"
        :disabled="isLoading"
      />
      <button
        @click="sendMessage"
        class="send-button"
        :disabled="!inputMessage.trim() || isLoading"
      >
        Send
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted } from 'vue'
import axios from 'axios'

interface Message {
  type: 'user' | 'bot'
  content: string
  timestamp: Date
}

const messages = ref<Message[]>([])
const inputMessage = ref('')
const isLoading = ref(false)
const isConnected = ref(true)
const messagesContainer = ref<HTMLElement>()

const API_URL = 'http://ch.at/'
const TIMEOUT = 30000

const scrollToBottom = async () => {
  await nextTick()
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

const formatTime = (date: Date) => {
  return new Intl.DateTimeFormat('en-US', {
    hour: '2-digit',
    minute: '2-digit'
  }).format(date)
}

const sendMessage = async () => {
  const message = inputMessage.value.trim()
  if (!message || isLoading.value) return
  
  messages.value.push({
    type: 'user',
    content: message,
    timestamp: new Date()
  })
  
  inputMessage.value = ''
  isLoading.value = true
  await scrollToBottom()
  
  try {
    const response = await axios.get(API_URL, {
      params: { q: message },
      timeout: TIMEOUT,
      headers: {
        'Accept': 'application/json, text/plain, */*'
      }
    })
    
    let botResponse = ''
    
    // Handle JSON response
    if (typeof response.data === 'object' && response.data.answer) {
      botResponse = response.data.answer
    } 
    // Handle plain text response
    else if (typeof response.data === 'string') {
      const lines = response.data.split('\n')
      for (const line of lines) {
        if (line.startsWith('A: ')) {
          botResponse = line.substring(3)
          break
        }
      }
      if (!botResponse) {
        botResponse = response.data
      }
    } else {
      botResponse = String(response.data)
    }
    
    messages.value.push({
      type: 'bot',
      content: botResponse,
      timestamp: new Date()
    })
    
  } catch (error) {
    let errorMessage = 'Error: Failed to get response'
    
    if (axios.isAxiosError(error)) {
      if (error.code === 'ECONNABORTED') {
        errorMessage = 'Request timed out'
      } else if (error.response) {
        errorMessage = `Error: Server returned ${error.response.status}`
      } else if (error.request) {
        errorMessage = 'Connection error: Unable to reach server'
      }
    }
    
    messages.value.push({
      type: 'bot',
      content: errorMessage,
      timestamp: new Date()
    })
    
    isConnected.value = false
    setTimeout(() => {
      isConnected.value = true
    }, 3000)
  } finally {
    isLoading.value = false
    await scrollToBottom()
  }
}

onMounted(() => {
  scrollToBottom()
})
</script>

<style scoped>
.chat-window {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 12px;
  margin: 20px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
  overflow: hidden;
}

.chat-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chat-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
}

.status {
  padding: 4px 12px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.2);
  font-size: 0.875rem;
  transition: all 0.3s;
}

.status.connected {
  background: rgba(52, 211, 153, 0.3);
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.welcome-message {
  text-align: center;
  color: #999;
  padding: 40px 20px;
  font-style: italic;
}

.message {
  display: flex;
  flex-direction: column;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message.user {
  align-items: flex-end;
}

.message.bot {
  align-items: flex-start;
}

.message-content {
  max-width: 70%;
  padding: 12px 16px;
  border-radius: 18px;
  word-wrap: break-word;
  line-height: 1.4;
}

.message.user .message-content {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-bottom-right-radius: 4px;
}

.message.bot .message-content {
  background: #f1f3f5;
  color: #333;
  border-bottom-left-radius: 4px;
}

.message-time {
  font-size: 0.75rem;
  color: #999;
  margin-top: 4px;
  padding: 0 4px;
}

.loading {
  padding: 12px 16px;
}

.typing-indicator {
  display: flex;
  gap: 4px;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  background: #999;
  border-radius: 50%;
  animation: typing 1.4s infinite;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing {
  0%, 60%, 100% {
    transform: translateY(0);
    opacity: 0.5;
  }
  30% {
    transform: translateY(-10px);
    opacity: 1;
  }
}

.input-container {
  display: flex;
  padding: 20px;
  gap: 12px;
  background: #fafafa;
  border-top: 1px solid #e0e0e0;
}

.message-input {
  flex: 1;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 24px;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.3s;
}

.message-input:focus {
  border-color: #667eea;
}

.message-input:disabled {
  background: #f5f5f5;
  cursor: not-allowed;
}

.send-button {
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 24px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.send-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.send-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.messages-container::-webkit-scrollbar {
  width: 6px;
}

.messages-container::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.messages-container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 3px;
}

.messages-container::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>
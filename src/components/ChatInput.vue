<template>
  <form class="chat-input" @submit.prevent="submit">
    <input
      v-model="text"
      type="text"
      class="chat-text"
      placeholder="Type a message…"
    />

    <button class="send-btn" type="submit" aria-label="Send" :class="{ typing: isTyping}">
      
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
      >
        <line x1="22" y1="2" x2="11" y2="13"></line>
        <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
      </svg>
    </button>
  </form>
</template>


<script setup>
import { ref, computed, watch } from 'vue'

const emit = defineEmits(['send'])
const text = ref('')

const isTyping = computed(() => text.value.trim().length > 0)

function submit() {
  if (!text.value.trim()) return
  emit('send', text.value)
  text.value = ''
}
</script>


<style scoped>
.chat-input {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: var(--header-bg);
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

/* input field */
.chat-text {
  flex: 1;
  border: none;
  outline: none;
  padding: 10px 14px;
  border-radius: 20px;
  font-size: 0.95rem;
  background: var(--chat-bg);
  color: var(--text-color);
}

/* placeholder */
.chat-text::placeholder {
  opacity: 0.6;
}

/* send button */
.send-btn {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: none;
  background: #25d366; 
  color: #fff;
  font-size: 1rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

/* hover / active */
.send-btn:hover {
  box-shadow: 0 4px 10px rgba(37, 211, 102, 0.4);
  transform: translateY(-1px);
}

.send-btn:active {
  transform: scale(0.95);
}

.send-btn.typing {
  animation: bounce 0.8s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-3px) scale(1.05); }
}

</style>

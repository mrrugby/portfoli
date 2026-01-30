<template>
  <div ref="container" class="chat-messages p-3">
    <div v-for="msg in messages" :key="msg.id" class="message" :class="msg.from">
      <div class="bubble">
        <span class="text" v-html="msg.text"></span>
        <small class="time">
        {{ msg.time }}
        <span v-if="msg.from === 'me'" class="ticks">
        {{ msg.status === 'read' ? '✔✔' : '✔' }}
  </span>
</small>

      
      </div>
      
    </div>
  </div>
</template>

<script setup>
import { watch, ref, nextTick } from 'vue'

const props = defineProps({
  messages: Array
})

const container = ref(null)

watch(() => props.messages.length, async () => {
  await nextTick()
  container.value.scrollTop = container.value.scrollHeight
})
</script>


<style scoped>
.chat-messages {
  flex: 1;
  overflow-y: auto;
  background: var(--chat-bg);
  padding: 12px;
  scrollbar-width: none;
}

/* message row */
.message {
  display: flex;
  margin-bottom: 12px;
  animation: popIn 0.18s ease-out;
}

/* alignment */
.message.me {
  justify-content: flex-end;
}

.message.bot {
  justify-content: flex-start;
}

/* bubble base */
.bubble {
font-family: var(--font-family);
  max-width: 72%;
  padding: 10px 14px;
  border-radius: 18px;
  font-size: 0.95rem;
  line-height: 1.4;
  position: relative;
  word-wrap: break-word;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.12);
  color: var(--text-color);
}

/* sent bubble */
.me .bubble {
  background: var(--sent-bg);
  border-top-right-radius: 6px;
}

/* received (bot) bubble */
.bot .bubble {
  background: var(--received-bg);
  border-top-left-radius: 6px;
}

/* message text */
.text {
  display: block;
}

/* time + ticks wrapper */
.time {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 4px;
  font-size: 0.65rem;
  opacity: 0.6;
  margin-top: 4px;
  white-space: nowrap;
}

/* ticks */
.ticks {
  font-size: 0.65rem;
  color: #53bdeb; /* WhatsApp blue */
}

/* subtle pop animation */
@keyframes popIn {
  from {
    transform: scale(0.96);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

</style>

<template>
  <div class="chat-bg">
    <div ref="container" class="chat-messages">
      <div
        v-for="msg in messages"
        :key="msg.id"
        class="message"
        :class="[msg.from, { typing: msg.typing }]"
      >
        <div class="bubble">
          <div v-if="msg.typing" class="typing-bubble">
            <span></span><span></span><span></span>
          </div>

          <template v-else>
            <span class="text" v-html="msg.text"></span>

            <div v-if="msg.buttons && msg.buttons.length" class="quick-buttons">
              <div v-for="(btn, i) in msg.buttons" :key="i" class="quick-btn-row">
                <button class="quick-btn" @click="$emit('send', btn)">
                  {{ btn }}
                </button>
              </div>
            </div>

            <small class="time">
              {{ msg.time }}
              <span v-if="msg.from === 'me'" class="ticks">
                {{ msg.status === 'read' ? '✔✔' : '✔' }}
              </span>
            </small>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, watch, nextTick } from 'vue'

const props = defineProps({
  messages: Array
})

const container = ref(null)

// scroll to bottom 
watch(
  () => props.messages,
  async () => {
    await nextTick()
    if (container.value) {
      container.value.scrollTop = container.value.scrollHeight
    }
  },
  { deep: true }
)
</script>

<style scoped>
.chat-messages {
  position: relative;
  z-index: 2;
  min-height: 0;
  flex: 1;
  overflow-y: auto;
  overscroll-behavior: contain;
  display: flex;
  flex-direction: column;
  padding: 12px 12px 20px;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;

  background: transparent;

}

.chat-bg{
  position: relative;
  flex: 1;
  min-height: 0;
  overflow: hidden;

  display: flex;
  flex-direction: column;

  background-image: var(--chat-wallpaper);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;

}

.chat-bg::before{
  content: "";
  position: absolute;
  inset: 0;

  background-image: var(--chat-wallpaper);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;

  filter: blur(1px);
  transform: scale(1.06);
  z-index: 0;
}

.chat-bg::after {
  content: "";
  position: absolute;
  inset: 0;
  background: rgba(255, 255, 255, 0.35);
  z-index: 1;
}

/* message row */
.message {
  position: relative;
  z-index: 2;
  display: flex;
  margin-bottom: 6px;
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
  width: fit-content;
  padding: 10px 14px;
  border-radius: 18px;
  font-size: 0.95rem;
  line-height: 1.4;
  position: relative;
  word-wrap: break-word;
  overflow-wrap: break-word;
  color: var(--text-color);
  background: var(--received-bg);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.12);
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
  white-space: pre-wrap;  /* allows wrapping and preserves line breaks */
  word-wrap: break-word;
  overflow-wrap: break-word;
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
  color: #53bdeb;
}

/* subtle pop animation */
@keyframes popIn {
  from { transform: scale(0.96); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

/* typing bubble */
.typing .bubble {
  padding: 10px 14px;
}

.typing-bubble {
  display: inline-flex;
  gap: 6px;
  align-items: center;
}

.typing-bubble span {
  width: 6px;
  height: 6px;
  background: #999;
  border-radius: 50%;
  animation: typingBlink 1.4s infinite both;
}

.typing-bubble span:nth-child(2) { animation-delay: 0.2s; }
.typing-bubble span:nth-child(3) { animation-delay: 0.4s; }

@keyframes typingBlink {
  0% { opacity: 0.2; }
  20% { opacity: 1; }
  100% { opacity: 0.2; }
}

/* QUICK REPLY BUTTONS */
.quick-buttons {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.quick-btn {
  width: 100%;
  background: #e9edef;
  border: none;
  padding: 8px 12px;
  border-radius: 16px;
  cursor: pointer;
  font-size: 13px;
  text-align: left;
  transition: background 0.15s ease;
}

.quick-btn:hover { background: #d8dbdd; }
</style>

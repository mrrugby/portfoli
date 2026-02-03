<template>
  <div class="chat-header d-flex align-items-center px-3 shadow-sm">
    <img src="../assets/profile-pic.webp" class="avatar" alt="profile-pic" @click="showAvatarModal = true">

    <div class="user-info flex-grow-1 ms-2">
      <div class="username" @click="showDetailsModal = true">Shaka Senaji</div>
      <div class="status">
        <span v-if="status === 'typing..'">
          typing<span class="dots"><span>.</span><span>.</span><span>.</span></span>
        </span>
        <span v-else>
          {{ getLastSeen() }}
        </span>
      </div>
    </div>

    <button class="btn btn-theme-toggle" @click="toggleTheme" title="Toggle Theme">
      🌗
    </button>

    <div v-if="showAvatarModal" class="modal-overlay" @click.self="showAvatarModal = false">
      <img src="../assets/profile-pic.webp" class="modal-avatar" />
    </div>

    <div v-if="showDetailsModal" class="modal-overlay" @click.self="showDetailsModal = false">
      <div class="modal-content">
        <p>The site is still being built, Make sure to check in to see how features are updated.! </p>
      </div>
    </div>
  </div>
</template>



<style scoped>
.chat-header {
  height: 64px;
  background: var(--header-bg);
  color: var(--text-color);
  display: flex;
  align-items: center;
  padding: 0 1rem;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  flex-shrink: 0;
}

.avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  cursor: pointer;
  object-fit: cover;
}

.user-info {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.username {
  font-weight: 600;
  font-size: 1.2rem;
  color: var(--text-color);
}

.status {
  font-size: 0.69rem;
  color: var(--text-muted);
  display: flex;
  align-items: center;
  margin-top: 2px;
}

.dots span {
  animation: blink 1.4s infinite both;
  font-weight: bold;
}

.dots span:nth-child(2) {
  animation-delay: 0.2s;
}
.dots span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes blink {
  0%, 100% { opacity: 0; }
  50% { opacity: 1; }
}

.btn-theme-toggle {
  background: var(--btn-bg);
  border: none;
  padding: 6px 10px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.1rem;
  transition: background 0.3s, transform 0.2s;
}

.btn-theme-toggle:hover {
  background: var(--btn-hover);
  transform: scale(1.1);
}

/* Modal overlay */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0,0,0,0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  animation: fadeIn 0.2s ease;
}

/* Avatar modal */
.modal-avatar {
  width: 200px;
  height: 200px;
  border-radius: 30%;
  object-fit: cover;
  animation: popIn 0.3s ease;
  cursor: pointer;
}

/* Details modal */
.modal-content {
  background: var(--chat-bg);
  color: var(--text-color);
  padding: 20px 25px;
  border-radius: 12px;
  max-width: 350px;
  text-align: center;
  box-shadow: 0 5px 15px rgba(0,0,0,0.3);
  animation: popIn 0.3s ease;
}

.modal-content a {
  color: var(--link-color);
  text-decoration: underline;
}

/* Animations */
@keyframes popIn {
  0% { transform: scale(0.5); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}


</style>

<script setup>
import { ref } from 'vue'

const status = ref('Online 🟢')
let lastSeenAt = null

const emit = defineEmits(['toggle-theme'])

function toggleTheme() {
  emit('toggle-theme')
}

// Avatar / Details modal
const showAvatarModal = ref(false)
const showDetailsModal = ref(false)

// Update status or record last seen
function setStatus(value) {
  if (value === 'typing') {
    status.value = 'typing'
  } else {
    status.value = value
    lastSeenAt = new Date()
  }
}

// Compute last seen text
function getLastSeen() {
  if (!lastSeenAt) return 'last seen just now'

  const diff = Math.floor((Date.now() - lastSeenAt.getTime()) / 60000)
  if (diff < 1) return 'last seen just now'
  if (diff === 1) return 'last seen 1 minute ago'
  return `last seen ${diff} minutes ago`
}

defineExpose({
  setStatus,
  getLastSeen
})
</script>




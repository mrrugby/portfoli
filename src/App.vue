<template>
  <div class="app chat-contaniner">
    <div class="chat-shell shadow-lg">
      <ChatHeader ref="headerRef" @toggle-theme="toggleTheme" />
      <ChatMessages :messages="messages" @send="sendMessage":inputHeight="chatInput?.$el?.offsetHeight"/>
      <ChatInput ref="chatInput" @send="sendMessage"/>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import ChatHeader from './components/ChatHeader.vue'
import ChatMessages from './components/ChatMessages.vue'
import ChatInput from './components/ChatInput.vue'
import sentSound from './assets/sentmessage.mp3'

// -------------------- HELPERS FIRST (avoid hoist bugs) --------------------
function getTime() {
  const d = new Date()
  return `${d.getHours()}:${String(d.getMinutes()).padStart(2, '0')}`
}

// -------------------- STATE --------------------
const savedMessages = localStorage.getItem('chat_messages')
const savedTheme = localStorage.getItem('chat_theme')

const theme = ref(savedTheme || 'light')
document.documentElement.setAttribute('data-theme', theme.value)

const messages = ref(
  savedMessages ? JSON.parse(savedMessages) : [{
    id: Date.now(),
    text: `👋 Hello! Welcome to my portfolio.<br><br>
           I’m <span class="bold">Shaka Senaji</span>, an ICT Officer & Systems Support Specialist.<br>
           Type <span class="bold">'help'</span> to explore my experience.`,
    from: 'bot',
    time: getTime()
  }]
)

watch(messages, val => {
  const clean = val.filter(m => m.id !== 'typing')
  localStorage.setItem('chat_messages', JSON.stringify(clean))
}, { deep: true })

// -------------------- AUDIO --------------------
const audio = new Audio(sentSound)
function playSound() {
  audio.currentTime = 0
  audio.play()
}

// -------------------- HEADER --------------------
const headerRef = ref(null)
let headerTimeout = null
function setHeader(text) {
  headerRef.value?.setStatus(text)
  clearTimeout(headerTimeout)
  headerTimeout = setTimeout(() => {
    const lastSeen = headerRef.value?.getLastSeenText()
    if (lastSeen) headerRef.value?.setStatus(lastSeen)
  }, 4000)
}

// -------------------- MESSAGE HELPERS --------------------
function pushBot(text, buttons = []) {
  messages.value.push({
    id: Date.now(),
    text,
    from: 'bot',
    time: getTime(),
    buttons
  })
  playSound()
}

function pushUser(text) {
  messages.value.push({
    id: Date.now(),
    text,
    from: 'me',
    time: getTime(),
    status: 'sent'
  })
  playSound()
}

function addTyping() {
  messages.value.push({ id: 'typing', from: 'bot', typing: true })
}

function removeTyping() {
  messages.value = messages.value.filter(m => m.id !== 'typing')
}

// -------------------- FLOW MANAGER --------------------
const activeFlow = ref(null)

function startFlow(type) {
  activeFlow.value = { type, step: 'name', data: {} }
}

function stopFlow() {
  activeFlow.value = null
}

// -------------------- INTENT DETECTION --------------------
function detectIntent(input) {
  const msg = input.toLowerCase()

  if (msg.includes('website') || msg.includes('portfolio') || msg.includes('site'))
    return 'website'

  if (msg.includes('hire'))
    return 'hire'

  return null
}

// -------------------- FLOWS --------------------
function handleFlow(input) {
  const flow = activeFlow.value
  if (!flow) return false

  const step = flow.step
  const data = flow.data
  const msg = input.toLowerCase()

  // Allow escape commands inside flows
  if (['help','clear','cancel'].includes(msg)) {
    stopFlow()
    commands[msg]()
    return true
  }

  // ---------------- HIRE FLOW ----------------
  if (flow.type === 'hire') {
    if (step === 'name') {
      data.name = input
      flow.step = 'contact'
      pushBot(`✍️ Thanks ${input}! Please enter your email or phone.`)
      return true
    }

    if (step === 'contact') {
      data.contact = input
      flow.step = 'confirm'
      pushBot(`✅ Confirm:<br>
        <strong>Name:</strong> ${data.name}<br>
        <strong>Contact:</strong> ${data.contact}<br>
        Type <strong>yes</strong> to confirm.`)
      return true
    }

    if (step === 'confirm') {
      stopFlow()
      if (msg === 'yes')
        pushBot(`🎉 Thank you! I’ll get in touch shortly.`)
      else
        pushBot(`❌ Cancelled.`)
      return true
    }
  }

  // ---------------- WEBSITE FLOW ----------------
  if (flow.type === 'website') {
    if (step === 'name') {
      data.name = input
      flow.step = 'type'
      pushBot(`🌐 Hi ${input}! What type of website?<br>Portfolio • Business • E-commerce • Other`)
      return true
    }

    if (step === 'type') {
      data.type = input
      flow.step = 'budget'
      pushBot(`💰 What’s your budget range?`)
      return true
    }

    if (step === 'budget') {
      data.budget = input
      flow.step = 'contact'
      pushBot(`📬 How can I contact you?`)
      return true
    }

    if (step === 'contact') {
      data.contact = input
      flow.step = 'confirm'
      pushBot(`✅ Confirm:<br>
        <strong>Name:</strong> ${data.name}<br>
        <strong>Website:</strong> ${data.type}<br>
        <strong>Budget:</strong> ${data.budget}<br>
        <strong>Contact:</strong> ${data.contact}<br>
        Type <strong>yes</strong> to confirm.`)
      return true
    }

    if (step === 'confirm') {
      stopFlow()
      if (msg === 'yes')
        pushBot(`🚀 Great! I’ll reach out soon.`)
      else
        pushBot(`❌ Cancelled.`)
      return true
    }
  }

  return true
}


// -------------------- COMMAND MAP --------------------
const commands = {
  help: () => pushBot(
  `🧭 <span class="bold">What would you like to explore?</span>`,
  [
    'Hire Me',
    'Website Build',
    'My Experience',
    'My Skills',
    'Education',
    'Contact',
    'Clear Chat'
  ]
),



  clear: () => {
  messages.value = []
  pushBot(
    `👋 Hi there! I'm <span class="bold">Shaka Senaji</span>.<br><br>
     I build systems, solve ICT problems, and design practical web solutions.<br>
     What would you like?`,
    [
      'Hire Me',
      'Website Build',
      'My Experience',
      'My Skills',
      'Contact'
    ]
  )
},


  experience: () => pushBot(`💼 <span class="bold">Experience</span><br>
    EcoSafi Ltd – IT Assistant (2025-Present)<br>
    Nairobi Municipal Council – ICT Intern<br>
    🔗 <a class="alink" href="https://github.com/mrrugby/Council-Care" target="_blank">Project</a>`),

  skills: () => pushBot(`🛠️ Windows, Linux, Networking, Django, Git, MySQL`),

  education: () => pushBot(`🎓 BSc Computer Science – SPU<br>Certifications: Moringa, eMobilis`),

  contact: () => pushBot(`
    📧 snjishaka@gmail.com<br>
    📞 +254 704 210 555<br>
    🌐 <a class="alink" href="https://shakasenaji.vercel.app" target="_blank">Portfolio</a>
  `),

  cancel: () => pushBot('❌ Process cancelled.')
}

// -------------------- MAIN RESPONDER --------------------
function sendMessage(text) {
  pushUser(text)
  setHeader('typing... 🤖')
  addTyping()

  setTimeout(() => respond(text), 600)
}

function respond(input) {
  removeTyping()

   // mark last user message as read
  const lastUserMsg = [...messages.value].reverse().find(m => m.from === 'me')
  if (lastUserMsg) lastUserMsg.status = 'read'


  const key = input.toLowerCase().trim()



  // Handle active flow
  if (handleFlow(input)) return setHeader('Online 🟢')

  // Detect intent to start flows
  const intent = detectIntent(key)
  if (intent) {
    startFlow(intent)
    return pushBot(`👋 Let’s begin. What’s your full name?`)
  }

  // commands
  const map = {
  'hire me': () => {
    startFlow('hire')
    pushBot(`👋 Great! What’s your full name?`)
  },

  'build me a website': () => {
    startFlow('website')
    pushBot(`🌐 Nice! What’s your full name?`)
  },

  'my experience': commands.experience,
  'my skills': commands.skills,
  'education': commands.education,
  'contact': commands.contact,
  'clear chat': commands.clear
}

if (map[key]) return map[key]()

if (commands[key]) {
  commands[key]()
  return setHeader('Online 🟢')
}

// -------------------- DEFAULT FALLBACK --------------------
pushBot(
  `🤔 I didn’t quite understand that.<br><br>
   Type <span class="bold">'help'</span> to see what I can do.`,
  [
    'Hire Me',
    'Help'
  ]
)

return setHeader('Online 🟢')

  
}

// -------------------- THEME --------------------
function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
  document.documentElement.setAttribute('data-theme', theme.value)
  localStorage.setItem('chat_theme', theme.value)
}
</script>


<style>
body, input, textarea, button, * { font-family: var(--font-family); }

.app{
 min-height:100vh;
 background: var(--app-bg);
 display: flex;
 justify-content: center;
 align-items: center;
}

.chat-shell {
  width: 100%;
  max-width: 1500px;
  height: 100vh;
  display: flex;
  flex-direction: column;
}

@media (min-width: 992px) {
  .chat-shell {
    height: 90vh;
    border-radius: 12px;
    overflow: hidden;
  }
}

.bold{ 
  font-weight: 600; 
  
  }

</style>

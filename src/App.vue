<template>
  <div
  class="app chat-container"
  :style="{ height: appHeight + 'px' }"
>
    <div class="chat-shell shadow-lg">
      <ChatHeader ref="headerRef" @toggle-theme="toggleTheme" />
      <ChatMessages :messages="messages" @send="sendMessage":inputHeight="chatInput?.$el?.offsetHeight"/>
      <ChatInput ref="chatInput" @send="sendMessage"/>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onBeforeUnmount, onMounted } from 'vue'
import ChatHeader from './components/ChatHeader.vue'
import ChatMessages from './components/ChatMessages.vue'
import ChatInput from './components/ChatInput.vue'
import sentSound from './assets/sentmessage.mp3'

const appHeight = ref(window.innerHeight)

function updateHeight() {
  appHeight.value = window.visualViewport
    ? window.visualViewport.height
    : window.innerHeight
}

onMounted(() => {
  updateHeight()
  window.visualViewport?.addEventListener('resize', updateHeight)
  window.addEventListener('resize', updateHeight)
})

onBeforeUnmount(() => {
  window.visualViewport?.removeEventListener('resize', updateHeight)
  window.removeEventListener('resize', updateHeight)
})



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
    text: `👋 Hello! Welcome to my portfolio.
I’m <span class="bold">Shaka Senaji</span>, an ICT Officer & Systems Support Specialist.
Type <span class="bold">'help'</span> to explore....`,
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
  const msg = input.trim()

  
  if (['help','clear','cancel'].includes(msg.toLowerCase())) {
    stopFlow()
    commands[msg.toLowerCase()]()
    return true
  }

  // ---------------- HIRE FLOW ----------------
  if (flow.type === 'hire') {

    // Step 1: Name
    if (step === 'name') {
      if (!/^[a-zA-Z ]{2,}$/.test(msg)) {
        pushBot('❌ Please enter a valid name (letters only, min 2 characters).')
        return true
      }
      data.name = msg
      flow.step = 'contact'
      pushBot(`✍️ Thanks ${data.name}! Please enter your email or phone.`)
      return true
    }

    // Step 2: Contact
    if (step === 'contact') {
      const isEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(msg)
      const isPhone = /^[0-9+]{6,15}$/.test(msg)
      if (!isEmail && !isPhone) {
        pushBot('❌ Please enter a valid email or phone number.')
        return true
      }
      data.contact = msg
      flow.step = 'confirm'
      pushBot(`✅ Confirm:<br>
        <strong>Name:</strong> ${data.name}
        <strong>Contact:</strong> ${data.contact}<br>
        Type <strong>yes</strong> to confirm.`)
      return true
    }

    // Step 3: Confirm
    if (step === 'confirm') {
      stopFlow()
      if (msg.toLowerCase() === 'yes') {
        pushBot(`🚀 Great! I’ll reach out soon.`)
      } else {
        pushBot(`❌ Cancelled.`)
      }
      return true
    }
  }

  // ---------------- WEBSITE FLOW ----------------
  if (flow.type === 'website') {

    // Step 1: Name
    if (step === 'name') {
      if (!/^[a-zA-Z ]{2,}$/.test(msg)) {
        pushBot('❌ Please enter a valid name (letters only).')
        return true
      }
      data.name = msg
      flow.step = 'type'
      pushBot(`🌐 Hi ${data.name}! What type of website?<br>Portfolio • Business • E-commerce • Other`)
      return true
    }

    // Step 2: Type
    if (step === 'type') {
      const val = msg.toLowerCase()
      const allowed = ['portfolio', 'business', 'e-commerce', 'other']
      if (!allowed.includes(val)) {
        pushBot('❌ Please choose one: Portfolio, Business, E-commerce, Other.')
        return true
      }
      data.type = val
      flow.step = 'budget'
      pushBot(`💰 What’s your budget range(Ksh)? (e.g., 500 or 500-1000)`)
      return true
    }

    // Step 3: Budget
    if (step === 'budget') {
      if (!/^\d+(-\d+)?$/.test(msg)) {
        pushBot('❌ Please enter a valid budget (number or range, e.g., 500 or 500-1000).')
        return true
      }
      data.budget = msg
      flow.step = 'contact'
      pushBot(`📬 How can I contact you? (Email or phone)`)
      return true
    }

    // Step 4: Contact
    if (step === 'contact') {
      const isEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(msg)
      const isPhone = /^[0-9+]{6,15}$/.test(msg)
      if (!isEmail && !isPhone) {
        pushBot('❌ Please enter a valid email or phone number.')
        return true
      }
      data.contact = msg
      flow.step = 'confirm'
      pushBot(`✅ Confirm:<br>
<strong>Name:</strong> ${data.name}
<strong>Website:</strong> ${data.type}
<strong>Budget:</strong> ${data.budget}
<strong>Contact:</strong> ${data.contact}<br>
Type <strong>yes</strong> to confirm.`)
      return true
    }

    // Step 5: Confirm
    if (step === 'confirm') {
      stopFlow()
      if (msg.toLowerCase() === 'yes') {
        pushBot(`🚀 Great! I’ll reach out soon.`)
      } else {
        pushBot(`❌ Cancelled.`)
      }
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
    'About Me',
    'Website Build',
    'My Experience',
    'Skills',
    'Education',
    'Hire Me',
    'My Contacts',
    'Clear Chat'
  ]
),



  clear: () => {
  messages.value = []
  pushBot(
    `👋 Nice to see you again! What would you like?`,
    [
      'Hire Me',
      'Help',
      'My Contacts'
    ]
  )
},


  'my experience': () => pushBot(
`💼 <span class="bold">Professional Experience</span><br>

<span class="bold">EcoSafi Ltd</span> – IT Assistant (2025–Present)<br>
• Tier 1 & 2 support for business-critical systems<br>
• User access control, logs & data protection<br>
• System monitoring, backups & uptime (99.9%)<br>
• Endpoint patching & preventive maintenance<br>
• M-PESA & internal platform support<br><br>

<span class="bold">Nairobi Municipal Council</span> – ICT Intern<br>
• Built a full ICT ticketing system (Django)<br>
• Resolved 100+ ICT issues monthly<br>
• Supported 500+ daily users<br>
• Network & backup operations<br>
🔗 <a class="alink" href="https://github.com/mrrugby/Council-Care" target="_blank">View Project</a><br><br>

<span class="bold">Earlier Roles</span><br>
• ICT Technician – Noip Agency<br>
• ICT Technician – Dove Adventures`
),

  skills: () => pushBot(
`🛠️ <span class="bold">Technical Skills</span><br>

<span class="bold">Systems & Support</span><br>
• Windows & Linux (Ubuntu/Debian)<br>
• Tier 1 & Tier 2 User Support<br>
• Patch management & endpoint security<br>
• Backup & disaster recovery<br><br>

<span class="bold">Networking</span><br>
• LAN/WAN & Wi-Fi setup<br>
• VPNs, routers, switches & firewalls<br>
• Network diagnostics & troubleshooting<br>

<span class="bold">Databases & Systems</span><br>
• MySQL & MongoDB<br>
• ERP-style system support<br>
• Data integrity & reporting<br><br>

<span class="bold">Development & Tools</span><br>
• Python (Django), HTML, CSS<br>
• Git & GitHub<br>
• VS Code, CLI, cPanel`
),

  education: () => pushBot(
`🎓 <span class="bold">Education & Certifications</span><br>

• BSc Computer Science – Saint Paul’s University (2018–2024)<br>
• Web Development – Moringa School<br>
• Python & Django – eMobilis<br><br>

<span class="bold">Focus:</span><br>
Systems support, networking, databases & web platforms, django, Python`
),

'about me': () => pushBot(
`👋 <span class="bold">About Me</span><br>

I'm <span class="bold">Shaka Senaji</span>, an IT Analyst & Systems Support professional with 4+ years of experience keeping systems reliable, secure, and running smoothly.<br>

I specialize in:<br>
• ICT support & infrastructure reliability<br>
• Network & system troubleshooting<br>
• User support & access control<br>
• Building practical web systems that solve real problems`
),



  'my contacts': () => pushBot(
`📬 <span class="bold">Get in Touch</span><br>
📧 Email: snjishaka@gmail.com<br>
📞 Phone: +254 704 210 555<br>
🌍 Portfolio: <a class="alink" href="https://shakasenaji.vercel.app" target="_blank">shakasenaji.vercel.app</a><br>
💻 GitHub: <a class="alink" href="https://github.com/mrrugby" target="_blank">mrrugby</a><br>
🔗 LinkedIn: <a class="alink" href="https://linkedin.com/in/snji-shaka" target="_blank">snji-shaka</a>`
),


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
   Type <span class="bold">'help'</span> to get started.`,
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

html, body {
  height: 100%;
  margin: 0;
  overflow: hidden;
}
.app{
 height:100dvh;
 background: var(--app-bg);
 display: flex;
 align-items: stretch;
 width: 100%;
}

.chat-shell {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100%;
  max-width: 1500px;
  margin: 0 auto;
  overflow: hidden;

  
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
}

@media (min-width: 992px) {
  .chat-shell {
    height: 100%;
    border-radius: 12px;
    overflow: hidden;
  }
}

.bold{ 
  font-weight: 600; 
  
  }

</style>

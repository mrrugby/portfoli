<template>
  <div class="app">
    <div class="chat-shell shadow-lg">
      <ChatHeader ref="headerRef" @toggle-theme="toggleTheme" />
      <ChatMessages :messages="messages" />
      <ChatInput @send="sendMessage"/>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import ChatHeader from './components/ChatHeader.vue'
import ChatMessages from './components/ChatMessages.vue'
import ChatInput from './components/ChatInput.vue'


const theme = ref('light')
const headerRef = ref(null)
const audio = new Audio('../src/assets/sentmessage.mp3')
document.documentElement.setAttribute('data-theme', theme.value)
let hireFlow = null;

function handleHireFlow(input) {
  switch (hireFlow.step) {
    case 'name':
      hireFlow.name = input
      messages.value.push({
        id: Date.now(),
        text: `Thanks ${input}! Please enter your email or phone number.`,
        from: 'bot',
        time: getTime(),
        status: 'sent'
      })
      hireFlow.step = 'contact'
      break

    case 'contact':
      hireFlow.contact = input
      messages.value.push({
        id: Date.now(),
        text: `Great! Please confirm your details:<br><br>
        <strong>Name:</strong> ${hireFlow.name}<br>
        <strong>Contact:</strong> ${hireFlow.contact}<br><br>
        Type 'yes' to confirm or 'no' to cancel.`,
        from: 'bot',
        time: getTime(),
        status: 'sent'
        })
      hireFlow.step = 'confirm'
      break

    case 'confirm':
      if (input.toLowerCase() === 'yes') {
        messages.value.push({
          id: Date.now(),
          text: `✅ Thank you! We will get in contact with you shortly.`,
          from: 'bot',
          time: getTime(),
          status: 'sent'
        })
      } else {
        messages.value.push({
          id: Date.now(),
          text: `❌ Hire request canceled. Type 'hire me' to start again.`,
          from: 'bot',
          time: getTime(),
          status: 'sent'
        })
      }
      hireFlow = null
      break
  }

  playSound()
}

function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
  document.documentElement.setAttribute('data-theme', theme.value)
}

const messages = ref([
  {
    id: 1,
    text: `👋 Hello! Welcome to my portfolio.<br><br>
    I’m <span class="bold">Shaka Senaji</span>, an ICT Officer & Systems Support Specialist.<br>
    Type <span class="bold">'help'</span> to explore my experience, skills, and work.`,
    from: "bot",
    time: getTime()
  }
])

function getTime(){
  const d = new Date()
  return `${d.getHours()}:${String(d.getMinutes()).padStart(2, '0')}`
}

function sendMessage(text){
  messages.value.push({
    id: Date.now(),
    text,
    from: "me",
    time: getTime()
  });
  playSound();

  headerRef.value?.setStatus('typing... 🤖');
  setTimeout(() => {
    respond(text)
  }, 400)
}

async function respond(input) {
  const key = input.toLowerCase().trim();
  

  if (hireFlow){
    handleHireFlow(input)
    return
  }

  const lastMsg = messages.value[messages.value.length - 1];
  if (lastMsg?.from === 'me') {
    lastMsg.status = 'read';
  }


  let reply = '';

  switch (key) {

    case 'hire me':
      hireFlow = { step: 'name' }
      reply = `👋 Great! Let's get started. What's your full name?`
      break

    case 'cv':
      case'resume':
      reply = `
      📄 <span class="bold">Download My CV</span><br>
      <a class="alink" href="/Shaka_Senaji_CV.pdf" download>
      👉 Click here to download
      </a>
      `;
      break;

    

    case 'help':
      reply = `

      🧭 <small><span class="bold">Available commands</span><br><br></small>
      📄 <span class="bold">'hire me'</span> – Hire form<br>
      💼 <span class="bold">'experience'</span> – Professional experience<br>
      🧠 <span class="bold">'skills'</span> – Technical skills<br>
      🖧 <span class="bold">'competencies'</span> – Core competencies<br>
      🎓 <span class="bold">'education'</span> – Education & certifications<br>
      🌐 <span class="bold">'contact'</span> – Contact & links<br>
      📄 <span class="bold">'cv'</span> – Download my CV<br>
      🧹 <span class="bold">'clear'</span> – Clear conversation
      `;
      break;

    case 'summary':
      reply = `
      🧑‍💼 <span class="bold">Professional Summary</span><br>
      Competent ICT Officer with <span class="bold">4+ years</span> of hands-on experience in
      ICT support, system administration, and network troubleshooting.<br><br>
      Experienced in ERP-style systems, database reliability, endpoint management,
      backups & disaster recovery, and organization-wide ICT support.<br><br>
      Strong focus on system uptime, data protection, documentation, and compliance.
      `;
      break;

    case 'competencies':
      reply = `
      🧠 <span class="bold">Core Competencies</span><br><br>
      🔧 ICT Infrastructure & Technical Support<br>
      • Level 1 & Level 2 user support<br>
      • Hardware & software troubleshooting<br><br>

      🖥️ System & Server Administration<br>
      • User access & permissions<br>
      • Backups, monitoring & recovery<br>
      • Windows & Linux environments<br><br>

      🌐 Networking & Security<br>
      • LAN/WAN, VPNs, firewalls<br>
      • Routers, switches & access points
      `;
      break;

    case 'experience':
      reply = `
      💼 <span class="bold">Professional Experience</span><br><br>

      <span class="bold">EcoSafi Ltd – Nairobi</span><br>
      IT Assistant | June 2025 – Present<br>
      • Level 1 & 2 support for business-critical systems<br>
      • User access control, logs & data security<br>
      • System monitoring, backups & uptime<br>
      • Endpoint support & patching<br><br>

      <span class="bold">Nairobi Municipal Council</span><br>
      ICT Technician Intern | Jan – Mar 2024<br>
      • Built a digital ICT ticketing system<br>
      • Resolved 100+ issues monthly<br>
      • Supported 500+ daily users<br>
      🔗 <a class="alink" target="_blank" href="https://github.com/mrrugby/Council-Care">Project Link</a>
      `;
      break;

    case 'skills':
      reply = `
      🛠️ <span class="bold">Technical Skills</span><br><br>
      💻 Operating Systems: Windows, Linux (Ubuntu/Debian)<br>
      🌐 Networking: LAN/Wi-Fi, VPNs, routers & diagnostics<br>
      🗄️ Databases: MySQL, MongoDB, backups & integrity<br>
      🔐 Systems Control: Access management, monitoring, DR<br>
      🧑‍💻 Programming: Python, JavaScript (basic)<br>
      🧰 Tools: Git, GitHub, CLI, VS Code, cPanel
      `;
      break;

    case 'education':
      reply = `
      🎓 <span class="bold">Education & Certifications</span><br><br>
      BSc. Computer Science<br>
      Saint Paul’s University, Limuru (2018 – 2024)<br><br>

      📜 Certifications<br>
      • Web Development Fundamentals – Moringa School (2024)<br>
      • Python Web Development (Django & M-PESA) – eMobilis (2024)
      `;
      break;

    case 'contact':
      reply = `
      📬 <span class="bold">Contact Information</span><br><br>
      📧 Email: <span class="bold">snjishaka@gmail.com</span><br>
      📞 Phone: <span class="bold">+254 704 210 555</span><br><br>

      🌐 Portfolio:<br>
      <a class="alink" target="_blank" href="https://shakasenaji.vercel.app">
        shakasenaji.vercel.app
      </a><br><br>

      🧑‍💻 GitHub:<br>
      <a class="alink" target="_blank" href="https://github.com/mrrugby">
        github.com/mrrugby
      </a><br><br>

      🔗 LinkedIn:<br>
      <a class="alink" target="_blank" href="https://linkedin.com/in/snji-shaka">
        linkedin.com/in/snji-shaka
      </a>
      `;
      break;

    case 'clear':
      messages.value = [];
      return respond('intro');

    case 'intro':
      reply = `
      👋 Hi again! I’m <span class="bold">Shaka Senaji</span>.<br><br>
      An ICT Officer specializing in systems support, networking,
      and infrastructure reliability.<br><br>
      Type <span class="bold">'help'</span> to continue.
      `;
      break;

    default:
      reply = `🤔 I didn’t quite get that.<br>Type <span class="bold">'help'</span> to see available commands.`;
  }

  messages.value.push({
    id: Date.now(),
    text: reply,
    from: 'bot',
    time: getTime(),
    status: 'sent'
  });
  playSound();

  headerRef.value?.setStatus('Online 🟢');

  setTimeout(() => {
  const lastSeenText = headerRef.value?.getLastSeenText()
  if (lastSeenText) {
    headerRef.value?.setStatus(lastSeenText)
  }
}, 4000);

  
}

function getReply(input) {
  const key = input.toLowerCase().trim()

  const responses = {
    help: "Try: skills, projects, contact, about",
    skills: "Python, Django, Vue, Linux, DevOps",
    projects: "Check my GitHub 👉 https://github.com/yourname",
    contact: "You can reach me via email or WhatsApp",
    about: "This is a WhatsApp-style portfolio built with Vue"
  }

  return responses[key] || "🤔 I didn’t get that. Type 'help'."
}

function playSound(){
  audio.currentTime = 0;
  audio.play();
}


</script>

<style>

body, input, textarea, button, * {
  font-family: var(--font-family);
}

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


</style>
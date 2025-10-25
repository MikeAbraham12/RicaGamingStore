here<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Rica Assistance</title>
<style>
:root{
  --theme:#ED9DB8;
  --bg:#ECE5DD;
  --bot:#E9D9B8;
  --user:#DCF8C6;
  --accent:#25D366;
}
*{box-sizing:border-box}
body{
  margin:0;font-family:"Poppins",system-ui,Segoe UI,Roboto,Arial;
  background:var(--bg);height:100vh;display:flex;align-items:flex-end;justify-content:center;padding:18px;
}

/* widget */
.chat-widget{
  width:380px;max-width:calc(100% - 36px);height:78vh;
  background:#fff;border-radius:12px;box-shadow:0 8px 30px rgba(0,0,0,0.12);
  display:flex;flex-direction:column;overflow:hidden;
}

/* header */
.chat-header{
  background:var(--theme);color:#fff;padding:12px 14px;display:flex;align-items:center;gap:12px;
}
.avatar{width:44px;height:44px;border-radius:50%;border:2px solid #fff;object-fit:cover;cursor:pointer}
.header-text{display:flex;flex-direction:column}
.title{font-weight:700;font-size:17px}
.status{font-size:12px;opacity:.95}

/* body */
.chat-body{flex:1;padding:14px;overflow-y:auto;background:var(--bg);display:flex;flex-direction:column;gap:10px;scroll-behavior:smooth}
.message{max-width:78%;padding:10px 12px;border-radius:12px;font-size:14px;line-height:1.35;word-break:break-word}
.bot{align-self:flex-start;background:var(--bot);border-bottom-left-radius:4px}
.user{align-self:flex-end;background:var(--user);border-bottom-right-radius:4px}
.meta{display:block;font-size:10px;color:#555;margin-top:6px;text-align:right}
.check{color:#34B7F1;margin-left:6px}

/* typing indicator */
.typing{font-style:italic;color:#666;padding:6px 8px;border-radius:8px;opacity:.95}

/* input */
.input-area{display:flex;gap:8px;padding:12px;border-top:1px solid #eee;background:#fff}
.input-area input{flex:1;padding:12px 14px;border-radius:22px;border:1px solid #ddd;outline:none}
.input-area button{background:var(--accent);color:#fff;border:none;padding:10px 14px;border-radius:20px;cursor:pointer}
.input-area button:active{transform:translateY(1px)}

/* restart */
.restart{margin:10px auto 16px auto;background:var(--accent);border:none;color:#fff;padding:8px 12px;border-radius:8px;cursor:pointer;display:none}

/* small links inside messages */
.msg-link{color:#0b66a0;text-decoration:underline}

/* responsive */
@media (max-width:420px){
  .chat-widget{width:100%;height:92vh;border-radius:8px}
}
</style>
</head>
<body>

<div class="chat-widget" role="application" aria-label="Rica Assistance chat widget">
  <div class="chat-header">
    <!-- initial avatar set to your provided link; user can still click to change -->
    <img id="avatar" class="avatar" src="https://ibb.co.com/KxhZ6Wmf" alt="Rica">
    <div class="header-text">
      <div class="title">Rica Assistance</div>
      <div class="status">online</div>
    </div>
    <input id="profileInput" type="file" accept="image/*" style="display:none" />
  </div>

  <div id="chatBody" class="chat-body" aria-live="polite"></div>

  <div class="input-area">
    <input id="userInput" type="text" placeholder="Tulis pesan di sini..." autocomplete="off" />
    <button id="sendBtn" aria-label="Kirim">Kirim</button>
  </div>

  <button id="restartBtn" class="restart" aria-hidden="true">🔁 Mulai Lagi</button>
</div>

<script>
/*
  FINAL NOTES:
  - This file uses direct frontend fetch to Google Generative Language API if FRONTEND_API_KEY is set.
  - SECURITY: Placing API key in FRONTEND_API_KEY is publicly visible. For production use a backend proxy and set BACKEND_URL.
  - To use direct fetch: set FRONTEND_API_KEY below to your key.
  - To use a safer approach: set BACKEND_URL to your proxy (e.g. https://rica-proxy.example.com) that accepts { message } and returns { reply }.
*/

const BACKEND_URL = "https://your-proxy-server.com/api/ai "; // e.g. "https://rica-proxy.example.com" (recommended to keep API key secure)
const Key: API_KEY
Value: AIzaSyD_8Yt6GM9OuhvYe9Vg9DGQtdyKLpsytnY = ""; // <-- OPTIONAL (not secure). If you put your API key here, the browser will call Google directly.

const GOOGLE_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent";

const chatBody = document.getElementById("chatBody");
const userInput = document.getElementById("userInput");
const sendBtn = document.getElementById("sendBtn");
const restartBtn = document.getElementById("restartBtn");
const avatar = document.getElementById("avatar");
const profileInput = document.getElementById("profileInput");

let sessionEnded = false;
let inactivityTimer = null;
let processing = false;
const USER_NAME = "Pengguna";

/* Avatar change stored in localStorage */
if(localStorage.getItem("ricaProfilePic")) avatar.src = localStorage.getItem("ricaProfilePic");
avatar.addEventListener("click", ()=> profileInput.click());
profileInput.addEventListener("change", e=>{
  const f = e.target.files[0];
  if(!f) return;
  const r = new FileReader();
  r.onload = ()=>{ avatar.src = r.result; localStorage.setItem("ricaProfilePic", r.result); };
  r.readAsDataURL(f);
});

/* Utilities */
function now(){ const d=new Date(); return d.toLocaleTimeString("id-ID",{hour:"2-digit",minute:"2-digit"}); }
function scrollToBottom(){ chatBody.scrollTop = chatBody.scrollHeight; }

function createMessageEl(who, html){
  const safe = String(html).replace(/<script[\s\S]*?>[\s\S]*?<\/script>/gi, "");
  const el = document.createElement("div");
  el.className = "message " + (who==="bot" ? "bot" : "user");
  el.innerHTML = safe + `<span class="meta">${now()}${who==="bot"? " <span class='check'>✔✔</span>":""}</span>`;
  chatBody.appendChild(el);
  scrollToBottom();
  return el;
}

function setTyping(state=true){
  if(state){
    if(document.getElementById("typingEl")) return;
    const t=document.createElement("div"); t.className="typing"; t.id="typingEl"; t.textContent="Rica sedang mengetik..."; chatBody.appendChild(t); scrollToBottom();
  } else {
    const t=document.getElementById("typingEl"); if(t) t.remove();
  }
}

/* Inactivity auto-end (3 minutes) */
function clearInactivity(){ if(inactivityTimer) clearTimeout(inactivityTimer); inactivityTimer=null; }
function startInactivity(){ clearInactivity(); inactivityTimer = setTimeout(()=>{
  sessionEnded = true;
  createMessageEl("bot", `Sesi chat otomatis diakhiri karena tidak ada aktivitas selama 3 menit 💚<br>Sebelum pergi, mohon beri ulasan layanan kami 👉 <a class="msg-link" href="https://rgsi.shopku.id/digital/366792" target="_blank" rel="noopener">Isi Ulasan Layanan</a>`);
  disableInput();
  restartBtn.style.display = "block";
  restartBtn.setAttribute("aria-hidden","false");
}, 180000); }

function disableInput(){ userInput.disabled=true; sendBtn.disabled=true; userInput.placeholder="Sesi berakhir... tekan Mulai Lagi"; }
function enableInput(){ userInput.disabled=false; sendBtn.disabled=false; userInput.placeholder="Tulis pesan di sini..."; }

/* Local template replies (fast fallback) */
function localTemplateReply(textLower){
  if(textLower.includes("assalamualaikum")) return `Waalaikumsalam, selamat datang di Customer Care Rica Gaming Store Indonesia 🌿<br>Ada yang perlu dibantu kak ${USER_NAME}?<br><br>Klik: <a class="msg-link" href="https://rgsi.shopku.id/digital/366790" target="_blank" rel="noopener">Deskripsi aplikasi</a>`;
  if(textLower.includes("shalom")) return `Shalom juga kak ${USER_NAME} 🙏<br><br>Klik: <a class="msg-link" href="https://rgsi.shopku.id/digital/366790" target="_blank" rel="noopener">Deskripsi aplikasi</a>`;
  if(textLower.includes("salam sejahtera")) return `Salam sejahtera kak ${USER_NAME} 🙏<br><br>Klik: <a class="msg-link" href="https://rgsi.shopku.id/digital/366790" target="_blank" rel="noopener">Deskripsi aplikasi</a>`;
  if(textLower.includes("rica gaming store indonesia")) return `Rica Gaming Store Indonesia adalah aplikasi resmi di bawah PT RSI Group COM (Persero) dengan partnership PT Digiflazz, PT Tripay, dan PT Telkom Indonesia (Persero). 🙌`;
  if(textLower.includes("maintenance")) return "Biasanya dilakukan pukul 13.00 - 17.35 WIB ⏰";
  if(textLower.includes("wa")||textLower.includes("hubungi")) return 'Silahkan klik: <a class="msg-link" href="https://wa.me/6285117535215" target="_blank" rel="noopener">Chat via WhatsApp</a>';
  if(textLower.includes("kenapa kamu dibatasi")) return "Astaga Nagasaki 😏 masih tahap pengembangan kak dan udah paham?";
  if(textLower.includes("paham")) return "Ok baguslah kalau kamu sudah mengerti 😄";
  if(textLower.includes("terima kasih")) return "Iya sama-sama, senang senantiasa bisa bantu kamu secara tulus 🥰🙏";
  if(textLower.includes("akhiri")||textLower.includes("selesai")){ sessionEnded=true; endSession(); return "Kalau Kakanya sudah ga ada obrolan ingin ditanyakan lagi kami akhiri sesi chat kami ya kak! ❤️"; }
  return null;
}

/* Call backend proxy (recommended) */
async function callBackend(message){
  try{
    const url = BACKEND_URL.replace(/\/$/, "") + "/api/rica";
    const res = await fetch(url, {
      method:"POST",
      headers:{"Content-Type":"application/json"},
      body: JSON.stringify({ message })
    });
    if(!res.ok){
      const txt = await res.text();
      throw new Error("Proxy error: " + res.status + " " + txt);
    }
    const j = await res.json();
    if(j.reply) return j.reply;
    if(j.error) throw new Error(j.error || "no-reply");
    return JSON.stringify(j);
  }catch(e){
    console.warn("Proxy failed:", e);
    return null;
  }
}

/* Direct call to Google Gemini (NOT RECOMMENDED for public sites) */
async function callGoogleDirect(message){
  if(!FRONTEND_API_KEY) return null;
  try{
    const payload = { contents: [{ parts: [{ text: message }] }] };
    const res = await fetch(GOOGLE_API_URL + "?key=" + encodeURIComponent(FRONTEND_API_KEY), {
      method:"POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    });
    if(!res.ok){
      const txt = await res.text();
      throw new Error("Google API error: " + res.status + " " + txt);
    }
    const data = await res.json();
    // try a few shapes to extract text
    let text = null;
    if(data?.candidates && data.candidates[0]?.content?.parts && data.candidates[0].content.parts[0]?.text) text = data.candidates[0].content.parts[0].text;
    else if(data?.output && Array.isArray(data.output)) text = data.output.map(o=> o?.content || o?.text || "").join(" ");
    else if(typeof data === "string") text = data;
    return text;
  }catch(err){
    console.warn("Google direct failed:", err);
    return null;
  }
}

/* High-level send handler */
async function handleSend(){
  if(sessionEnded) return;
  if(processing) return;
  const text = userInput.value.trim();
  if(!text) return;
  processing = true;
  createMessageEl("user", escapeHtml(text));
  userInput.value = "";
  clearInactivity();

  const lower = text.toLowerCase();
  const local = localTemplateReply(lower);
  setTyping(true);

  // small delay for typing realism
  await new Promise(r => setTimeout(r, 700));

  // try backend first if configured
  let reply = null;
  if(BACKEND_URL){
    reply = await callBackend(text);
  } else {
    reply = await callGoogleDirect(text);
  }

  // fallback logic: local templates or friendly default
  if(!reply){
    reply = local || localTemplateReply(lower) || "Maaf kak, Rica belum paham maksudnya 😅 coba ketik lebih jelas ya.";
  }

  setTyping(false);
  createMessageEl("bot", reply);
  if(!sessionEnded) startInactivity();
  processing = false;
}

/* End/restart */
function endSession(){
  disableInput();
  restartBtn.style.display="block";
  restartBtn.setAttribute("aria-hidden","false");
}
function restartChat(){
  sessionEnded=false;
  chatBody.innerHTML="";
  enableInput();
  restartBtn.style.display="none";
  restartBtn.setAttribute("aria-hidden","true");
  createMessageEl("bot","Jangan lupa beritahu teman, keluarga/sahabat terdekat untuk gunakan aplikasi Rica Gaming Store 🤗");
  startInactivity();
}

/* Small helpers */
function escapeHtml(s){ return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#039;'}[c])); }
function clearInactivity(){ if(inactivityTimer) clearTimeout(inactivityTimer); inactivityTimer = null; }
function startInactivity(){ clearInactivity(); inactivityTimer = setTimeout(()=>{ sessionEnded=true; createMessageEl("bot", `Sesi chat otomatis diakhiri karena tidak ada aktivitas selama 3 menit 💚<br>Sebelum pergi, mohon beri ulasan layanan kami 👉 <a class="msg-link" href="https://rgsi.shopku.id/digital/366792" target="_blank" rel="noopener">Isi Ulasan Layanan</a>`); disableInput(); restartBtn.style.display="block"; restartBtn.setAttribute("aria-hidden","false"); }, 180000); }

/* Event listeners */
sendBtn.addEventListener("click", handleSend);
userInput.addEventListener("keypress", e => { if(e.key === "Enter") handleSend(); });
restartBtn.addEventListener("click", restartChat);

/* initial state */
startInactivity();
userInput.focus();

</script>
</body>
</html>

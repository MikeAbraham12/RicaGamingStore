here{
  "name": "rica-ai",
  "version": "1.0.0",
  "description": "AI Chat sederhana untuk Rica Assistance",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "axios": "^1.6.7",
    "express": "^4.19.2",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5"
  }
}import express from "express";
import cors from "cors";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 3000;
const API_KEY = process.env.GOOGLE_API_KEY;
const API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent";

app.get("/", (req, res) => {
  res.send("Rica AI server aktif 🚀");
});

app.post("/api/rica", async (req, res) => {
  try {
    const userMessage = req.body.message;
    if (!userMessage) return res.status(400).json({ error: "Pesan kosong" });

    const response = await axios.post(`${API_URL}?key=${API_KEY}`, {
      contents: [{ role: "user", parts: [{ text: userMessage }] }]
    });

    const botReply = response.data.candidates?.[0]?.content?.parts?.[0]?.text || "Rica gak ngerti maksud kamu 😅";
    res.json({ reply: botReply });
  } catch (error) {
    console.error("Error:", error.response?.data || error.message);
    res.status(500).json({ error: "Terjadi kesalahan saat hubungi AI" });
  }
});

app.listen(PORT, () => console.log(`🚀 Rica AI berjalan di port ${PORT}`));📦 RICA AI - Simple Assistant

Cara pakai di Render.com:

1️⃣ Zip semua file ini: server.js, package.json, .env, README.txt  
2️⃣ Upload ke Render > "New Web Service"
3️⃣ Pilih Node.js environment
4️⃣ Build command: npm install
5️⃣ Start command: npm start
6️⃣ Setelah deploy, kamu akan dapat URL API seperti:
   https://rica-ai.onrender.com/api/rica

Cara tes (misal via Postman atau browser console):
POST ke https://rica-ai.onrender.com/api/rica
Body JSON:
{
  "message": "Halo Rica, apa kabar?"
}

/rica-assistance/
├── index.html
├── style.css
├── script.js
├── .env.example
├── README.md
/rica-assistance/
├── index.html      → tampilan chat
├── script.js       → fetch request ke API Gemini
├── style.css       → gaya tampilan (#ED9DB8, model WhatsApp)
js 
const API_KEY = "ISI_API_KEY_KAMU"; // ganti dengan API key kamu
const API_URL =
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent";

const chatBox = document.querySelector(".chat-box");
const userInput = document.querySelector("#user-input");
const sendBtn = document.querySelector("#send-btn");

function addMessage(text, sender) {
  const msg = document.createElement("div");
  msg.classList.add("message", sender);
  msg.innerHTML = `<p>${text}</p>`;
  chatBox.appendChild(msg);
  chatBox.scrollTop = chatBox.scrollHeight;
}

sendBtn.addEventListener("click", async () => {
  const text = userInput.value.trim();
  if (!text) return;
  addMessage(text, "user");
  userInput.value = "";

  addMessage("⏳ Rica is thinking...", "bot");

  try {
    const res = await fetch(`${API_URL}?key=${API_KEY}`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        contents: [{ role: "user", parts: [{ text }] }]
      })
    });

    const data = await res.json();
    document.querySelector(".bot:last-child p").textContent =
      data.candidates?.[0]?.content?.parts?.[0]?.text || "⚠️ No response";
  } catch (err) {
    document.querySelector(".bot:last-child p").textContent =
      "❌ Error: " + err.message;
  }
}); index.html<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Rica Assistance</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="chat-container">
    <div class="chat-header">Rica Assistance 💬</div>
    <div class="chat-box"></div>
    <div class="chat-input">
      <input type="text" id="user-input" placeholder="Ketik pesan..." />
      <button id="send-btn">Kirim</button>
    </div>
  </div>

  <script src="script.js"></script>
</body>
</html>body {
  background-color: #ED9DB8;
  font-family: Arial, sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.chat-container {
  width: 360px;
  height: 600px;
  background: #fff;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}

.chat-header {
  background: #d86a91;
  color: white;
  padding: 15px;
  text-align: center;
  font-weight: bold;
}

.chat-box {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.message {
  padding: 10px 15px;
  border-radius: 15px;
  max-width: 80%;
  word-wrap: break-word;
}

.message.user {
  background: #ED9DB8;
  align-self: flex-end;
  color: #fff;
}

.message.bot {
  background: #f1f1f1;
  align-self: flex-start;
  color: #333;
}

.chat-input {
  display: flex;
  padding: 10px;
  background: #fff;
  border-top: 1px solid #eee;
}

.chat-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 10px;
}

.chat-input button {
  margin-left: 10px;
  background: #d86a91;
  border: none;
  color: white;
  padding: 10px 15px;
  border-radius: 10px;
}

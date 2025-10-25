hereconst PROXY_URL = "https://creative-genie-ce108b.netlify.app";

async function askGemini(message) {
  const apiKey = "AIzaSyDZhpzNLaPSFStnZnXFL_S4dgtyW--ylEk
"; // taruh manual
  const response = await fetch(`${PROXY_URL}/chat?key=${apiKey}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      contents: [{ parts: [{ text: message }] }]
    })
  });
  const data = await response.json();
  console.log(data);
  return data?.candidates?.[0]?.content?.parts?.[0]?.text || "AI tidak merespon 😅";
}

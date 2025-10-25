here[[redirects]]
  from = "/chat"
  to = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent"
  status = 200
  force = true
  headers = { Access-Control-Allow-Origin = "*", Access-Control-Allow-Headers = "Content-Type" }

[[redirects]]
  from = "/ping"
  to = "https://generativelanguage.googleapis.com"
  status = 200
  force = true
  headers = { Access-Control-Allow-Origin = "*" }

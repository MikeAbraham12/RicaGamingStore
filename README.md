# backend.py
# Simple Flask proxy for Google Generative Language (Gemini)
# - Set environment variable GOOGLE_API_KEY before running.
# - Deploy this file to Render/Heroku or any Python host that supports Flask.

from flask import Flask, request, jsonify
from flask_cors import CORS
import os
import requests
import logging

app = Flask(__name__)
CORS(app)  # allow cross-origin from your frontend domain (you can restrict in production)

logging.basicConfig(level=logging.INFO)

GOOGLE_API_KEY = os.environ.get("GOOGLE_API_KEY")
GOOGLE_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent"

if not GOOGLE_API_KEY:
    app.logger.warning("GOOGLE_API_KEY not set. The proxy will not call Google API without it.")

def ask_google_generative(message: str):
    """
    Calls Google Generative Language REST endpoint.
    Returns extracted text reply (or None if failed).
    """
    if not GOOGLE_API_KEY:
        return None

    # Build payload — simple text content shape
    payload = {
        "content": [
            {
                "type": "text",
                "text": message
            }
        ]
    }

    params = {"key": GOOGLE_API_KEY}
    headers = {"Content-Type": "application/json"}

    try:
        resp = requests.post(GOOGLE_URL, params=params, json=payload, headers=headers, timeout=25)
        resp.raise_for_status()
        data = resp.json()
        # Try common shapes returned by API
        # 1) candidates -> content -> parts -> text
        if isinstance(data, dict):
            cand = data.get("candidates")
            if cand and isinstance(cand, list):
                first = cand[0]
                cont = first.get("content") if isinstance(first, dict) else None
                if cont and isinstance(cont, dict):
                    parts = cont.get("parts")
                    if parts and isinstance(parts, list) and parts[0].get("text"):
                        return parts[0]["text"]
            # 2) output array
            if "output" in data and isinstance(data["output"], list):
                return " ".join([ str(o.get("content") or o.get("text") or "") for o in data["output"] ])
            # 3) fallback to any 'response' text-like field
            for k in ("text","reply","message"):
                if k in data and isinstance(data[k], str):
                    return data[k]
        # default: return truncated JSON for debugging
        return (str(data))[:1000]
    except Exception as e:
        app.logger.exception("Google API call failed")
        return None

@app.route("/api/rica", methods=["POST"])
def rica():
    body = request.get_json(silent=True)
    if not body or "message" not in body:
        return jsonify({"error": "missing 'message' in body"}), 400

    message = str(body["message"])

    # quick local templates before external call (same logic as frontend's local templates)
    lower = message.lower()
    if "assalamualaikum" in lower:
        reply = f"Waalaikumsalam, selamat datang di Customer Care Rica Gaming Store Indonesia 🌿\nAda yang bisa dibantu kak Pengguna?\nDeskripsi: https://rgsi.shopku.id/digital/366790"
        return jsonify({"reply": reply})

    # call Google Generative API
    reply = ask_google_generative(message)
    if reply:
        return jsonify({"reply": reply})
    else:
        # fallback reply
        return jsonify({"reply": "Maaf kak, Rica belum paham maksudnya 😅 coba ketik dengan kata yang lebih jelas ya."})

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host="0.0.0.0", port=port, debug=False)￼Enter

# Art Gallery Voice Agent

An AI-powered voice and chat assistant for art gallery exhibitions. Visitors can ask questions about artworks, artists, and the exhibition — by typing or speaking — and receive rich responses including artwork images and detailed descriptions.

> **This is a generic, reusable template.** The included exhibition content (artworks, artist bios, system prompt) is sample data that can be replaced with any gallery's collection.

## 🌐 Live Demo

**[https://artagent.azurewebsites.net/](https://artagent.azurewebsites.net/)**

![Art Gallery Voice Agent Screenshot](https://github.com/user-attachments/assets/d1c3fad7-708e-47e9-b2f7-4700fdfcfe01)

---

## How It Works

### Overview

The application is a Flask web app that acts as an intelligent, voice-enabled guide for any art gallery or museum exhibition. Visitors can interact with the agent via:

- **Text chat** — type questions directly in the browser
- **Voice input** — speak using the microphone button (powered by Azure Speech Services)
- **QR codes** — scan artwork QR codes to immediately ask questions about the artwork in front of them

### Architecture

```
Visitor (Browser)
    │
    ├─ Text/Voice Input
    │
    ▼
Flask Web App (app.py)
    │
    ├─ Azure OpenAI GPT-4o   ← Generates responses (streaming)
    │       │
    │       └─ Tool Calls (function calling)
    │               ├─ artwork_information  → artworks.py (local artwork database)
    │               ├─ get_current_datetime → returns current time
    │               ├─ get_current_weather  → AccuWeather API
    │               └─ search_for_news      → news search
    │
    └─ Azure Speech Services ← Speech-to-text and text-to-speech
```

### Key Components

| File | Description |
|---|---|
| `app.py` | Flask application — routes, session management, streaming response handler |
| `agents.py` | Agent configuration — system prompt, tools list, sample prompts, initial greeting |
| `artworks.py` | Local artwork database with titles, artist info, descriptions (TR/EN), and image paths |
| `tools.py` | Tool function implementations called by the LLM (artwork lookup, weather, news, etc.) |
| `oaistreaming.py` | Azure OpenAI streaming client wrapper |
| `azureopenai.py` | Non-streaming Azure OpenAI client wrapper |
| `config.py` | Loads environment variables (API keys, endpoints) |
| `templates/` | Jinja2 HTML templates (`home.html`, `mainchat.html`) |
| `static/` | CSS, JavaScript, artwork images |

### Conversation Flow

1. **Visitor opens the app** — the agent greets them in Turkish or English based on the `?lang=` query parameter.
2. **Visitor sends a message** (text or voice) — the frontend POSTs it to `/generatestream`.
3. **Streaming response** — GPT-4o processes the message with the exhibition system prompt and streams back a response via Server-Sent Events.
4. **Tool calling** — if the user references an artwork by ID (e.g., "artwork 2"), the model calls the `artwork_information` tool, which returns structured data from `artworks.py` including the image URL.
5. **Rich response** — the agent renders the artwork image inline in the chat alongside a description.
6. **Voice output** — responses can be read aloud via Azure Speech Services text-to-speech.

### Multilingual Support

The agent responds in the same language as the user. Turkish and English are fully supported, with dedicated system prompts, artwork descriptions, and UI labels for each language. Language is toggled via the **TR | EN** switch in the top navigation bar.

### QR Code Integration

Each physical artwork in the gallery has a QR code. Scanning a code opens the chat with a `?qr=<artwork_id>` parameter, which pre-populates the session so the agent immediately knows which artwork the visitor is standing in front of.

---

## Getting Started

### Prerequisites

- Python 3.10+
- An **Azure OpenAI** resource with a `gpt-4o` deployment
- An **Azure Speech Services** resource

### Installation

```bash
git clone https://github.com/mustafaasiroglu/art-gallery-voice-agent.git
cd art-gallery-voice-agent
pip install -r requirements.txt
```

### Configuration

Copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
```

```env
SPEECH_REGION=eastus2
SPEECH_KEY=your_speech_key_here
SPEECH_LANGUAGE=tr-TR
SPEECH_VOICE=en-US-CoraMultilingualNeural
SPEECH_VOICE_2=tr-TR-EmelNeural

OPENAI_ENDPOINT=https://your-openai-endpoint.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2024-08-01-preview
OPENAI_KEY=your_openai_key_here
```

### Run Locally

```bash
flask run
```

Then open [http://localhost:5000](http://localhost:5000) in your browser.

### Deploy to Azure App Service

The repository includes a `.deployment` file for Azure App Service deployment. Deploy using the Azure CLI or GitHub Actions:

```bash
az webapp up --name <your-app-name> --resource-group <your-rg> --runtime PYTHON:3.10
```

---

## Sample Exhibition Data

The repository ships with sample exhibition content to demonstrate how the agent works out of the box. It includes a few example artworks, artist bios, and a system prompt — all defined in `artworks.py` and `agents.py`.

To adapt the agent to your own gallery:

1. **Replace `artworks.py`** — add your own artworks with IDs, titles, artists, descriptions, and image URLs.
2. **Update the system prompt in `agents.py`** — describe your exhibition, venue, and any special instructions for the agent.
3. **Swap artwork images** in `static/images/artworks/`.
4. **Adjust sample prompts and the initial greeting** in `agents.py` to match your exhibition's context.

No other code changes are needed.

---

## Tech Stack

- **Backend**: Python, Flask
- **AI**: Azure OpenAI (GPT-4o) with function calling and streaming
- **Voice**: Azure Speech Services (speech-to-text & text-to-speech)
- **Hosting**: Azure App Service
- **Frontend**: HTML/CSS/JavaScript with Server-Sent Events for streaming

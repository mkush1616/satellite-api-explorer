# 🛰 Red Hat Satellite API Explorer

A containerized, AI-powered web application for discovering, executing, and understanding all Red Hat Satellite REST API endpoints — across every supported Satellite version.

![Tech Stack](https://img.shields.io/badge/Python-Flask-blue) ![AI](https://img.shields.io/badge/AI-Ollama%20%7C%20vLLM-green) ![Container](https://img.shields.io/badge/Container-Docker-blue) ![Satellite](https://img.shields.io/badge/RH%20Satellite-6.2--6.16-red)

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| **Full API Catalog** | Pre-loaded catalog of all known Satellite REST APIs (6.2–6.16) organized by category |
| **Live Discovery** | Connect to your Satellite and auto-discover all APIs from `/apidoc/v2.json` |
| **One-Click Execution** | Execute any API call directly from the browser with real credentials |
| **Version Diff** | Compare API changes between any two Satellite versions — see added, removed, changed endpoints |
| **Hammer → API** | Type any `hammer` command and see exactly which API calls it makes internally (with debug simulation) |
| **AI API Generation** | Describe what you want in plain English; AI builds the complete curl command |
| **Live Execution Log** | Real-time streaming log of all API calls executed, with curl commands |
| **Techy UI** | Terminal/cyberpunk aesthetic — designed for sysadmins who mean business |

---

## 🚀 Quick Start

### Option 1: Docker Compose (recommended)

```bash
git clone https://github.com/YOUR_USERNAME/satellite-api-explorer.git
cd satellite-api-explorer

# Start the app (with Ollama AI backend)
docker compose up -d

# Pull a local AI model for intelligent API generation (optional but recommended)
docker exec -it satellite-ollama ollama pull mistral

# Open in browser
open http://localhost:5000
```

### Option 2: Docker only (no AI)

```bash
docker build -t satellite-api-explorer .
docker run -d -p 5000:5000 --name sat-api satellite-api-explorer
open http://localhost:5000
```

### Option 3: Run directly on RHEL (no container)

```bash
# Install Python 3.9+
dnf install python3 python3-pip -y

cd satellite-api-explorer
pip3 install -r backend/requirements.txt

# Run the server
python3 server.py

# Or with Gunicorn (production)
gunicorn -w 4 -b 0.0.0.0:5000 'backend.app:app'
```

---

## 🤖 AI Backend Setup

The application auto-detects available AI backends in this order:

### 1. Ollama (Recommended for air-gapped/RHEL environments)

[Ollama](https://ollama.ai) runs LLMs locally — no internet required, RHEL-compatible.

```bash
# Install Ollama on RHEL
curl -fsSL https://ollama.ai/install.sh | sh

# Pull a recommended lightweight model
ollama pull mistral          # Excellent for API tasks, ~4GB
# OR
ollama pull llama3.2:3b      # Smaller, faster, ~2GB
# OR  
ollama pull qwen2.5:7b       # Great at structured output, ~5GB

# The app will auto-detect Ollama at http://localhost:11434
```

Set environment variables to customize:
```bash
OLLAMA_URL=http://ollama-host:11434
OLLAMA_MODEL=mistral
```

### 2. OpenAI-Compatible API (vLLM, llama.cpp, LM Studio)

```bash
OPENAI_COMPAT_URL=http://your-server:8080
OPENAI_API_KEY=sk-optional
```

### 3. Rule-Based Fallback (no AI)

Works without any AI backend using keyword pattern matching for 30+ common hammer commands.

---

## 📡 Using the App

### Connect to Your Satellite

1. Click the **Connect** tab in the left sidebar
2. Enter your Satellite URL (e.g., `https://satellite.example.com`)
3. Enter credentials (admin user recommended for full API access)
4. Click **⚡ CONNECT** to save settings
5. Click **🔍 DISCOVER** to pull all available APIs from your live Satellite's `/apidoc`

### Execute an API Call

1. Select any API from the left catalog
2. Connection details auto-fill from saved connection
3. Fill in any path parameters (e.g., replace `:id` with a real ID)
4. Click **⚡ EXECUTE API CALL**
5. See the HTTP response, timing, and the equivalent curl command

### Hammer → API Conversion

The `hammer --debug` mode shows which API calls each hammer command makes. This app simulates that:

1. Click the **Hammer** tab in the sidebar
2. Type any hammer command:
   ```
   hammer host list
   hammer content-view publish --id 5
   hammer job-invocation create --job-template-id 1 --inputs command="uptime"
   ```
3. See the equivalent REST API call, debug simulation, and curl command

### AI API Generation

1. Click the **⬡ AI GENERATE** tab
2. Describe what you want in plain English:
   - *"List all hosts with critical errata that haven't been patched"*
   - *"Find subscriptions expiring in the next 30 days"*
   - *"Promote the latest content view version to production lifecycle environment"*
3. AI analyzes your requirement and generates the API call with:
   - HTTP method + path
   - Required parameters
   - Confidence score
   - Step-by-step explanation of how the API was constructed
   - Ready-to-use curl command

### Version Diff

1. Click the **⇄ VERSION DIFF** tab
2. Select two Satellite versions
3. See a breakdown of:
   - ✚ APIs added in the newer version
   - ✖ APIs removed from the older version  
   - ⇄ APIs that changed (path or method)

---

## 🏗 Architecture

```
satellite-api-explorer/
├── Dockerfile
├── docker-compose.yml
├── README.md
├── backend/
│   ├── app.py              # Flask API server
│   ├── satellite_apis.py   # Full API catalog (6.2–6.16)
│   ├── ai_engine.py        # Ollama/OpenAI AI engine + rule-based fallback
│   └── requirements.txt
└── frontend/
    └── index.html          # Single-page app (no build step needed)
```

### API Endpoints (Backend)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/health` | GET | Health check + AI backend info |
| `/api/catalog?version=6.11` | GET | Get API catalog for a version |
| `/api/versions` | GET | List supported versions |
| `/api/version-diff?v1=6.10&v2=6.11` | GET | Compare two versions |
| `/api/discover` | POST | Connect & discover APIs from live Satellite |
| `/api/execute` | POST | Execute a Satellite API call |
| `/api/hammer-to-api` | POST | Convert hammer command to API |
| `/api/ai-generate` | POST | AI-generate an API call |
| `/api/logs` | GET | Execution log (polling) |
| `/api/logs/stream` | GET | Execution log (SSE streaming) |

---

## 🔧 Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `PORT` | `5000` | HTTP port to listen on |
| `OLLAMA_URL` | `http://localhost:11434` | Ollama server URL |
| `OLLAMA_MODEL` | `mistral` | Ollama model to use |
| `OPENAI_COMPAT_URL` | `` | OpenAI-compatible API URL |
| `OPENAI_API_KEY` | `` | API key for OpenAI-compatible APIs |

---

## 🔒 Security Notes

- Credentials are sent to the **backend container only** — never stored persistently
- All Satellite API calls are proxied through the Flask backend
- SSL verification is configurable (disable only in lab environments)
- The app does **not** log or persist credentials

---

## 🧩 Extending the Catalog

Add custom API entries to `backend/satellite_apis.py`:

```python
BASE_APIS["My Custom Category"] = [
    {
        "id": "my_unique_id",
        "name": "My Custom API",
        "description": "What this API does",
        "method": "GET",
        "path": "/api/v2/my_resource",
        "hammer": "hammer my-command --option value",
        "params": {"organization_id": "1"},
        "body": {}
    }
]
```

---

## 📋 Supported Satellite Versions

6.2, 6.3, 6.4, 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 6.11, 6.12, 6.13, 6.14, 6.15, 6.16

Notable version differences tracked:
- **6.3+**: Remote Execution APIs added
- **6.7+**: Ansible roles/variables APIs added
- **6.10+**: New Global Registration endpoint
- **6.13+**: Content Credentials API, Puppet removed by default

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

*Built for Red Hat Satellite administrators who want API-first control of their infrastructure.*

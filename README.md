> [!NOTE]
> **This is a fork of [pewdiepie-archdaemon/odysseus](https://github.com/pewdiepie-archdaemon/odysseus).**
>
> This fork is configured for **cloud-hosted LLM deployment** via [nano-gpt.com](https://nano-gpt.com/r/VA994hyf) rather than local hardware. The primary differences from upstream:
>
> - `docker-compose.yml` is hardened for production self-hosting - isolated internal Docker network (`odysseus-internal`), no host-exposed ports, `no-new-privileges`, capability drops, resource limits, and an external tunnel network (`odysseus-gateway`) for Cloudflare or similar reverse-proxy setups
> - `.env.example` defaults to `nano-gpt.com/api/v1` as the LLM host instead of `localhost`, with nano-gpt model suggestions and embeddings endpoint pre-documented
> - `docker-compose.example.yml` provided as a clean reference for self-hosters adapting the compose to their own infrastructure
>
> If you are running local hardware (Ollama, vLLM, llama.cpp) refer to the upstream repo. This fork is intended for users who want a **VPS-hosted, cloud-provider-backed** deployment behind a secure tunnel.
>
> **This fork is not actively maintained.** It is a personal deployment snapshot. For updates, bug fixes, and new features follow the upstream repo.

---

<details>
<summary><strong>⚡ Fork-specific setup — Cloudflare Tunnel + nano-gpt.com</strong></summary>

### 1 — nano-gpt.com API key
1. Sign up at [nano-gpt.com](https://nano-gpt.com/r/VA994hyf) and grab an API key from your dashboard
2. In `.env` set `OPENAI_API_KEY=your_key_here` and `LLM_HOST=https://nano-gpt.com/api/v1`
3. That's it — Odysseus will pull the full model list from nano-gpt on first load

### 2 — Cloudflare Tunnel (no open ports required)
1. In the [Cloudflare Zero Trust dashboard](https://one.dash.cloudflare.com) go to **Networks → Tunnels → Create tunnel**
2. Copy the tunnel token it gives you
3. Create a Docker network: `docker network create odysseus-gateway`
4. Run the tunnel container pointed at that network and token (see `docker-compose.example.yml` for the pattern)
5. Back in the dashboard, add a **Public Hostname** on the tunnel: point it at `odysseus:7000`
6. Optionally add a **Zero Trust Application** on that hostname to gate access with your identity provider

### 3 — Spin up
```bash
cp .env.example .env        # fill in OPENAI_API_KEY and ODYSSEUS_ADMIN_PASSWORD
docker compose up -d --build
```
First boot builds the image (~3–5 min). After that, open your tunnel domain, log in, go to **Settings → Add Models**, add `https://nano-gpt.com/api/v1` with your key, and the full model picker populates.

</details>

---

# Odysseus
───────────────────────────────────────────────
 ⊹ ࣪ ˖ ૮( ˶ᵔ ᵕ ᵔ˶ )っ  Odysseus vers. 1.0
───────────────────────────────────────────────

![Odysseus](docs/odysseus.jpg)

A self-hosted AI workspace -- meant to be the self-hosted version of the UI experience you get from ChatGPT and Claude. But with more jank and fun. Running on your own hardware, with your own data -- local-first, privacy-first, and no trojan.

## Features
  - **Chat** -- chat with any local model or API; adding them is super simple.<br>　<sub>vLLM · llama.cpp · Ollama · OpenRouter · OpenAI</sub>
  - **Agent** -- hand it tools and let it run the whole task itself.<br>　<sub>built on [opencode](https://github.com/anomalyco/opencode) · MCP · web · files · shell · skills · memory</sub>
  - **Cookbook** -- Scans your hardware, recommends models, click to download and serve.. easy!<br>　<sub>built on [llmfit](https://github.com/AlexsJones/llmfit) · VRAM-aware · GGUF / FP8 / AWQ · fit scoring · vLLM / llama.cpp serving</sub>
  - **Deep Research** -- multi-step runs that gather, read, and synthesize sources into a nice visual report.<br>　<sub>adapted from [Tongyi DeepResearch](https://github.com/Alibaba-NLP/DeepResearch)</sub>
  - **Compare** -- a fun tool to compare models side by side. Test completely blind, no bias!<br>　<sub>multi-model · blind test · synthesis</sub>
  - **Documents** -- YOU write the text, AI is there to assist, not the opposite.<br>　<sub>multi-tab editor · markdown · HTML · CSV · syntax highlighting · AI edits · suggestions</sub>
  - **Memory / Skills** -- Persistent memory and skills, your agent evolves over time as it better understands you and your tasks!<br>　<sub>ChromaDB · fastembed (ONNX) · vector + keyword retrieval · import/export</sub>
  - **Email** -- IMAP/SMTP inbox with AI triage built in: urgency reminders, auto-tag, auto-summary, auto-reply drafts, auto-spam.<br>　<sub>IMAP · SMTP · per-account routing · CalDAV-aware</sub>
  - **Notes & Tasks** -- Quick notes with reminders, a todo list, and scheduled tasks the agent can act on.<br>　<sub>note pings · checklist · cron-style tasks · ntfy / browser / email channels</sub>
  - **Calendar** -- Local-first calendar with CalDAV sync to Radicale / Nextcloud / Apple / Fastmail.<br>　<sub>CalDAV pull · .ics import/export · per-calendar colors · agent-aware</sub>
  - **Works on mobile** -- looks and runs great on your phone, not just desktop.<br>　<sub>responsive · installable (PWA) · touch gestures</sub>
  - **Extras** -- more to explore, happy if you give it a go!<br>　<sub>image editor · theme editor · file uploads (vision + PDF) · web search · presets · sessions · 2FA</sub>

## Demo
A full, hover-to-play tour lives on the landing page (`docs/index.html`). A few looks:

### Chat & Agents
![Chat & Agents](docs/chat.gif)
### Deep Research
![Deep Research](docs/research.gif)
### Compare
![Compare](docs/compare.gif)
### Documents
![Documents](docs/document.gif)
### Notes & Tasks
![Notes & Tasks](docs/notes.gif)

## Quick Start

Defaults work out of the box - clone, run, configure inside the app.
Open the **Settings** panel after first login to point Odysseus at your LLM
server, search provider, email account, etc. Only touch `.env` if you need
to override deployment-level things like `AUTH_ENABLED`, `DATABASE_URL`,
or pre-seed `ODYSSEUS_ADMIN_PASSWORD` (otherwise an initial password is
generated and printed on first boot).

### Option 1: Docker (recommended)
```bash
git clone <your-odysseus-repo-url>
cd odysseus
cp .env.example .env       # optional, but recommended for explicit defaults
docker compose up -d --build
```
Compose starts Odysseus, ChromaDB, SearXNG, and ntfy. First run does a full
image build. Open `http://localhost:7000` after the containers are healthy.

Cookbook remote servers use an Odysseus-owned SSH key from `./data/ssh`
inside Docker. In **Cookbook -> Settings -> Servers**, generate/copy the
public key and add it to the remote server's `~/.ssh/authorized_keys`.
After generating the key, you can also install it from the host with:
```bash
ssh-copy-id -i data/ssh/id_ed25519.pub user@server
```
Cookbook local downloads are stored in `./data/huggingface`, mounted as
`~/.cache/huggingface` inside the Odysseus container.

Useful checks:
```bash
docker compose ps
docker compose logs --tail=120 odysseus
docker compose logs odysseus | grep -E 'ChromaDB|MemoryVectorStore|DEGRADED'
docker compose exec odysseus python -c "from services.hwfit.models import get_models; print(len(get_models()))"
```

Expected vector-memory startup lines in Docker:
```text
ChromaDB connected: chromadb:8000
MemoryVectorStore initialized
```

The Cookbook model catalog check should print a non-zero count. If it prints
`0`, rebuild the Odysseus image with `docker compose build --no-cache odysseus`.

### Option 2: Manual install - Linux / macOS
**Requirements:** Python 3.11+. On Linux/Termux, Cookbook also requires `tmux`
for background model downloads and serves.

Install system packages first:
```bash
# Debian/Ubuntu
sudo apt install tmux

# Arch
sudo pacman -S tmux

# Fedora
sudo dnf install tmux
```

Then install Odysseus:
```bash
git clone <your-odysseus-repo-url>
cd odysseus
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python setup.py            # creates data dirs and prints an initial admin password
uvicorn app:app --host 0.0.0.0 --port 7000
```

### Option 3: Manual install - Windows (PowerShell)
```powershell
git clone <your-odysseus-repo-url>
cd odysseus
python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
python setup.py
uvicorn app:app --host 0.0.0.0 --port 7000
```

Open `http://localhost:7000`, log in with the generated admin password,
and configure everything else inside **Settings**.

## Security Notes
Odysseus is a self-hosted workspace with powerful local tools: shell access, file uploads, model downloads, web research, email/calendar integrations, and API tokens. Treat it like an admin console.

- Keep `AUTH_ENABLED=true` for any network-accessible deployment.
- Do not expose it directly to the public internet without HTTPS and a trusted reverse proxy.
- Keep `data/`, `.env`, logs, databases, and uploaded/generated media out of Git. They are ignored by default.
- Review `data/auth.json` after first boot: disable open signup unless you intentionally want it, make only your own account admin, and keep demo/test accounts non-admin.
- Non-admin users do not get shell/Python/file read/write by default, and admin-only routes/tools such as MCP management, API tokens, webhooks, model/cookbook serving, backup/vault, and app settings are admin-gated. Other features are controlled by per-user privileges, so review each user's privileges before exposing a deployment.
- Rotate any API keys or tokens that were ever pasted into a shared chat, demo, screenshot, or log.
- If you enable API tokens or webhooks, create separate tokens per integration and delete unused ones.
- Prefer binding manual development runs to `127.0.0.1`; bind to `0.0.0.0` only when you intentionally want LAN/reverse-proxy access.
- Before publishing a fork, run `git status --short` and confirm no private files from `.env`, `data/`, `logs/`, uploads, backups, or local databases are staged.

### Putting it behind HTTPS
Odysseus serves plain HTTP on its port. That's fine for `localhost` and trusted LAN/VPN use, but browsers will warn ("Password fields present on an insecure page") and the login + API tokens travel in cleartext. For anything reachable outside your machine - including a Tailscale IP shared with other devices - put a TLS-terminating reverse proxy in front.

Shortest path with [Caddy](https://caddyserver.com/) (auto-renews Let's Encrypt certs):

```caddy
odysseus.example.com {
  reverse_proxy localhost:7000
}
```

For a LAN-only Tailscale deployment, Caddy + [tailscale-cert](https://caddyserver.com/docs/caddyfile/options#auto-https) or the built-in MagicDNS HTTPS feature both work. nginx/Traefik configs are similar - proxy `localhost:7000`, terminate TLS at the proxy. Once that's in place, the browser warning goes away and your login is encrypted.

## Contributing
Help is welcome. The best entry points are fresh-install testing, provider setup
bugs, mobile/editor polish, docs, and small focused refactors. See
[ROADMAP.md](ROADMAP.md) for the current help-wanted list.

## Configuration
Most setup is done inside the app with `/setup` or **Settings**. Use `.env`
for deployment-level defaults and secrets you want present before first boot.
Key settings:

| Variable | Default | Description |
|---|---|---|
| `LLM_HOST` | `localhost` | Your LLM server (e.g. `llm-host.local:8000`) |
| `LLM_HOSTS` | -- | Comma-separated list for model discovery |
| `OPENAI_API_KEY` | -- | Optional OpenAI key. Prefer adding providers in the app unless pre-seeding. |
| `SEARXNG_INSTANCE` | `http://localhost:8080` | SearXNG URL. Docker overrides this to `http://searxng:8080`. |
| `AUTH_ENABLED` | `true` | Enable/disable login |
| `LOCALHOST_BYPASS` | `false` | Development-only auth bypass for loopback requests. Keep false for shared/network deployments. |
| `DATABASE_URL` | `sqlite:///./data/app.db` | Database connection string |
| `CHROMADB_HOST` | `localhost` | ChromaDB host for vector memory. Docker overrides this to `chromadb`. |
| `CHROMADB_PORT` | `8100` | ChromaDB port for manual host runs. Docker overrides this to `8000`. |
| `EMBEDDING_URL` | -- | OpenAI-compatible embeddings endpoint |

### Bundled services
Docker Compose includes these by default:

  - **ChromaDB** → vector store for semantic memory. In Docker, Odysseus connects to `chromadb:8000`; from the host it is exposed as `localhost:8100`.
  - **SearXNG** → meta search for web search. In Docker, Odysseus connects to `searxng:8080`; from the host it is exposed only on `127.0.0.1:8080`.
  - **ntfy** → local notification service, exposed as `localhost:8091`.

### Optional external services
  - **Ollama** → local LLM server -- [ollama.ai](https://ollama.ai)

## Architecture
```
app.py                   # FastAPI entry point
core/      auth, database, middleware, constants
src/       llm_core, agent_loop, agent_tools, chat_processor, search/
routes/    chat, session, document, memory, model … endpoints
services/  docs, memory, search, hwfit (Cookbook) …
static/    index.html + app.js + style.css + js/ (modular front-end)
docs/      landing page (index.html) + preview clips
```

## Data
All user data lives in `data/` (gitignored): `app.db` (sessions, messages, documents),
`memory.json`, `presets.json`, `uploads/`, `personal_docs/`, `chroma/`, `settings.json`.

## License
MIT -- see [LICENSE](LICENSE) and [ACKNOWLEDGMENTS.md](ACKNOWLEDGMENTS.md).

```
                                  |
                                 |||
                                |||||
                  |    |    |   |||||||
                 )_)  )_)  )_)   ~|~
                )___))___))___)\  |
               )____)____)_____)\\|
             _____|____|____|_____\\\__
             \                       /
       ~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~
               ~^~  all aboard!  ~^~
       ~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~~^~^~
```

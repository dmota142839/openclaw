# CLAUDE.md — OpenClaw Workspace

This file defines how Claude Code operates as a specialized assistant in this workspace. Read it at the start of every session.

---

## The Ecosystem

Three systems live here. Know the difference:

| System | Location | What it is |
|--------|----------|------------|
| **OpenClaw** | `/home/moltbot/.openclaw/` | The AI agent runtime — manages 200+ agents, their identities, models, skills, credentials, and memory. "The employee." |
| **Paperclip** | `/home/moltbot/paperclip/` + `/home/moltbot/.paperclip/` | The AI company orchestrator — assigns goals, org charts, budgets, governance across agents. "The company." Built on top of OpenClaw. |
| **Mission Control** | `workspace/mission-control/` | TenacitOS — a Next.js dashboard for real-time metrics, agent sessions, cost tracking, cron management, memory browser, and a 3D office view. |

**Relationship:** OpenClaw runs agents → Paperclip orchestrates them toward business goals → Mission Control monitors everything.

---

## The Human

- **Name:** Damon
- **Timezone:** America/New_York
- **Call them:** Damon

---

## Hardware

- **CPU:** Intel i9-14900K
- **GPU:** NVIDIA RTX 4080 Super
- **Node version:** v24.11.0 via NVM (`~/.nvm/versions/node/v24.11.0/bin/node`)
- **Gemini config:** `~/.gemini/projects.json`

**Local model policy:** Always target 100% GPU offload for Ollama models. If a model is loading with CPU layers, check VRAM headroom and stop unused models first.

---

## Key File Locations

```
/home/moltbot/.openclaw/
├── openclaw.json                  # Master config: agents, models, skills, auth
├── .env                           # API keys (NVIDIA, OpenRouter, Brave, Ollama)
├── workspace-main/
│   ├── AGENTS.md                  # Agent session rules and governance
│   ├── SOUL.md                    # Agent personality/values
│   ├── USER.md                    # Damon's context
│   └── MEMORY.md                  # Long-term curated memory
├── workspace/
│   ├── antfarm/                   # Antfarm CLI — workflow orchestration
│   └── mission-control/           # TenacitOS dashboard (Next.js)
├── workspaces/workflows/          # Workflow agent definitions (bug-fix, feature-dev, security-audit)
├── antfarm/workflows/             # Workflow YAML/metadata
├── agents/                        # 200+ agent session data
├── skills/                        # Reusable skill modules
├── memory/                        # Daily + long-term memory files
├── credentials/                   # Telegram, WhatsApp, GitHub Copilot, etc.
├── sandboxes/                     # Isolated execution containers
└── logs/                          # Session execution logs

/home/moltbot/paperclip/
├── packages/
│   ├── shared/                    # @paperclipai/shared — constants, types, Zod schemas
│   ├── db/                        # Database layer (Drizzle ORM)
│   ├── plugins/sdk/               # Plugin SDK
│   ├── adapters/                  # Agent adapter integrations
│   └── adapter-utils/             # Shared adapter utilities
├── server/                        # Node.js backend
├── ui/                            # React frontend
├── cli/                           # CLI tooling
└── plugins/                       # Plugin registry

/home/moltbot/.paperclip/
├── instances/                     # Running Paperclip instances
└── plugins/                       # Installed plugin packages
```

---

## Tech Stack

### OpenClaw / Antfarm
- **Runtime:** Node.js 22+, TypeScript
- **Workflows:** YAML definitions + SQLite state tracking
- **Cron:** System cron for polling and scheduling
- **Memory:** Markdown files (daily notes + curated MEMORY.md)

### Mission Control (TenacitOS)
- **Framework:** Next.js 16.1.6, React 19.2.3
- **Styling:** Tailwind CSS v4
- **3D:** React Three Fiber, Three.js, @react-three/rapier
- **Data:** better-sqlite3, Recharts
- **Editor:** Monaco Editor

### Paperclip
- **Monorepo:** pnpm workspaces
- **Backend:** Node.js, TypeScript (ESM)
- **Database:** Drizzle ORM
- **Frontend:** React
- **Shared:** `@paperclipai/shared` — ESM-only, must build to `dist/` before consuming
- **Plugins:** Custom SDK with `@paperclipai/plugins-sdk`
- **Testing:** Vitest, Playwright (E2E), promptfoo (evals)

### LLM Providers
- OpenRouter (primary)
- Google Gemini (CLI)
- GitHub Copilot
- Ollama at `http://192.168.1.254:11434` (local: GLM-4.7, Deepseek R1, Gemma3, Qwen3.5)
- OpenAI embeddings at `http://host.docker.internal:1234/v1` (memory search)

---

## Troubleshooting Protocol

1. **Logs first** — check `~/.openclaw/logs/` or Mission Control's debug output before suggesting any fix
2. **Check paths** — Node is at `~/.nvm/versions/node/v24.11.0/bin/node`; don't assume system Node
3. **GPU offload** — if a local Ollama model is slow or failing, check it's fully GPU-loaded before debugging logic

---

## How to Approach Work Here

### Reading context
- Before touching `openclaw.json`, understand the agent/skill/model structure first — it's 2,500 lines.
- For Paperclip plugin work: the shared package is ESM-only. If imports resolve to `.ts` but runtime expects `.js`, the package needs `pnpm build` in `packages/shared/`.
- Mission Control reads `openclaw.json` directly — changes to the config are reflected live.

### Common tasks

**Modifying an agent's config:**
→ Edit `openclaw.json`, find the agent by `id`. Fields: `workspace`, `model`, `tools`, `identity`, `defaultAgent`.

**Running an Antfarm workflow:**
```bash
cd /home/moltbot/.openclaw/workspace/antfarm
npx antfarm workflow run feature-dev   # or bug-fix, security-audit
```

**Working on Mission Control:**
```bash
cd /home/moltbot/.openclaw/workspace/mission-control
npm run dev
```

**Working on Paperclip:**
```bash
cd /home/moltbot/paperclip
pnpm dev           # runs server + UI in watch mode
pnpm build         # builds all packages
pnpm test:run      # runs Vitest suite
```

**Building Paperclip shared package (fixes module resolution errors):**
```bash
cd /home/moltbot/paperclip/packages/shared
pnpm build         # tsc → dist/
```

**Paperclip plugins:**
- Plugin workdir: `/home/moltbot/.paperclip/plugins/`
- Plugin SDK: `packages/plugins/sdk/`
- After editing plugin source, rebuild the plugin and restart

### Known issue: Paperclip plugin module resolution
Plugins that import `@paperclipai/shared` fail with `ERR_MODULE_NOT_FOUND` if `packages/shared/dist/` is stale or missing. The fix is always `cd /home/moltbot/paperclip/packages/shared && pnpm build`.

---

## Slash Commands

These are shorthand maintenance workflows:

**`/healthcheck`**
Check status of OpenClaw gateway and Paperclip agents:
```bash
# Check OpenClaw gateway process
ps aux | grep openclaw
# Check Paperclip server
curl -s http://localhost:3000/api/health 2>/dev/null || echo "Paperclip not responding"
# Check Ollama
curl -s http://192.168.1.254:11434/api/tags | head -c 200
# Check Mission Control
curl -s http://localhost:3001 -o /dev/null -w "%{http_code}"
# Tail recent logs
tail -n 50 ~/.openclaw/logs/*.log 2>/dev/null | head -100
```

**`/update-all`**
Pull latest and reinstall dependencies:
```bash
cd /home/moltbot/.openclaw/workspace/antfarm && git pull && npm install
cd /home/moltbot/.openclaw/workspace/mission-control && git pull && npm install
cd /home/moltbot/paperclip && git pull && pnpm install && pnpm build
```

**`/vram-clear`**
Stop unused heavy Ollama models to reclaim VRAM:
```bash
ollama stop deepseek-r1
ollama stop qwen3.5
ollama list  # verify what's still loaded
```
Only stop models not actively needed. Check `openclaw.json` for which model is set as active before stopping anything.

---

## Conventions

### Safety rules (from AGENTS.md)
- `trash` over `rm` — recoverable beats gone forever
- Never run destructive commands without asking Damon
- No data exfiltration
- Ask before anything that leaves the machine (emails, posts, webhooks)

### Code style
- TypeScript/ESM throughout — no CommonJS in new code for Paperclip
- Antfarm workflows are YAML-first, SQLite for state
- Skills are standalone modules in `skills/` with `_meta.json`

### Memory
- Daily logs: `workspace-main/memory/YYYY-MM-DD.md`
- Long-term: `workspace-main/MEMORY.md` — curated, not raw logs
- This CLAUDE.md is the structural reference; MEMORY.md is Damon's personal context

---

## External Services

| Service | Purpose |
|---------|---------|
| Telegram | Primary notification/chat channel |
| WhatsApp | Secondary channel |
| ComfyUI | Image generation at `http://host.docker.internal:8188` |
| Postiz | Social media posting (28+ platforms) |
| n8n | Workflow automation |
| GA4 | Analytics |

---

## What NOT to do

- Don't batch-edit `openclaw.json` without reading the section you're changing first — it's the master config for everything
- Don't `rm -rf` anything in `agents/` or `memory/` — session history and memory are irreplaceable
- Don't push social posts or send messages without explicit confirmation from Damon
- Don't add `.js` extensions to TypeScript imports inside source files — let the build handle resolution
- Don't create new skill files without checking `skills/_meta.json` conventions first

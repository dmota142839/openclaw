# Git Fork Setup — Knowledge Base
**Date:** 2026-03-27
**Author:** Damon (dmota142839)
**Purpose:** Full reference for the multi-repo fork setup. Use this to troubleshoot remote issues, credential problems, or sync drift.

---

## Repository Map

| Repo | Local Path | Your Fork (origin) | Upstream |
|------|-----------|-------------------|----------|
| openclaw | `/home/moltbot/.openclaw` | `dmota142839/openclaw` | *(none — local runtime)* |
| mission-control | `/home/moltbot/mission-control` | `dmota142839/mission-control` | `builderz-labs/mission-control` |
| paperclip | `/home/moltbot/paperclip` | `dmota142839/paperclip` | `paperclipai/paperclip` |
| agency-agents | `/home/moltbot/agency-agents` | `dmota142839/agency-agents` | `msitarzewski/agency-agents` |

**Rule:** Always push to `origin` (your fork). Pull updates from `upstream` (original project).

---

## Daily Workflow

```bash
# Push your changes to your fork
git add <files>
git commit -m "your message"
git push origin main          # or master for paperclip

# Pull upstream updates without losing your work
git fetch upstream
git merge upstream/main       # or upstream/master for paperclip

# Check all 4 repos at once (Mission Control view)
for d in mission-control paperclip agency-agents .openclaw; do
  echo -e "\n--- $d ---"
  git -C ~/  "$d" status -s
  git -C ~/"$d" remote -v
done
```

---

## Running the Paperclip Scripts

The `MC_API_KEY` was moved from hardcoded to `.env`. Scripts now read from the environment.

```bash
# JavaScript scripts — use Node's built-in env file flag (Node v20.6+ required; running v25.8.1)
node --env-file=.env scripts/score_leads.js
node --env-file=.env bridge.js

# Bash script — sources .env automatically via export line at top of script
./scripts/lead_scraper.sh "keyword" "location"
```

The key lives in `/home/moltbot/paperclip/.env` (gitignored):
```
MC_API_KEY=967affacc0a03d0e6f667de61cd54d54ede0142a8bdde20264c5f579fe3ab63a
```

---

## Credentials — Where Everything Lives

All API keys are consolidated in `/home/moltbot/.openclaw/.env` (gitignored):

| Variable | Source |
|----------|--------|
| `NVIDIA_API_KEY` | Was already in .env |
| `OLLAMA_API_KEY` | Was already in .env |
| `OPENROUTER_API_KEY` | Was already in .env |
| `BRAVE_API_KEY` | Was already in .env |
| `GOOGLE_API_KEY` | Moved from `openclaw.json` line 121 |
| `LMSTUDIO_API_KEY` | Moved from `openclaw.json` line 233 |
| `OPENCLAW_GATEWAY_TOKEN` | Moved from `openclaw.json` line 2513 |
| `OPENAI_WHISPER_API_KEY` | Moved from `openclaw.json` line 2526 |
| `N8N_API_KEY` | Moved from `openclaw.json` line 2550 |

**Important:** `openclaw.json` still contains these values in-place (the application reads from the file directly — openclaw does not support env var substitution in config). The `.env` entries are the consolidated reference copy. If you rotate a key, update **both** `openclaw.json` and `.env`.

---

## What Is and Isn't Tracked in Each Repo

### openclaw (`/home/moltbot/.openclaw`)
**Tracked:** `CLAUDE.md`, `MEMORY.md`, `.github/`, `agency-agents/` (definitions only), `memory/`, `skills/`, `workspaces/`, `.openclaw.code-workspace`

**Gitignored (sensitive/runtime):**
```
.env                    # API keys
openclaw.json           # embedded API keys — manage locally
credentials/            # tokens, auth files
identity/               # agent identity
logs/ cache/ completions/ browser/ delivery-queue/ canvas/ cron/ devices/
agents/                 # session data only
workspace/              # nested git repos (antfarm, mission-control)
workspace-main/         # has its own .git
sandboxes/ telegram/ sandbox/ subagents/ antfarm/
*.bak  *.bak-*  archive-*/  *.db  *.db-shm  *.db-wal
```

### mission-control (`/home/moltbot/mission-control`)
**Gitignored (sensitive):**
```
.env                    # gateway token + auth credentials
.deploy-credentials.txt # AUTH_USER / AUTH_PASS
mc-*.log                # runtime logs
```

### paperclip (`/home/moltbot/paperclip`)
**Gitignored (sensitive):**
```
.env                    # MC_API_KEY and others
server/config-backup.json
skills/                 # custom playbooks (untracked for now)
```

### agency-agents (`/home/moltbot/agency-agents`)
**Gitignored:**
```
.cursor/ .opencode/ .qwen/   # IDE/AI tool config dirs
```

---

## Sync Status (as of 2026-03-27)

| Repo | Upstream commits merged |
|------|------------------------|
| mission-control | ✅ Merged 3 upstream commits |
| agency-agents | ✅ Merged 243 upstream commits (conflict in `.gitignore` resolved — kept both `!integrations/kimi/README.md` and local IDE ignores) |
| paperclip | ✅ Merged 54 upstream commits |
| openclaw | N/A (no upstream) |

---

## Troubleshooting

### "rejected — non-fast-forward" on push
Your fork has upstream commits your local doesn't have yet.
```bash
git fetch upstream
git merge upstream/main --no-edit   # resolve conflicts if any
git push origin main
```

### Merge conflict in .gitignore
Keep BOTH sides — your custom ignores AND upstream's new ignores. Never discard either block.

### Secret scan (run before any big push)
```bash
grep -rEn "(PASSWORD|SECRET|TOKEN|KEY|AUTH_PASS)\s*[=:]\s*['\"]?[A-Za-z0-9+/\-_]{16,}" \
  . --exclude-dir={.git,node_modules,.next,dist,build} \
  --include="*.json" --include="*.env*" --include="*.sh" --include="*.js" --include="*.ts" \
  | grep -v "process\.env\|\${" | head -30
```

### Verify a file is gitignored
```bash
git check-ignore -v <filename>
# Expected output: .gitignore:<line>:<rule>   <filename>
# If no output: NOT gitignored — fix before pushing
```

### Mission Control view of all repos
```bash
for d in mission-control paperclip agency-agents .openclaw; do
  echo -e "\n--- $d ---"
  git -C /home/moltbot/"$d" status -s 2>/dev/null
  git -C /home/moltbot/"$d" remote -v 2>/dev/null
done
```

---

## Node Version Note
System Node: **v25.8.1** (via NVM). The `--env-file` flag requires Node v20.6+. No `dotenv` package needed.

Check active version: `node --version`
NVM path: `~/.nvm/versions/node/` (v24.11.0 directory was removed/not present — using system node)

# OpenClaw Deployment Guide

## Overview

This repository has been migrated from **Clawdbot** to **OpenClaw** (February 2026) due to trademark requirements. Both development (MacBook Pro) and production (Mac Mini) environments are configured and synchronized.

---

## Environment Setup

### Production (Mac Mini - macmini-ts)
- **Purpose:** Production gateway running 24/7
- **OS:** macOS 26.2 (arm64)
- **Node:** 25.4.0
- **OpenClaw:** 2026.2.14
- **Access:** `ssh macmini-ts` (Tailscale)
- **Config:** `~/.openclaw/openclaw.json`
- **Gateway:** Running via LaunchAgent (`ai.openclaw.gateway`)
- **Workspace:** `/Users/marcobianco/code/critical-brain`
- **Skills:** 79 active (56 from workspace + ~40 bundled)

### Development (MacBook Pro)
- **Purpose:** Development and testing
- **OS:** macOS 26.2 (arm64)
- **Node:** 24.12.0
- **OpenClaw:** 2026.2.14
- **Config:** `~/.openclaw/openclaw.json` (synced with production)
- **Workspace:** `/Users/marcobianco/code/critical-brain`
- **Gateway:** Stopped (only runs when needed for testing)

---

## Security Configuration

### Applied Hardening (February 2026)

#### Network Security
- **Gateway binding:** Loopback only (`127.0.0.1:18789`)
- **Authentication:** Token-based (never exposed publicly)
- **mDNS:** Minimal mode (reduced network broadcasting)
- **Tailscale:** Off (loopback access only)

#### Access Control
- **Elevated tools:** Restricted to owner only (`["owner"]`)
- **Session isolation:** Per-channel-peer (isolated user contexts)
- **DM policy:** Pairing required for Telegram
- **Group policy:** Allowlist with mention-gating

#### File Security
- **Permissions:**
  - Config: `600` (user read/write only)
  - State dir: `700` (user only)
  - Credentials: `600`
- **API keys:** Stored in environment variables (not hardcoded)
- **Secrets location:** `~/.openclaw/credentials/`

#### Development Access
- **Sandboxing:** Disabled (`mode: "off"`) for full dev capabilities
- **Workspace:** Read/write access
- **Tools:** Native execution (no container isolation)
- **Rationale:** Owner-only personal dev machine, prioritize functionality over isolation

### Security Audit Status
```bash
openclaw security audit
# Summary: 0 critical · 1 warn · 2 info
```

---

## Voice Call Integration

### Twilio Configuration
- **Provider:** Twilio
- **From Number:** +1 (310) 919-2338
- **To Number:** +1 (310) 975-5297
- **Inbound Policy:** Allowlist (only +13109755297)
- **Greeting:** "Hello Marco! I'm Claude, your AI assistant. What can I help you with?"

### TTS Providers
- **Primary:** OpenAI TTS (via `OPENAI_VOICE_API_KEY`)
- **Secondary:** Talk API (via `TALK_API_KEY`)

### Webhook Infrastructure
- **Local server:** `http://127.0.0.1:3334/voice/webhook`
- **WebSocket stream:** `ws://127.0.0.1:3334/voice/stream`
- **Public tunnel:** Cloudflare Tunnel
  - Current: `https://europe-furthermore-consider-jan.trycloudflare.com` (temporary)
  - Setup script: `/tmp/setup-permanent-tunnel.sh` (for permanent URL)

### Security Features
- **Signature verification:** Enabled (validates Twilio requests)
- **API keys:** Stored in environment variables
- **Allowlist:** Single phone number permitted

### Testing Voice Calls
```bash
# Call the number
# Expected: Greeting plays, can interact with Claude via voice
# Troubleshoot: Check tunnel status
ps aux | grep cloudflared
```

---

## Configuration Files

### Primary Config Location
```
~/.openclaw/openclaw.json
```

### Key Configuration Sections

#### Agents
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5-20250929"
      },
      "workspace": "/Users/marcobianco/code/critical-brain",
      "sandbox": {
        "mode": "off"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  }
}
```

#### Tools & Elevated Access
```json
{
  "tools": {
    "elevated": {
      "enabled": true,
      "allowFrom": {
        "telegram": ["owner"]
      }
    },
    "exec": {
      "security": "full",
      "ask": "off"
    }
  }
}
```

#### Session Isolation
```json
{
  "session": {
    "dmScope": "per-channel-peer"
  }
}
```

#### Discovery
```json
{
  "discovery": {
    "mdns": {
      "mode": "minimal"
    }
  }
}
```

### Environment Variables
All API keys are stored in `env.vars`:
- `ANTHROPIC_API_KEY`
- `GEMINI_API_KEY`
- `OPENAI_VOICE_API_KEY`
- `TALK_API_KEY`
- `TODOIST_API_TOKEN`
- `RUNWAYML_API_KEY` / `RUNWAY_API_KEY`
- `HIGGSFIELD_KEY_ID` / `HIGGSFIELD_KEY_SECRET`
- `FAL_KEY_ID` / `FAL_KEY_SECRET`
- `CELLCOG_API_KEY`
- `ELEVENLABS_API_KEY`

---

## Channels

### Telegram
- **Status:** Enabled
- **Bot Token:** Configured via `channels.telegram.botToken`
- **DM Policy:** Pairing (requires verification code)
- **Group Policy:** Allowlist with mention-gating
- **Elevated Access:** Owner only

### Slack
- **Status:** Enabled
- **Mode:** Socket mode
- **Bot Token:** Configured
- **App Token:** Configured
- **Group Policy:** Allowlist

### Google Chat
- **Status:** Enabled but not configured
- **Setup Required:** Yes

---

## Skills & Plugins

### Skill Locations
1. **Bundled:** ~40 skills (built into OpenClaw)
2. **Workspace:** 56 skills (`~/code/critical-brain/skills/`)
3. **Runtime:** Custom additions (`~/.openclaw/skills/soundcore-sync`)

### Active Skills (79/101 ready)
**Key skills with API keys:**
- `agent-browser` - Browser automation
- `todoist` - Task management
- `tavily-search` - Web search
- `perplexity-sonar` - AI search
- `openai-image-gen` - Image generation
- `openai-whisper-api` - Speech transcription
- `runway` / `veo2` / `ai-video-gen` - Video generation
- `cellcog` - #1 DeepResearch Bench, any-to-any AI (multi-modal inputs/outputs)

**Automation & workflows:**
- `n8n-workflow-automation` - Generate n8n workflows with error handling, retries, logging

**System integration:**
- `apple-notes` - Apple Notes management
- `apple-reminders` - Reminders management
- `voice-call` - Twilio voice integration

### Skill Configuration
```bash
# List all skills
openclaw skills list

# Check specific skill config
openclaw config get skills.entries.todoist
```

---

## Gateway Management

### Start/Stop/Restart
```bash
# Production (Mac Mini)
openclaw gateway start
openclaw gateway stop
openclaw gateway restart

# Check status
openclaw gateway status
```

### LaunchAgent (macOS)
```bash
# Service file
~/Library/LaunchAgents/ai.openclaw.gateway.plist

# Check if loaded
launchctl print gui/$UID | grep openclaw

# Manual load/unload
launchctl load ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl unload ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

### Logs
```bash
# Gateway logs
tail -f ~/.openclaw/logs/gateway.log

# System logs (macOS)
./scripts/clawlog.sh

# Today's gateway log
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

---

## Development Workflow

### Local Development (MacBook)
```bash
# Work on clawdbot core
cd ~/code/clawdbot
pnpm install
pnpm dev

# Test locally
pnpm clawdbot ...

# Run tests
pnpm test
```

### Deploying to Production (Mac Mini)
```bash
# Option 1: Push changes and pull on production
git add .
git commit -m "Your changes"
git push origin main
ssh macmini-ts "cd ~/clawdbot && git pull"

# Option 2: Direct SCP for quick tests
scp -r skill-folder/ macmini-ts:~/code/critical-brain/skills/

# Restart gateway after changes
ssh macmini-ts "bash -lc 'openclaw gateway restart'"
```

---

## Troubleshooting

### Gateway Won't Start
```bash
# Check logs
tail -100 ~/.openclaw/logs/gateway.log

# Check if already running
ps aux | grep openclaw | grep -v grep

# Check port availability
lsof -i :18789
```

### Voice Calls Not Working
```bash
# Check tunnel status
ps aux | grep cloudflared

# Check webhook server
lsof -i :3334

# Test webhook endpoint
curl http://127.0.0.1:3334/voice/webhook

# View voice logs
tail -f ~/.openclaw/logs/gateway.log | grep voice
```

### Skills Not Loading
```bash
# Check skill status
openclaw skills list

# Verify workspace path
openclaw config get agents.defaults.workspace

# Check skill directories
ls ~/code/critical-brain/skills/
ls ~/.openclaw/skills/
```

### API Key Issues
```bash
# Check environment variables
openclaw config get env.vars

# Verify specific key
openclaw config get env.vars.ANTHROPIC_API_KEY

# Test if key works (example)
openclaw config get skills.entries.todoist.apiKey
```

---

## Migration Notes

### Clawdbot → OpenClaw (February 2026)

**Changes:**
- Binary: `clawdbot` → `openclaw`
- State directory: `~/.clawdbot` → `~/.openclaw` (symlinked for compatibility)
- Config file: `clawdbot.json` → `openclaw.json` (auto-migrated)
- Service: `com.clawdbot.gateway` → `ai.openclaw.gateway`

**Migration was automatic:**
- First run of `openclaw` migrated config
- Legacy symlinks created for compatibility
- All settings preserved

**Verify migration:**
```bash
openclaw doctor
```

---

## Maintenance

### Regular Tasks

**Daily:**
- Monitor gateway uptime: `openclaw status`
- Check security: `openclaw security audit`

**Weekly:**
- Review logs: `tail ~/.openclaw/logs/gateway.log`
- Check for updates: `npm view openclaw version`

**Monthly:**
- Update OpenClaw: `npm install -g openclaw@latest`
- Restart gateway: `openclaw gateway restart`
- Backup config: `cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup-$(date +%Y%m%d)`

### Updates
```bash
# Check current version
openclaw --version

# Update to latest
npm install -g openclaw@latest

# Restart gateway
openclaw gateway restart

# Verify update
openclaw --version
openclaw doctor
```

---

## References

- **OpenClaw Docs:** https://docs.openclaw.ai/
- **Security Guide:** https://docs.openclaw.ai/gateway/security
- **GitHub Repo:** https://github.com/openclaw/openclaw
- **This Fork:** https://github.com/mmbianco78/clawdbot-1

---

## Support

- **Issues:** https://github.com/openclaw/openclaw/issues
- **Docs:** https://docs.openclaw.ai/
- **Help:** `openclaw --help`

---

**Last Updated:** February 15, 2026
**OpenClaw Version:** 2026.2.14
**Security Audit:** 0 critical issues

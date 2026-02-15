# Mac Mini Production Setup

Quick reference for the Mac Mini production environment.

## SSH Access
```bash
ssh macmini-ts  # ALWAYS use this alias
```

**Never use:**
- ~~`ssh marco@...`~~ (wrong username)
- ~~Raw IPs~~ (use the alias)

## Quick Commands

### Gateway
```bash
# Status
openclaw gateway status
openclaw status

# Restart
openclaw gateway restart

# Logs
tail -f ~/.openclaw/logs/gateway.log
```

### Channels
```bash
# Check all channels
openclaw channels status

# Test with probe
openclaw channels status --probe
```

### Skills
```bash
# List all
openclaw skills list

# Count ready skills
openclaw skills list | grep ready | wc -l
```

### Security
```bash
# Run audit
openclaw security audit

# Deep scan
openclaw security audit --deep

# Fix issues
openclaw security audit --fix
```

## File Locations

| What | Path |
|------|------|
| **Binary** | `/opt/homebrew/bin/openclaw` |
| **Config** | `~/.openclaw/openclaw.json` |
| **Gateway logs** | `~/.openclaw/logs/gateway.log` |
| **Session logs** | `~/.openclaw/agents/main/sessions/` |
| **Skills (workspace)** | `~/code/critical-brain/skills/` (56 skills) |
| **Skills (runtime)** | `~/.openclaw/skills/` (soundcore-sync) |
| **Credentials** | `~/.openclaw/credentials/` |
| **Code repo** | `~/clawdbot/` |

## Critical Services

### OpenClaw Gateway
```bash
# LaunchAgent
~/Library/LaunchAgents/ai.openclaw.gateway.plist

# Check if running
launchctl print gui/$UID | grep openclaw
# or
ps aux | grep openclaw | grep gateway
```

### Cloudflare Tunnels (Voice)
```bash
# Check tunnels
ps aux | grep cloudflared

# Should see 2 processes:
# - Port 3334 (voice webhook)
# - Port 3456 (other service)
```

## Configuration Quick Reference

### Current Settings
- **Gateway:** Loopback only, port 18789
- **Workspace:** `/Users/marcobianco/code/critical-brain`
- **Model:** Claude Sonnet 4.5 (anthropic/claude-sonnet-4-5-20250929)
- **Sandbox:** OFF (full dev access)
- **Elevated tools:** Owner only
- **Session isolation:** Per-channel-peer
- **mDNS:** Minimal

### Voice Call
- **From:** +1 (310) 919-2338
- **To:** +1 (310) 975-5297
- **Webhook:** https://europe-furthermore-consider-jan.trycloudflare.com
- **Local:** http://127.0.0.1:3334/voice/webhook

## Emergency Commands

### Gateway Stuck
```bash
# Kill and restart
pkill -9 -f openclaw
openclaw gateway restart
```

### Reset Config (DANGER)
```bash
# Backup first!
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.emergency-backup

# Then reset if needed
openclaw onboard
```

### Check System Health
```bash
openclaw doctor
openclaw status --deep
```

## Common Tasks

### Update OpenClaw
```bash
npm install -g openclaw@latest
openclaw gateway restart
openclaw --version
```

### Add API Key
```bash
openclaw config set env.vars.NEW_API_KEY "key-value"
openclaw gateway restart
```

### Test Voice Call
```bash
# 1. Check tunnel
ps aux | grep cloudflared

# 2. Check webhook server
lsof -i :3334

# 3. Call the number
# From: +1 (310) 975-5297
# To: +1 (310) 919-2338

# 4. Check logs during call
tail -f ~/.openclaw/logs/gateway.log | grep voice
```

## Security Checklist

- [ ] Gateway on loopback only
- [ ] Elevated tools restricted to owner
- [ ] API keys in environment variables
- [ ] File permissions: config=600, dir=700
- [ ] Security audit clean (0 critical)
- [ ] Voice signature verification enabled
- [ ] Session isolation active
- [ ] mDNS broadcasting minimal

---

**Last Updated:** February 15, 2026

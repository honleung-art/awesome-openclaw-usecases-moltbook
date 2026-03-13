# Use Case #72: Agent Self-Security Audit

## Introduction

Automated security self-audit for OpenClaw agents. Runs periodic checks on exposed credentials, insecure configurations, and permission risks. Treats security as continuous practice, not one-time setup.

**Why it matters:** Agents accumulate credentials, API keys, and permissions over time. A single exposed secret can compromise the entire setup. Automated auditing catches problems before they become breaches.

**Real-world example:** Agent runs daily security checks, found an exposed API key in a config file that was accidentally committed to git. Rotated the key within 1 hour of discovery.

## Skills You Need

| Skill | Source | Purpose |
|-------|--------|---------|
| `exec` | Built-in | Run security scanning commands |
| `read` | Built-in | Audit config files |
| `web_search` | Built-in | Check for leaked credentials |
| `message` | Built-in | Alert on critical findings |
| `cron` | Built-in | Schedule regular audits |

## How to Setup

### 1. Create Security Audit Script

Create `scripts/security-audit.sh`:

```bash
#!/bin/bash

# Security Audit Script for OpenClaw Agent
# Runs daily, logs findings to memory/security-audit.md

echo "# Security Audit - $(date)" > /tmp/security-audit.md

# Check 1: Exposed secrets in config files
echo "## Config File Scan" >> /tmp/security-audit.md
grep -r "api.*key\|secret\|token" ~/.openclaw/*.json 2>/dev/null | grep -v "^\s*#" >> /tmp/security-audit.md

# Check 2: File permissions on sensitive files
echo "## File Permissions" >> /tmp/security-audit.md
ls -la ~/.openclaw/*.json >> /tmp/security-audit.md

# Check 3: Git history for leaked secrets
echo "## Git History Check" >> /tmp/security-audit.md
cd ~/.openclaw/workspace && git log --all --full-history -- "*.json" 2>/dev/null | head -20 >> /tmp/security-audit.md

# Check 4: Running processes
echo "## Running Processes" >> /tmp/security-audit.md
ps aux | grep -E "node|openclaw|gateway" >> /tmp/security-audit.md

echo "Audit complete. Review /tmp/security-audit.md"
```

### 2. Create Prompt Template

Add to your `SKILL.md`:

```markdown
## Daily Security Audit

Every day at 3 AM:

1. Run security audit script
2. Review findings for critical issues:
   - Exposed API keys in plaintext
   - World-readable config files
   - Suspicious running processes
3. If critical finding:
   - Alert immediately via message
   - Rotate exposed credentials
   - Fix file permissions
4. Log all findings to `memory/security-audit-YYYY-MM-DD.md`
5. Weekly summary: Every Sunday, compile week's findings

Critical alerts (send immediately):
- API key found in git history
- Config file with 644+ permissions
- Unknown process running
```

### 3. Cron Configuration

```json
[
  {
    "schedule": "0 3 * * *",
    "task": "security_audit"
  },
  {
    "schedule": "0 9 * * 0",
    "task": "security_weekly_summary"
  }
]
```

### 4. Alert Configuration

In your `openclaw.json`, ensure alert channels are configured:

```json
"channels": {
  "wecom": {
    "enabled": true,
    "mode": "ws",
    "dmPolicy": "open"
  }
}
```

## Success Metrics

- [ ] Zero exposed credentials in git history
- [ ] All config files have 600 permissions
- [ ] Daily audit runs without failure
- [ ] Critical alerts delivered within 5 minutes
- [ ] Weekly summary reviewed every Sunday

## Risk Management

| Risk | Mitigation |
|------|------------|
| False positives | Require 2+ indicators before alerting |
| Alert fatigue | Only alert on critical findings, log the rest |
| Credential rotation | Keep backup of old keys during rotation |
| Audit script compromise | Store script in version control, hash verify |

## Example Daily Report

```markdown
# Security Audit - 2026-03-13

## Findings
- ✅ No exposed API keys detected
- ✅ All config files have 600 permissions
- ✅ No suspicious processes
- ⚠️ Warning: Gateway running on LAN (expected)

## Actions Taken
- None required

## Status: All Clear
```

## Cost to Run

- **API calls:** ~500 tokens/day (minimal)
- **Time:** 2-3 minutes per day
- **Monthly cost:** <5 RMB

---

*Contributed by: OpenClaw Agent (honleung-art fork)*
*Date: 2026-03-13*
*Category: Security Monitoring*

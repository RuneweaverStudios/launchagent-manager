---
name: launchagent-manager
description: List, classify, prune LaunchAgents; analyze openclaw.json so the proper gateway LaunchAgent remains connected and tokens match. Keeps only OpenClaw-related agents; can unload/delete others.
version: 1.0.0
---

# LaunchAgent Manager

Manages LaunchAgents in `~/Library/LaunchAgents` and analyzes **openclaw.json** so the **gateway LaunchAgent** stays correct: loaded when it should be, and **tokens matching** config vs running gateway.

- **List/prune:** Classify agents as OpenClaw (keep) or other (prune). OpenClaw = Label or path contains "openclaw".
- **Config check:** Read `openclaw.json` → gateway port, auth mode, token set; find the gateway plist (e.g. ai.openclaw.gateway); report loaded? running? tokens match? (Uses gateway-guard status when available.) Recommends loading the plist or running gateway-guard ensure --apply if needed.
- **--fix:** With --config, can load the gateway plist if not loaded and run gateway-guard ensure --apply if tokens mismatch.

## When to use

- **Ensure gateway stays connected:** Run `--config` to verify the gateway LaunchAgent is loaded and tokens match; use `--config --fix` to load plist and sync auth.
- List what's running: see OpenClaw vs other agents.
- Remove non-OpenClaw LaunchAgents: prune so only OpenClaw daemons remain.

## Commands (use absolute path for exec from any cwd)

```bash
python3 <skill-dir>/scripts/launchagent_manager.py [--list] [--json]
python3 <skill-dir>/scripts/launchagent_manager.py --config [--fix] [--json]
python3 <skill-dir>/scripts/launchagent_manager.py --prune [--dry-run]
python3 <skill-dir>/scripts/launchagent_manager.py --prune --apply [--delete-plists]
```

- **--list** (default) — List all LaunchAgents; show OpenClaw (kept) vs others (prune targets). Shows loaded/unloaded.
- **--config** — Analyze openclaw.json and gateway LaunchAgent: config path, gateway port, auth, token set; gateway plist label and loaded?; gateway process running?; tokens match (config vs running)? Recommendations if plist not loaded or tokens mismatch. Exit 0 if all ok, 1 if action needed.
- **--config --fix** — If gateway plist not loaded: run `launchctl load <plist>`. If tokens don't match: run gateway-guard `ensure --apply --json`. Requires gateway-guard skill.
- **--config --json** — Machine-readable report: config_path, gateway, gateway_launchagent, gateway_loaded, tokens_match, gateway_running, recommendations.
- **--json** — For --list: `{ "openclaw": [...], "others": [...] }`.
- **--prune** — Operate on non-OpenClaw agents. Without --apply this is a dry-run (show what would be unloaded).
- **--prune --dry-run** — Only show what would be unloaded.
- **--prune --apply** — Unload each non-OpenClaw LaunchAgent. Plist files kept unless --delete-plists.
- **--prune --apply --delete-plists** — Unload and delete plist files (backed up to OPENCLAW_HOME/backups/launchagents). Requires `--yes` to confirm.
- **--yes / -y** — Skip confirmation prompt for destructive operations (required with `--delete-plists`).

## Usage Examples

### List all LaunchAgents

```bash
$ python3 scripts/launchagent_manager.py --list
OpenClaw LaunchAgents (kept):
   ai.openclaw.gateway — loaded
   com.openclaw.watchdog — loaded
Other LaunchAgents (prune with --prune --apply):
   com.apple.Safari — unloaded — /Users/you/Library/LaunchAgents/com.apple.Safari.plist
   com.spotify.webhelper — loaded — /Users/you/Library/LaunchAgents/com.spotify.webhelper.plist
```

### Config check (JSON output)

```bash
$ python3 scripts/launchagent_manager.py --config --json
{
  "ok": true,
  "config_path": "/Users/you/.openclaw/openclaw.json",
  "gateway": {
    "port": 18789,
    "auth_mode": "token",
    "token_set_in_config": true,
    "config_ok": true
  },
  "gateway_launchagent": {
    "label": "ai.openclaw.gateway",
    "path": "/Users/you/Library/LaunchAgents/ai.openclaw.gateway.plist"
  },
  "gateway_loaded": true,
  "tokens_match": true,
  "recommendations": []
}
```

### Prune dry-run

```bash
$ python3 scripts/launchagent_manager.py --prune --dry-run
Non-OpenClaw LaunchAgents (would be unloaded):
   com.spotify.webhelper /Users/you/Library/LaunchAgents/com.spotify.webhelper.plist
OpenClaw (kept): ai.openclaw.gateway, com.openclaw.watchdog
```

### Prune with deletion (requires --yes)

```bash
$ python3 scripts/launchagent_manager.py --prune --apply --delete-plists --yes
Unloaded 1 non-OpenClaw agent(s).
```

## Safety

- Only user domain is touched: `~/Library/LaunchAgents/`. System domain is not modified.
- OpenClaw detection is conservative: Label `com.openclaw.*` or any ProgramArgument containing "openclaw" → kept.
- With --delete-plists, backups are written to `OPENCLAW_HOME/backups/launchagents/` before deletion.

## Requirements

- macOS (launchctl, plist in user LaunchAgents).
- Python 3 with plistlib (standard library).

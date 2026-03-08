# LaunchAgent Manager | OpenClaw Skill

Manages macOS LaunchAgents for OpenClaw: lists, classifies, prunes non-OpenClaw agents, and verifies gateway LaunchAgent configuration including token synchronization.

## Features

- **List and classify** all `~/Library/LaunchAgents` plists as OpenClaw (kept) or other (prune targets)
- **Config check** (`--config`): Analyze `openclaw.json` gateway settings, verify the gateway LaunchAgent is loaded and running, check token synchronization between config and running gateway
- **Auto-fix** (`--config --fix`): Load gateway plist if unloaded, run `gateway-guard ensure --apply` if tokens mismatch
- **Prune** (`--prune`): Unload non-OpenClaw LaunchAgents with dry-run preview and optional plist deletion with backup
- **JSON output** (`--json`): Machine-readable output for all operations including list, config check, prune, and dry-run

## Quick Start

```bash
clawhub install launchagent-manager
# or: git clone https://github.com/RuneweaverStudios/launchagent-manager.git workspace/skills/launchagent-manager

# List all LaunchAgents
python3 scripts/launchagent_manager.py --list

# Check gateway config
python3 scripts/launchagent_manager.py --config --json

# Preview what would be pruned
python3 scripts/launchagent_manager.py --prune --dry-run --json

# Actually prune
python3 scripts/launchagent_manager.py --prune --apply
```

## Requirements

- macOS (uses `launchctl` and `~/Library/LaunchAgents`)
- Python 3 with `plistlib` (standard library)
- Optional: `gateway-guard` skill for token sync verification and `--config --fix`

## Commands

| Command | Description |
|---------|-------------|
| `--list` (default) | List all LaunchAgents, classify as OpenClaw or other |
| `--list --json` | JSON output: `{ "openclaw": [...], "others": [...] }` |
| `--config` | Analyze gateway config, check loaded/running/tokens |
| `--config --json` | Machine-readable config report |
| `--config --fix` | Auto-load gateway plist, sync tokens via gateway-guard |
| `--prune --dry-run` | Preview what would be unloaded |
| `--prune --dry-run --json` | JSON preview of prune targets |
| `--prune --apply` | Unload non-OpenClaw agents |
| `--prune --apply --delete-plists --yes` | Unload and delete plists (with backup) |

## Safety

- Only touches user domain (`~/Library/LaunchAgents`), never system domain
- OpenClaw detection is conservative: label or ProgramArguments containing "openclaw"
- Plist deletion always backs up to `$OPENCLAW_HOME/backups/launchagents/`
- Destructive operations require `--yes` confirmation

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENCLAW_HOME` | No | OpenClaw workspace root (default: `~/.openclaw`) |

## License / Author

OpenClaw. Part of the OpenClaw skills ecosystem.

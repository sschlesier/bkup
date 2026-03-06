# bkup

Personal macOS backup tool. Backs up directories to a restic repository on Backblaze B2, scheduled nightly via launchd.

## Architecture

Three bash scripts in `bin/`, no build system:

- `bin/bkup` — main backup script; loads env, validates config, runs `restic backup` then `restic forget --prune`
- `bin/bkup-init` — interactive setup wizard; writes `~/.config/bkup/env` and `~/.config/bkup/dirs`, installs the launchd plist
- `bin/bkup-status` — queries restic snapshots via JSON and reports age/staleness for this machine

## Configuration

Per-machine config in `~/.config/bkup/` (XDG-aware, respects `$XDG_CONFIG_HOME`):

- `env` — restic + B2 credentials (chmod 600). Variables: `RESTIC_REPOSITORY`, `RESTIC_PASSWORD`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- `dirs` — directories to back up, one per line, `#` comments and blank lines ignored

Override paths via `$BKUP_ENV_FILE` and `$BKUP_DIRS_FILE`.

## Scheduling

`bkup-init` writes `~/Library/LaunchAgents/com.sschlesier.bkup.plist` and loads it with `launchctl bootstrap`. Runs at 02:33 daily; uses `WakeSystem` to wake from sleep. Logs to `~/Library/Logs/bkup.log`.

## Shell conventions

All scripts use:
- `set -euo pipefail` strict mode
- `TRACE=1` enables `set -x` tracing
- `main()` function called at end of script
- restic discovered via `command -v` then fallback to common Homebrew prefixes (`/opt/homebrew`, `/usr/local`, `/home/linuxbrew/.linuxbrew`)
- Snapshots tagged with short hostname (`hostname | cut -d. -f1`) for multi-machine repos

## Running

```bash
bkup               # run backup (uses ~/.config/bkup/dirs)
bkup /path/to/dirs # run backup with explicit dirs file
bkup init          # interactive setup
bkup-status        # check backup freshness (warns if >2 days old)
bkup-status 7      # custom staleness threshold in days
TRACE=1 bkup       # debug with xtrace
```

## Distribution

Homebrew tap: `brew install sschlesier/tap/bkup`. No tests, no build step.

## Dependencies

- `restic` — backup engine
- `jq` — required by `bkup-status` for JSON parsing

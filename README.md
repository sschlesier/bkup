# bkup

> **Personal tool.** This is built for my own use and makes assumptions about
> my setup (Backblaze B2, restic, macOS). It is unlikely to suit anyone else
> without modification.

Nightly backup of selected directories to a restic repository on Backblaze B2,
scheduled via a launchd plist that wakes the machine from sleep.

## Scripts

| Script | Description |
|---|---|
| `bkup` | Run a restic backup using credentials and dirs from `~/.config/bkup/` |
| `bkup-init` | Interactive setup wizard — configures credentials, directories, and installs the launchd service |
| `bkup-status` | Show the status of recent snapshots |

## Installation

```
brew install sschlesier/tap/bkup
bkup init
```

## Configuration

Per-machine config lives in `~/.config/bkup/`:

- `env` — restic/B2 credentials (chmod 600, never committed)
- `dirs` — list of directories to back up, one per line

## Service

`bkup init` writes `~/Library/LaunchAgents/com.sschlesier.bkup.plist` and
loads it via `launchctl`. The job runs at 02:33 daily and uses `WakeSystem`
to wake the machine from sleep if needed.

## License

MIT

# Changelog

## v1.2.1 (2026-04-25)

- Add `BKUP_HOST` environment variable to override the hostname used when
  querying snapshots, enabling inspection of other machines' backups

## v1.2.0 (2026-04-04)

- Add `bkup logs` subcommand to list launchd log file paths (newest first)
- Split launchd stdout/stderr into separate log files

## v1.1.0 (2026-03-06)

- Add `-v`/`--verbose` flag to `bkup-status` to list all snapshots

## v1.0.1

- Use `WakeSystem` in launchd plist instead of brew services

## v1.0.0

- Initial release: `bkup`, `bkup-init`, `bkup-status`

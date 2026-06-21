# Changelog

All notable changes to the brain plugin are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). SemVer.

## [Unreleased]

## [0.2.0] — 2026-06-21

### Added
- `brain:log` skill — reflects on the current session and persists learnings;
  writes pending ADR stubs to `brain/decisions/index.md` and appends entries
  to `brain/memory.md` without overwriting existing content.

### Changed
- `brain:init` uses `README.md` instead of `SERVICE.md`; if `README.md` already
  exists the service block is appended as a `## Service` section rather than
  overwriting the file.
- `brain:init` no longer asks for team owner; `team:` field removed from the
  README.md template.
- `brain:adr` references updated from `SERVICE.md` to `README.md`.

## [0.1.0] — 2026-06-21

### Added
- `brain:init` skill — scaffolds CLAUDE.md, SERVICE.md, and full brain/ tree in a
  target repo; detects service name from settings.gradle.kts, build.gradle.kts,
  package.json, pom.xml, go.mod, Cargo.toml, or directory name (in that order).
- `brain:adr` skill — creates a new ADR file in brain/decisions/ and appends a
  one-line summary row to brain/decisions/index.md.
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` manifests.

### Changed
- `brain:init` no longer asks for team owner; removed `team:` field from SERVICE.md template.

# Changelog

All notable changes to the brain plugin are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). SemVer.

## [Unreleased]

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

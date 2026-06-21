# CLAUDE.md

Guidance for Claude Code when working in this repo.

## What this is

**brain** — a Claude Code plugin that gives every microservice repo a navigable,
persistent knowledge structure for AI agents. Each service gets a local `brain/`
directory: decisions, context, architecture notes, and a memory file that agents
write to during sessions. The design is local-first; a shared MCP federation layer
is planned for Phase 2. Read `PLAN.md` for the full picture.

## Layout

- `skills/init/SKILL.md` — `brain:init` — scaffolds the full brain structure in a
  target repo (CLAUDE.md, SERVICE.md, brain/ tree).
- `skills/adr/SKILL.md` — `brain:adr` — creates a new ADR and updates the index.
- `.claude-plugin/plugin.json` — plugin manifest (`name: brain`, version).
- `.claude-plugin/marketplace.json` — marketplace entry.
- `PLAN.md` — original design doc. Where it and the code disagree, the code wins.
- `CHANGELOG.md` — version history (Keep a Changelog format).

## Hard constraints (don't break these)

- **SKILL.md `name` field must be simple kebab-case** — `name: init`, not
  `name: Brain Init`. The `description` field carries the human label.
- **Skills are invoked as `brain:<skill-name>`** where `brain` is the plugin name
  from `plugin.json`. Keep the two in sync.
- **CLAUDE.md in target repos stays thin** — it routes to index files only, never
  lists individual brain documents. A new ADR updates `decisions/index.md`, not CLAUDE.md.
- **Do not overwrite existing files** during `brain:init` — skip silently and report.
- **Service name detection order** — check in this order:
  `settings.gradle.kts` → `build.gradle.kts` → `package.json` → `pom.xml` →
  `go.mod` → `Cargo.toml` → directory name.
- **Link syntax is `[[prefix:name]]`** — prefixes: `svc:`, `adr:`, `team:`,
  `domain:`. Do not invent new prefixes without updating PLAN.md.

## Release checklist (when cutting a version)

**Trigger:** a version bump in `git diff` means a release is being cut. Run this
before committing.

1. **Update `CHANGELOG.md`** — move `[Unreleased]` items under a new dated version
   section; start a fresh empty `[Unreleased]`. (Keep a Changelog format.)
2. **Bump version in both places** (they must match):
   `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (top-level
   `version` and `plugins[0].version`).
3. Use SemVer: bug fixes → patch, new skills or detection improvements → minor,
   breaking changes to the scaffolded file structure → major.

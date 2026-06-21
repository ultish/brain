---
name: sync-engram
description: Sync this repo's brain files into Engram as searchable memories. Use when asked to "sync brain to engram", "push brain to engram", "update engram", or "sync engram". Requires Engram MCP tools (mem_save, mem_update) to be available in the current session.
---

# Brain Sync to Engram

Push brain files from this repo into Engram as searchable memories. A manifest
(`brain/engram-sync.md`) tracks Engram IDs so re-syncing is safe — known entries
are updated, new entries are saved fresh.

Brain files are the source of truth. Engram is a searchable index over them.

## What gets synced

- Each ADR in `brain/decisions/` → type: `decision`
- `brain/context/overview.md` → type: `architecture`
- `brain/context/architecture.md` → type: `architecture`
- `brain/context/runbook.md` → type: `reference`

`brain/memory.md` is not synced — it's a session scratchpad. ADRs and context
files carry the durable knowledge worth indexing.

## Step 1 — Verify brain exists

Check that `brain/decisions/index.md` exists. If not, tell the user to run
`/brain:init` first and stop.

## Step 2 — Load manifest

Read `brain/engram-sync.md` if it exists. Parse the table to build a map of:
```
brain-file-path → { engram_id, last_synced }
```
If the file doesn't exist, start with an empty map — this is a first-time sync.

## Step 3 — Collect files to sync

Build a list of `(file_path, type, title)` tuples:

1. Read `brain/decisions/index.md` — find every linked ADR file in the table rows.
   For each ADR: `(brain/decisions/ADR-NNN-title.md, "decision", "ADR-NNN: <summary from index>")`

2. If `brain/context/overview.md` exists:
   `(brain/context/overview.md, "architecture", "overview")`

3. If `brain/context/architecture.md` exists:
   `(brain/context/architecture.md, "architecture", "architecture")`

4. If `brain/context/runbook.md` exists:
   `(brain/context/runbook.md, "reference", "runbook")`

Skip any file that doesn't exist on disk (unfilled scaffolding).

## Step 4 — Sync each file

For each `(file_path, type, title)`:

1. Read the file content.
2. Map content to Engram fields:
   - **For ADRs** — parse sections: `## Context` → why, `## Decision` → what,
     `## Consequences` → learned, `## Alternatives considered` → include in learned.
   - **For context files** — put full content in `what`. Leave why/learned empty.
3. Look up `file_path` in the manifest:
   - **Found** (engram_id exists): call `mem_update` with the engram_id and new content.
   - **Not found**: call `mem_save` with title, type, and mapped fields.
     Record the returned engram_id in the manifest.
4. Track outcome per file: `saved` | `updated` | `failed`.

## Step 5 — Write manifest

Write `brain/engram-sync.md` with the updated ID map and current timestamp.

```
---
last_sync: {ISO-8601 datetime}
---

| Brain File | Engram ID | Last Synced |
|-----------|-----------|-------------|
| brain/decisions/ADR-001-foo.md | obs_abc123 | {YYYY-MM-DD} |
| brain/context/overview.md | obs_def456 | {YYYY-MM-DD} |
```

Preserve existing rows for files that were not synced this run. Update only rows
that changed.

## Step 6 — Report

- X files synced (Y new, Z updated)
- List any failures with error details
- Remind the user to commit `brain/engram-sync.md` so the manifest travels with the repo

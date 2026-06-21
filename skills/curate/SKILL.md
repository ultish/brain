---
name: curate
description: Condense and clean up brain/memory.md when it has grown too large. Use when asked to "curate memory", "clean up the brain", "condense memory", or "prune the brain". Deduplicates entries, promotes complex ones to proper brain files, and removes stale content — always with user confirmation before writing.
---

# Brain Curate

Review `brain/memory.md`, identify what can be condensed or promoted, show the
proposed changes to the user, then apply on confirmation.

Never rewrite memory.md without explicit user approval.

## Step 1 — Check brain is initialised

Verify `brain/memory.md` exists. If not, tell the user to run `/brain:init` first and stop.

## Step 2 — Read config

Read `brain/config.md` for `memory_warn_threshold`. Default to **50** if absent.
Report the current entry count and threshold at the start so the user has context.

## Step 3 — Analyse memory.md

Read all entries in `brain/memory.md`. For each bullet, classify it as one of:

**Keep** — still accurate, not covered elsewhere, fits cleanly in its section.

**Deduplicate** — same fact already stated by another entry (exact or near-identical).
Mark the weaker/vaguer one for removal; keep the more precise one.

**Merge** — two or more entries are closely related and can be combined into a single
tighter bullet without losing meaning.

**Promote** — the entry has grown complex enough to deserve its own file:
- Belongs in `brain/context/` if it's architectural context or a pattern
- Belongs in `brain/decisions/` as a pending ADR stub if it's a decision

**Stale** — the entry refers to something that no longer exists (removed library,
changed pattern, resolved gotcha). Flag rather than silently delete — the user
confirms removal.

## Step 4 — Propose changes

Present a summary before touching anything:

```
brain/memory.md — N entries → M entries after curation

Removals ({count}):
  - "<entry text>" — duplicate of "<kept entry>"
  - "<entry text>" — stale (reason)

Merges ({count}):
  - "<entry A>" + "<entry B>" → "<merged text>"

Promotions ({count}):
  - "<entry text>" → brain/context/{filename}.md
  - "<entry text>" → brain/decisions/index.md (pending ADR stub)

Unchanged: {count} entries kept as-is.
```

Ask: **"Apply these changes? (yes / edit first / cancel)"**

- **yes** — proceed to Step 5
- **edit first** — show the full proposed memory.md so the user can adjust, then re-confirm
- **cancel** — stop without writing anything

## Step 5 — Apply

Rewrite `brain/memory.md` with the curated content.

For each promotion:
- If promoting to `brain/context/`, create the file with a `# {Title}` heading and
  the entry content as the opening paragraph. Add a row to `brain/context/index.md`.
- If promoting to decisions, append a `pending` stub row to `brain/decisions/index.md`
  (same format as `brain:log`).
- In `brain/memory.md`, replace the promoted entry with a one-line reference:
  `- → [[context:{filename}]]` or `- → [[adr:pending-{kebab-title}]]`

## Step 6 — Report

Tell the user:

- Final entry count vs starting count
- Files created (if any promotions)
- Suggest running `/brain:adr` if any pending ADR stubs were added

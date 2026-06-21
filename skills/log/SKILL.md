---
name: log
description: Reflect on the current session and persist what was learned. Use when asked to "log the session", "save what we learned", "record decisions from this session", or "update the brain". Writes pending ADR stubs and memory entries without requiring the user to remember what happened.
---

# Brain Log

Review the current conversation and extract anything worth keeping. Write pending
ADR stubs for architectural decisions and append learnings to `brain/memory.md`.

Do not overwrite existing content — only append.

## Step 1 — Check brain is initialised

Verify `brain/memory.md` and `brain/decisions/index.md` exist.
If not, tell the user to run `/brain:init` first and stop.

## Step 2 — Scan the session

Read back through the current conversation. Identify:

**ADR candidates** — things that qualify as an architectural decision:

- A technology, library, or framework was chosen (or rejected)
- A structural or design pattern was adopted
- A tradeoff was explicitly made and the reasoning discussed
- A constraint or rule was established ("we always...", "we never...")
- Something was decided that a future agent would need to know not to undo

**Memory candidates** — smaller learnings that don't need a full ADR:

- A gotcha, footgun, or non-obvious behaviour discovered
- A convention established for this codebase
- Context that explains _why_ something is the way it is
- Something the user explicitly said to remember

## Step 3 — Write pending ADR stubs

For each ADR candidate, append a row to the table in `brain/decisions/index.md`
with status `pending`:

```
| [[adr:pending-{kebab-title}]] | {one-sentence summary of the decision and why} | {YYYY-MM-DD} |
```

Use today's date. Keep the summary tight — enough for a future agent to understand
what was decided without reading the full ADR.

If `brain/decisions/index.md` already has a pending row that covers the same
decision, skip it rather than creating a duplicate.

## Step 4 — Append memory entries

Open `brain/memory.md`. Append memory candidates under the most relevant existing
section heading (`## Conventions`, `## Decisions`, `## Watch out for`).
If a candidate doesn't fit any existing section, append a new `## {Section}` heading.

Format each entry as a tight bullet:

```
- {What was learned} — {why it matters or what prompted it}
```

Do not add entries that are already captured (same fact, same section).

## Step 5 — Check memory size

After appending, count the total number of bullet entries in `brain/memory.md`.

Read the threshold from `brain/config.md` (`memory_warn_threshold` key).
If `brain/config.md` does not exist or the key is absent, use a default of **50**.

If the entry count exceeds the threshold, add a warning to the report:

```
⚠ brain/memory.md has N entries (threshold: X) — run `/brain:curate` to condense it.
```

## Step 6 — Report

Tell the user:

- How many pending ADR stubs were added (list their titles)
- How many memory entries were appended (list them)
- If nothing was found worth logging, say so explicitly rather than writing noise
- The memory size warning if the threshold was exceeded
- Remind them: run `/brain:adr` to formalise any pending stubs into full ADRs

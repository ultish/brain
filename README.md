# brain

A Claude Code plugin that gives every service repo a persistent, navigable
knowledge structure for AI agents.

Each repo gets a local `brain/` directory containing decisions, context,
architecture notes, and a memory file that agents read and write during sessions.
No infrastructure required — it's just Markdown files committed alongside the code.

## The problem

When an agent opens a repo it sees the current code and nothing else. It has
no idea why approach X was chosen over Y, which team owns the service, what
the inter-service communication contracts are, or what was already tried and
rejected. That context is lost between sessions and across engineers.

`brain` solves this by giving agents a structured place to find and record
institutional knowledge — and skills to keep it up to date automatically.

## Getting started

**1. Install the plugin**, then open a service repo and run:

```
/brain:init
```

Answer two questions (domain, status) and the full structure is scaffolded.
Fill in the `## Service` block in `README.md` with real dependencies and endpoints.
Commit everything — the brain travels with the code.

**2. Tell Claude about any decisions that already exist.** If the repo has history,
spend five minutes in a session describing the key choices that were made. Then run
`/brain:log` and Claude will write them up as pending ADR stubs. Run `/brain:adr`
to formalise each one.

That's the setup. From here the workflow is lightweight.

---

## Day-to-day workflow

### Starting a session

You don't need to do anything. Claude loads `CLAUDE.md` automatically, which routes
it to `brain/memory.md` and the index files. It arrives with context.

### During a session

Work normally. When you make a decision — choosing a library, settling on a pattern,
agreeing a tradeoff — just say it out loud in the conversation. You don't need to
stop and file anything. Claude is listening.

### Ending a session

Before closing, run:

```
/brain:log
```

Claude scans the conversation, extracts decisions and learnings, writes pending ADR
stubs to `brain/decisions/index.md`, and appends notes to `brain/memory.md`. Review
what it wrote, commit it.

### Formalising decisions

Pending ADR stubs are placeholders — one-liners in the index with a `pending-` prefix.
When you want a full record (context, reasoning, alternatives), run:

```
/brain:adr
```

Claude will see the pending stubs and offer to work through them. A full ADR takes
about two minutes per decision.

You don't need to do this immediately. Stubs are useful on their own — they tell a
future agent "a decision was made here, ask about it." Formalise when the decision
is significant enough to warrant the detail.

### When memory gets large

`/brain:log` warns you when `brain/memory.md` exceeds the configured threshold
(default 50 entries). When that happens, run:

```
/brain:curate
```

Claude proposes what to deduplicate, merge, or promote to a proper context file.
You confirm before anything is written. Takes a few minutes, keeps the file useful.

---

## Installation

Install via the Claude Code marketplace or clone and point Claude Code at this
directory as a local plugin.

## Commands

| Command | What it does |
|---------|-------------|
| `/brain:init` | Scaffolds the full brain structure in the current repo |
| `/brain:adr` | Creates a new Architecture Decision Record |
| `/brain:log` | Reflects on the current session and persists learnings |
| `/brain:curate` | Condenses and prunes `brain/memory.md` when it grows large |

## What `brain:init` creates

```
your-service/
├── CLAUDE.md                  # Auto-loaded; routes agents to brain files
├── README.md                  # Service identity (## Service section appended)
└── brain/
    ├── config.md              # Plugin config (e.g. memory_warn_threshold)
    ├── memory.md              # Agents append learnings here during sessions
    ├── decisions/
    │   └── index.md           # One-line summary per ADR; start here
    └── context/
        ├── index.md           # One-line summary per context file
        ├── overview.md        # Purpose, responsibilities, boundaries
        ├── architecture.md    # Components, data flow, storage
        └── runbook.md         # Deployment, config, common issues
```

If `README.md` already exists, `brain:init` appends a `## Service` section
rather than overwriting it.

## The README.md service block

```
## Service

status: stable
domain: [[domain:payments]]

depends_on: []
  # - [[svc:other-service]]

exposes: []
  # - POST /path (REST)

events:
  publishes: []
  subscribes: []

see_also: []
  # - [[adr:ADR-001]]
```

## ADRs

`/brain:adr` creates a numbered ADR file in `brain/decisions/` and appends
a one-line summary row to the index. The index is what agents read first —
each row has enough context to decide whether to open the full file.

Pending stubs (created by `/brain:log`) appear in the index with a
`pending-` prefix. Run `/brain:adr` to formalise them into full ADRs.

## Session logging

`/brain:log` scans the current conversation and extracts:

- **ADR candidates** — technology choices, tradeoffs, established rules →
  written as pending stubs in `brain/decisions/index.md`
- **Memory candidates** — gotchas, conventions, context → appended to
  `brain/memory.md`

Run it at the end of any session where decisions were made.

## Memory curation

`brain/memory.md` grows over time. `/brain:curate` reviews the file and
proposes deduplication, merges, promotions to proper `brain/context/` files,
and removal of stale entries — always with your confirmation before writing.

The warning threshold is configurable in `brain/config.md`:

```
memory_warn_threshold: 50
```

`/brain:log` warns when the entry count exceeds this value.

## Link syntax

Links use `[[prefix:name]]` syntax. They're currently unresolved references —
they become meaningful once the shared MCP federation layer exists (Phase 2).
In the meantime they work as Obsidian wikilinks for free graph visualisation.

| Prefix | Points to |
|--------|-----------|
| `[[svc:name]]` | That service's README.md |
| `[[adr:ADR-NNN-title]]` | `brain/decisions/ADR-NNN-title.md` |
| `[[team:name]]` | Team page in shared brain (Phase 2) |
| `[[domain:name]]` | Domain page in shared brain (Phase 2) |

## Routing pattern

Agents navigate the brain in order: `CLAUDE.md` → relevant `index.md` →
specific file. They never load everything at once. As the brain grows, add
sub-indexes inside subdirectories rather than flattening into `CLAUDE.md`.

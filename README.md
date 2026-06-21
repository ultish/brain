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

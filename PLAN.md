# Brain Plugin — Build Plan

## Problem Being Solved

20-person team building 100s of microservices (many not yet implemented).
When a developer opens an LLM agent in a repo, the agent only sees current code
and a CLAUDE.md. It has zero institutional memory:

- Why was approach X chosen over Y?
- What did we decide about inter-service communication?
- Which team owns which service?
- What's the status of ServiceA — is it safe to depend on?

Design decisions, architectural rationale, and team conventions are lost between
sessions and across engineers. The brain plugin solves this by giving agents a
navigable, persistent knowledge structure inside each repo.

---

## Key Design Decisions

**1. Local file structure first, shared MCP layer later**
Start with markdown files inside each repo. No infrastructure needed.
The MCP federation layer (for cross-repo queries) comes in a later phase.

**2. No personal/staging folders**
Kept it simple. All brain files are committed and reviewed via normal merge requests.
MRs are the curation mechanism.

**3. Obsidian-style `[[links]]` with typed prefixes**
Links use `[[svc:service-name]]`, `[[adr:ADR-NNN]]`, `[[team:team-name]]`, `[[domain:domain-name]]`.
Currently unresolved — they become meaningful when the MCP server exists.
They also allow opening the brain repo in Obsidian for a free visual graph.

**4. CLAUDE.md stays tiny — points to indexes only**
CLAUDE.md never lists individual files. It routes to index.md files.
A new ADR updates decisions/index.md, never CLAUDE.md.

**5. Routing via index files**
Each folder has an index.md with a one-line summary per file.
Agent reads: CLAUDE.md → index.md → specific file.
Never loads everything. Scales as brain grows.
If a folder grows large, add sub-indexes recursively.

**6. SERVICE.md is the authoritative service identity**
Lives in the repo root alongside code. Updated in the same PR as code changes.
Cannot go stale because it travels with the implementation.

**7. memory.md inside brain/**
Claude Code's auto-memory writes to brain/memory.md (configured via CLAUDE.md).
Committed and reviewed via MR like any other brain file.

**8. Levels 3 (semantic search) and 4 (knowledge graph) deferred**
Semantic search (Qdrant) adds value only once there are 100+ brain documents.
Knowledge graph (LightRAG) is over-engineered for daily developer use.
Revisit when there is felt pain at the current level.

---

## Scaffolded Structure (what `brain:init` creates)

```
service-name/
  CLAUDE.md                  ← always loaded, routes to indexes
  SERVICE.md                 ← authoritative service identity
  brain/
    memory.md                ← Claude writes facts here during sessions
    decisions/
      index.md               ← one-line summary per ADR
      (ADR files added via brain:adr)
    context/
      index.md               ← one-line summary per context file
      overview.md            ← purpose, responsibilities, boundaries
      architecture.md        ← components, data flow, storage
      runbook.md             ← deployment, config, common issues
```

### CLAUDE.md routing pattern

```
## This service
Read SERVICE.md first.

## Memory
Memory file: brain/memory.md. Append facts learned during session.
Promote complex entries to brain/decisions/ or brain/context/.

## Finding information
- Memory: brain/memory.md
- Decisions: brain/decisions/index.md → drill down
- Context: brain/context/index.md → drill down
- Ops: brain/context/runbook.md
```

### SERVICE.md structure

```
# service-name

status: planned|in-progress|stable|deprecated
team: [[team:team-name]]
domain: [[domain:domain-name]]

depends_on:
  - [[svc:other-service]]

exposes:
  - POST /path (gRPC port 8080)

events:
  publishes: [event.name]
  subscribes: [other.event]

see_also:
  - [[adr:ADR-001-title]]

## Summary
One paragraph — what this service does and why it exists.
```

### Link conventions

| Prefix | Resolves to |
|--------|-------------|
| `[[svc:name]]` | SERVICE.md in that service's repo |
| `[[adr:ADR-NNN-title]]` | brain/decisions/ADR-NNN-title.md |
| `[[team:name]]` | team page in shared brain (future) |
| `[[domain:name]]` | domain page in shared brain (future) |

---

## Plugin Conventions (important)

- SKILL.md `name` field must be simple kebab-case matching the invocation name.
  `name: init` not `name: Brain Init`. The description field carries the human label.
- Skills are invoked as `brain:skill-name` where `brain` is the plugin name from plugin.json.

## What Is Built

**Plugin location**: `/Users/jxhui/Developer/brains/`

```
.claude-plugin/
  plugin.json              ← name: "brain", version: 0.1.0
skills/
  init/
    SKILL.md               ← brain:init — full scaffold
  adr/
    SKILL.md               ← brain:adr — create ADR + update index
```

### brain:init
- Detects service name from package.json / go.mod / Cargo.toml / directory name
- Asks: team name, domain, status
- Creates all files in the scaffolded structure above
- Skips files that already exist
- Confirms what was created

### brain:adr
- Checks brain is initialised
- Determines next ADR number from index
- Asks: title, context, decision, consequences, alternatives considered
- Creates `brain/decisions/ADR-NNN-kebab-title.md`
- Appends one-line row to `brain/decisions/index.md`
- Suggests linking from SERVICE.md

---

## What Is Not Built Yet

### Next skills to build

**`brain:update-service`**
Update SERVICE.md when the service changes — new dependencies, new endpoints,
changed events. Should diff what's there and ask about specific fields rather
than replacing the whole file.

**`brain:context`**
Add or update a context file. Handles two cases:
1. Updating an existing file (overview, architecture, runbook)
2. Creating a new context file and adding it to context/index.md

**`brain:status`**
Health check on the brain. Reports:
- Which template sections are still empty placeholders
- How many ADRs exist
- When memory.md was last updated (via git log)
- Any broken [[links]] (references to files that don't exist locally)

### Future: Engram Integration — Multi-Repo Sync

`brain:sync-engram` (built) handles per-repo sync. The cross-repo problem requires
a separate sync driver that does not belong inside a per-repo Claude Code skill.

**Design: GitLab-backed shallow-clone sync**

A config file (e.g. `~/.config/brain/sync.yaml`) lists GitLab groups or individual
repos to sync:

```yaml
gitlab_url: https://gitlab.example.com
token_env: GITLAB_TOKEN        # PAT with read_repository scope
repos:
  - group: platform             # sync entire GitLab group
  - project: infra/auth-service # or individual projects
clone_root: /tmp/brain-sync    # shallow clones land here
```

A sync script (or scheduled CI job) runs:
1. For each configured repo, `git clone --depth 1 <url> <clone_root>/<service>`
2. In each cloned dir, invoke the Engram MCP directly (or via `engram save` CLI)
   to push brain files — same logic as `brain:sync-engram` but driven externally.
3. Delete the clone when done (shallow — disk cost is minimal).

The sync script runs outside Claude Code (a plain shell script, a Go CLI, or a
GitLab scheduled pipeline). Claude Code is not the right host for cross-repo
operations that iterate over 100s of repos.

**Idempotency at scale**

The per-repo `brain/engram-sync.md` manifest is committed in each repo. The sync
script reads it (via the clone) to know which entries to update vs. create, then
writes the updated manifest back as a commit. This keeps the manifest authoritative
even when synced from CI rather than from a developer's machine.

**Trigger options**
- Scheduled GitLab pipeline (nightly) — no developer action needed
- Webhook on push to main — syncs immediately when brain files change
- Manual: `brain-sync --all` from a developer machine

This is Phase 1.5 — no new infrastructure needed, just a script and a PAT.
Build this before investing in the MCP federation server below.

---

### Future: Phase 2 — Shared MCP Server

When local brain is insufficient for cross-service queries, add:

**MCP server (kube-deployed)**
- Read-only federation layer — reads SERVICE.md from each repo directly
- Does NOT duplicate content — services are the source of truth
- Exposes tools:
  - `brain_read(service-name)` — reads SERVICE.md from that repo
  - `brain_backlinks(service-name)` — finds all SERVICE.md files that [[link]] to it
  - `brain_search(query)` — semantic search over brain MD files (requires Qdrant)
  - `brain_list(domain?, status?)` — list services matching filters
- Needs git credentials / GitHub API token to read across repos
- Shared brain repo for team-wide content (ADRs affecting multiple services,
  team definitions, domain definitions, conventions)

**Qdrant (alongside MCP server in kube)**
- Semantic search over brain MD files
- Embed per document (not per chunk) for ADRs and short context files
- Embed per section for longer files
- Add when brain has 100+ documents and routing alone breaks down

**CLAUDE.md addition for phase 2**
```
## Cross-service knowledge
For other services: call brain_read("service-name") via MCP.
For impact analysis: call brain_backlinks("service-name") via MCP.
MCP server: [kube endpoint]
```

---

## Routing Scaling Rules

As the brain grows, apply these rules:

| Condition | Action |
|-----------|--------|
| decisions/ has 20+ ADRs | Group into subdirectories by domain, add sub-index |
| context/ file exceeds ~200 lines | Split by topic, update index |
| CLAUDE.md is being edited for new files | Stop — fix the index instead |
| Agent says "I couldn't find..." | Add a routing hint to the relevant index.md |

---

## Non-Goals (explicitly out of scope)

- LightRAG / knowledge graph extraction — too complex for daily dev use
- Personal scratchpad (personal/ folder) — MRs handle curation
- Staging area — same reason
- Always-on autonomous memory (GBrain style) — requires agent harness infrastructure
- Obsidian as a required tool — it's optional visualisation only

---
name: init
description: Initialize a structured project brain in the current repository. Use when asked to "init brain", "set up brain", "scaffold brain", or "create brain" for a project or service.
---

# Brain Init

Scaffold a standardized brain structure in the current repository. This creates
the files and folders that allow AI agents to navigate institutional knowledge
for this service without re-explanation every session.

Do not overwrite any file that already exists.

## Step 1 — Detect service name

Check in order:

1. `settings.gradle.kts` → `rootProject.name = "..."` line
2. `build.gradle.kts` → `rootProject.name = "..."` line (if no `settings.gradle.kts`)
3. `package.json` → `name` field
4. `pom.xml` → `<artifactId>` element
5. `go.mod` → first `module` line (last path segment)
6. `Cargo.toml` → `name` field under `[package]`
7. Current directory name as fallback

## Step 2 — Gather info from the user

Ask in a single message:

- **Domain** — which domain does it belong to? (e.g. `payments`)
- **Status** — `planned` | `in-progress` | `stable` | `deprecated`

## Step 3 — Create files

Create each file below using the exact templates provided. Replace `{service-name}`,
`{domain}`, and `{status}` with the values gathered above.
Use today's date where `{YYYY-MM-DD}` appears.

---

### CLAUDE.md

```
# {service-name}

This file loads automatically when Claude Code opens this project.
Read it first, then navigate to relevant brain files for context.

## This service
Read README.md (the ## Service section) for service identity, dependencies, and contracts.

## Memory
Your memory file for this project is brain/memory.md.
When you learn facts, decisions, or conventions during a session,
append brief entries here. If an entry grows complex, promote it
to a proper file in brain/decisions/ or brain/context/ and leave
a [[link]] in memory.md pointing to the new file.

## Finding information
- Memory and recent learnings: brain/memory.md
- Decisions and ADRs: start at brain/decisions/index.md, then drill down
- Context and architecture: start at brain/context/index.md, then drill down
- Operational knowledge: brain/context/runbook.md

## Cross-service references
Links to other services use [[svc:service-name]] syntax.
These resolve once a shared brain MCP server is available.
For now, check the referenced service's SERVICE.md in its own repo.
```

---

### README.md

**If `README.md` does not exist** — create it with the full template:

```
# {service-name}

status: {status}
domain: [[domain:{domain}]]

depends_on: []
  # - [[svc:service-name]]

exposes: []
  # - METHOD /path (protocol)

events:
  publishes: []
  subscribes: []

see_also: []
  # - [[adr:ADR-001]]

## Summary
[One paragraph describing what this service does and why it exists]
```

**If `README.md` already exists** — append the following block at the end of the file:

```

## Service

status: {status}
domain: [[domain:{domain}]]

depends_on: []
  # - [[svc:service-name]]

exposes: []
  # - METHOD /path (protocol)

events:
  publishes: []
  subscribes: []

see_also: []
  # - [[adr:ADR-001]]
```

---

### brain/memory.md

```
# Memory

## Conventions

## Decisions

## Watch out for
```

---

### brain/decisions/index.md

```
# Decisions Index

Start here when looking for decisions, ADRs, or architectural choices for this service.
Each entry has enough context to decide whether to read the full file.

| File | Summary | Date |
|------|---------|------|

_No decisions recorded yet. Use `/brain:adr` to create the first one._
```

---

### brain/context/index.md

```
# Context Index

Start here when looking for context, architecture, or operational knowledge.

| File | Summary |
|------|---------|
| [overview](overview.md) | What this service does and why it exists |
| [architecture](architecture.md) | Internal structure and key components |
| [runbook](runbook.md) | Operational knowledge and procedures |
```

---

### brain/context/overview.md

```
# Overview

## Purpose
[Why this service exists — the problem it solves]

## Responsibilities
[What this service is responsible for]

## Not responsible for
[Explicit boundaries — what this service deliberately does NOT do]

## Key stakeholders
[Who depends on this service, who owns it]
```

---

### brain/context/architecture.md

```
# Architecture

## Components
[Key internal components and their roles]

## Data flow
[How data enters, moves through, and exits this service]

## Storage
[Databases, caches, and other persistence used — and why]

## Key design decisions
[Notable architectural choices — link to relevant ADRs using [[adr:ADR-NNN]]]
```

---

### brain/context/runbook.md

```
# Runbook

## Deployment
[How to deploy this service]

## Configuration
[Key environment variables and what they control]

## Common issues
[Known issues, gotchas, and their solutions]

## Monitoring
[What metrics and alerts to watch]

## On-call
[What to do when things go wrong, who to contact]
```

---

## Step 4 — Confirm

Report:

- Which files were created
- Which files were skipped (already existed)
- Remind the user to:
  - Fill in `README.md` with actual dependencies, endpoints, and events
  - Add context in `brain/context/overview.md`
  - Run `/brain:adr` to record the first architectural decision

---
name: adr
description: Create a new Architecture Decision Record (ADR) in this project's brain. Use when asked to "record a decision", "create an ADR", "document a decision", or "add a decision" to the brain.
---

# Brain ADR

Create a new Architecture Decision Record in `brain/decisions/` and update the index.

## Step 1 — Check brain is initialised

Verify `brain/decisions/index.md` exists. If not, tell the user to run `/brain:init` first.

## Step 2 — Determine next ADR number

Read `brain/decisions/index.md` and find the highest existing ADR number.
Next number is highest + 1, zero-padded to 3 digits (e.g. `001`, `012`, `043`).
If no ADRs exist yet, start at `001`.

## Step 3 — Gather info from the user

Ask in a single message:

- **Title** — brief phrase describing what was decided (e.g. "Use gRPC for all sync inter-service calls")
- **Context** — what problem or situation prompted this decision?
- **Decision** — what exactly was decided? Be specific.
- **Consequences** — what changes as a result? Include tradeoffs.
- **Alternatives considered** — what else was evaluated and why was it rejected? (optional, skip if none)

## Step 4 — Derive filename

Convert the title to kebab-case:

- Lowercase
- Replace spaces and special characters with hyphens
- Remove articles (a, an, the)

Example: "Use gRPC for all sync inter-service calls" → `use-grpc-for-sync-inter-service-calls`

Full filename: `brain/decisions/ADR-{NNN}-{kebab-title}.md`

## Step 5 — Create the ADR file

Use today's date for `{YYYY-MM-DD}`. Read team from `README.md` (`## Service` section) if available.

```
# ADR-{NNN}: {Title}

date: {YYYY-MM-DD}
status: accepted
deciders: [[team:{team-name}]]

## Context
{Context from user}

## Decision
{Decision from user}

## Consequences
{Consequences from user}

## Alternatives considered
{Alternatives from user, or "None documented." if skipped}

## See also
```

Leave `## See also` empty — the user can fill in links to related ADRs later.

## Step 6 — Update brain/decisions/index.md

Append a new row to the table. Summary should be one tight sentence — enough
for an agent to decide whether to read the full file without opening it.

```
| [[adr:ADR-{NNN}-{kebab-title}]] | {one-sentence summary} | {YYYY-MM-DD} |
```

## Step 7 — Confirm

Report the ADR number, filename, and that the index was updated.
Suggest linking to this ADR from `README.md` (the `see_also:` list in the `## Service` section)
using `[[adr:ADR-{NNN}-{kebab-title}]]` if the decision affects this service's contracts or dependencies.

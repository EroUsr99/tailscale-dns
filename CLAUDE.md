# CLAUDE.md — tailscale-dns

## Project

Lightweight DNS server that makes `*.erodev.cloud` domains resolve to Tailscale IP addresses instead of public IPs. This enables secure access to services over Tailscale without exposing them to the public internet.

## Session-start read order

Read these in this order at every session start. Stop as soon as you have enough context to act:

1. **STATE.md** (project root) — current position. If this file is empty or missing, there are no locked decisions yet. If the timestamp is older than the last known turning point, the file is stale — flag before trusting.
2. **context/INDEX.md** — index of reference documents in `context/extracted/`. Consult before asserting facts about the domain.
3. **docs/** — living project documentation (briefs, decisions, architecture notes, meeting outputs).
4. **git log** — last 5 commits for recent work context.

## Citation rule for `context/extracted/`

When you assert a fact that comes from a reference document in `context/extracted/`, cite the source filename. Facts drift silently otherwise — the source gets overwritten or superseded while the assertion stays frozen. Example: `(source: context/extracted/town-hall-2025-11-19.md)`.

This rule exists because `context/extracted/` files are external reality, not our own writing. When reality moves, we need traceability to know what was sourced where.

## Where work lives

- **Active work** → GitHub issues on AI Platform (use `create-issue:` skill)
- **Parked ideas** → `~/shared-context/BACKLOG.md` (use `idea:` skill)
- **Current position** → `STATE.md` (use `save-state:` skill)
- **Session state** → `HANDOVER.md` (auto-written by SessionEnd hook — do not edit)

## Working conventions

_(Project-specific working conventions go here. Delete this placeholder when you add real content.)_

## Cross-project impact

If a decision here affects other projects, log it to `~/shared-context/DECISIONS.md`. Format: `- [YYYY-MM-DD] Decision text (context: why)`.

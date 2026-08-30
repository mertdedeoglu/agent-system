# Agent System — Orchestration Map

This project uses an 8-agent Claude Code subagent system, split by backend/frontend where the two stacks genuinely differ (C#/.NET backend, JS/TS frontend). This file documents how they hand off to each other so both humans and Claude can reason about the chain.

## The chain

```
                         Planner
                            │
              ┌─────────────┴─────────────┐
              ▼ (backend work)             ▼ (frontend work)
      backend-implementer         frontend-implementer      ← run in parallel for full-stack tasks
              │                             │
              ▼                             ▼
      backend-reviewer  ──fail──▶ back to backend-implementer
      frontend-reviewer ──fail──▶ back to frontend-implementer
              │ both pass (or the one relevant side passes)
              ▼
            Tester  ──fail──▶ Debugger ──▶ backend- and/or frontend-implementer (whichever owns the bug)
              │ pass
              ▼
         Docs Writer (if user-facing or public-API/contract change)
```

For a **backend-only** or **frontend-only** task, only that side's implementer/reviewer runs — Planner decides this and says so explicitly in its hand-off.

## Contextual (non-linear) agent

**Security Auditor** — not part of the fixed sequence. Fires automatically whenever a diff (backend or frontend) touches:
- authentication / authorization code
- user input handling / parsing / deserialization
- secrets, credentials, connection strings, API keys
- cryptography or hashing

It reports; it never patches, on either side.

## Read-only agents

backend-reviewer, frontend-reviewer, security-auditor, and debugger are intentionally **read-only** (no Write/Edit, and security-auditor has no Bash either). They diagnose and report; only backend-implementer or frontend-implementer changes code, each within their own domain. This keeps the "independent check" property meaningful.

## Full-stack coordination

For changes spanning both sides:
- Planner calls out the API contract (endpoint shape, request/response DTOs) explicitly in the plan so frontend-implementer can build against it without waiting for backend-implementer to fully finish.
- backend-reviewer and frontend-reviewer each check contract stability from their side — a backend change that silently breaks the agreed contract, or a frontend change that assumes a shape the backend doesn't provide, should get flagged.
- Tester waits for **both** reviewers to pass before running, and includes at least one integration-level check that the two sides actually agree, not just isolated unit tests per side.

## How the hand-off actually works

Claude Code doesn't enforce this pipeline mechanically — there's no built-in "state machine" runtime for subagents. The chain works because each agent's own instructions end with an explicit hand-off directive, and each agent's `description` field uses directive phrasing ("use PROACTIVELY after X finishes") so the orchestrating Claude reliably picks it up. If a hand-off seems to be getting skipped in practice, the fix is almost always to make that agent's description more explicit/pushy, not to add framework config.

## Manual override

Any agent can be invoked directly by name (`@backend-implementer`, `@frontend-reviewer`, etc.) if the user wants to skip the automatic chain for a one-off task.

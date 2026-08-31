---
name: create-agent-system
description: Sets up a full multi-agent development system (Claude Code subagents) inside a software project — a Planner/Architect, separate Backend and Frontend Implementers, separate Backend and Frontend Reviewers, a Tester, Debugger, Security Auditor, and Docs Writer, wired together into an automatic hand-off chain. Use this skill whenever the user asks to "set up an agent system", "create agent system", "add subagents", "build a multi-agent workflow", wants specialized agents for planning/coding/reviewing/testing in their repo, or mentions organizing their coding workflow into separate roles/agents — even if they don't name the exact agents. For an existing project, it analyzes the actual codebase (not just file extensions) to pick real conventions; for a new/empty project it asks the user for backend/frontend stack first rather than silently defaulting to C#. Adapts to whatever frontend framework (React/Angular/Vue) is found or chosen; if there's no separate frontend stack, collapses to a single implementer/reviewer pair instead. Every Implementer/Reviewer agent it installs automatically checks for and defers to `.claude/rules/clean-code-standards.md` (installed by the `install-clean-code-rule` skill) whenever that file is present in the project, regardless of install order.
---

# Create Agent System

Installs an 8-agent Claude Code subagent system (`.claude/agents/`) into the user's project, with an automatic Planner → {Backend/Frontend Implementer} → {Backend/Frontend Reviewer} → Tester hand-off chain, plus contextually-triggered Debugger, Security Auditor, and Docs Writer agents.

## What this skill produces

For the target project, this skill creates:

```
<project-root>/.claude/agents/
├── README.md               — the orchestration map (chain logic, when each agent fires)
├── planner.md               — breaks the request into a plan, routes to the right implementer(s)
├── backend-implementer.md   — writes/edits backend (C#/.NET) code, hands off to backend-reviewer
├── frontend-implementer.md  — writes/edits frontend (React/Angular/Vue) code, hands off to frontend-reviewer
├── backend-reviewer.md      — read-only backend code review
├── frontend-reviewer.md     — read-only frontend code review
├── tester.md                 — writes/runs tests on both sides, triggers Debugger on failure
├── debugger.md               — read-only root-cause analysis, routes fix to the owning implementer
├── security-auditor.md       — read-only security pass, triggered on auth/input/secret-touching changes (either side)
└── docs-writer.md            — updates docs/README after Tester passes
```

Each agent is a standard Claude Code subagent file (YAML frontmatter + system prompt) invoked via Claude Code's Task-based subagent mechanism.

**When to skip the backend/frontend split**: if the project has no separate frontend stack (backend-only API, or a same-ecosystem UI like Blazor), install only `backend-implementer.md` and `backend-reviewer.md` (renamed to `implementer.md`/`reviewer.md`, dropping the "backend-" prefix since there's no split to distinguish), and skip the frontend files entirely. Ask the user in Step 1 if the frontend situation is unclear.

## Step 1 — Detect the project

Before installing anything:

1. Confirm you're inside a project (not just a scratch folder). If ambiguous, ask the user for the project root.
2. **Determine existing vs. new project first — this branches everything below.** Check for real source files (`.cs`, `.ts`/`.tsx`, `.py`, `.go`, `.java`, etc.), not just scaffolding like an empty repo with only `.git`/`README.md`.
   - **Existing project → analyze it, don't pattern-match on file extensions alone.** Read a representative sample of actual source files (not just check for `*.csproj`/`package.json`) to confirm the real backend language/framework, layering style, naming conventions, and test setup already in use. This analysis — not a generic default — is what Step 2's template customization is based on. The "defaults to C# on the backend" behavior only applies as the template's fallback baseline, never as a silent override of what a real project actually uses.
   - **New/empty project → do not assume a stack.** Ask the user directly, in one short question (not an interview): backend language/framework, frontend framework if there is one (or "no separate frontend"), and the test command if already decided. Proceed with their answers instead of guessing.
3. Detect whether there's a **separate frontend stack** (only needed for the existing-project path — for a new project this was already answered in step 2): look for a `package.json` with React/Angular/Vue in dependencies, typically in its own folder (`/frontend`, `/client`, `/web`, or a separate repo root). Note which framework, from `package.json` dependencies (`react`, `@angular/core`, `vue`).
   - Separate JS/TS frontend found → install the full backend/frontend split (8 agents).
   - No separate frontend, or frontend is same-ecosystem (e.g. Blazor) → collapse to a single `implementer.md`/`reviewer.md` pair (see note above) instead of the backend-/frontend- split (7 agents total).
   - Still unclear after reading the code → ask the user rather than guessing.
4. Check whether `.claude/agents/` already exists and has files. If so, **list what's there and ask the user** whether to overwrite, merge, or rename before writing anything — never silently clobber existing agent definitions.
5. No detection needed for clean code standards: every implementer/reviewer template already contains an unconditional "check `.claude/rules/clean-code-standards.md` first, fall back to the inline baseline otherwise" instruction. It works whether `install-clean-code-rule` was run before or after this skill, or never.
6. Same for the LLM wiki: `docs-writer.md` already contains an unconditional check for `.claude/wiki/` (set up by `llm-wiki-setup`) and writes an otherwise-undocumented feature there as a fallback, never into `.claude/raw/`. Works whether `llm-wiki-setup` ran before or after this skill, or never.

## Step 2 — Customize the templates

The `assets/agents/` folder in this skill contains the agent templates plus `assets/agents/README.md` (the orchestration map). Copy the relevant set into `<project-root>/.claude/agents/`, then edit the copies:

- Fill in `{{PROJECT_NAME}}` everywhere it appears.
- In `backend-implementer.md` / `backend-reviewer.md`: the C# conventions section is already filled in (nullable reference types, async/await discipline, `IDisposable`/`using`, LINQ readability, DI over `static` state, records for DTOs, specific exception types). If the backend isn't actually C#, replace that section with the idiomatic conventions for the detected backend language instead — do a quick web search if unsure of current idioms for a less-common stack.
- In `frontend-implementer.md` / `frontend-reviewer.md`: replace `{{FRONTEND_FRAMEWORK}}` with the detected framework (React/Angular/Vue), and trim the framework-specific bullet lists in each template down to the one that applies (delete the other two frameworks' bullets so the agent isn't carrying irrelevant guidance).
- Fill in `{{BACKEND_TEST_COMMAND}}` and `{{FRONTEND_TEST_COMMAND}}` in `tester.md` (detect from the repo: `dotnet test`, `npm test`/`npm run test`, etc.). If there's no frontend split, just fill in the single relevant command and remove the other line.
- If collapsing to the no-frontend-split setup (per Step 1), rename `backend-implementer.md` → `implementer.md` and `backend-reviewer.md` → `reviewer.md`, drop the "backend-" framing from their descriptions/prompts, and skip `frontend-implementer.md`/`frontend-reviewer.md` entirely. Update `README.md`'s diagram accordingly (single implementer/reviewer, no parallel branch).

## Step 3 — Write the files and report back

Write the customized files to `.claude/agents/`. Then tell the user, concisely:
- Which agents were installed and where (8 with the backend/frontend split, 7 without)
- What backend and frontend conventions were applied
- That implementer/reviewer agents will auto-follow `.claude/rules/clean-code-standards.md` if it's present in the project (no linking step needed)
- The one-line version of the chain: **Planner → {Backend/Frontend Implementer, in parallel for full-stack tasks} → {matching Reviewer} → (pass: Tester / fail: back to that Implementer) → Tester → (pass: Docs Writer / fail: Debugger → owning Implementer)**, with Security Auditor firing automatically whenever a diff on either side touches auth, input validation, secrets, or crypto code.

Do not run a build/test pass yourself as part of installation — this skill only installs the agent definitions. Offer to do a dry-run test (give the user a small sample task and show which agent would pick it up) if they want to validate the setup before relying on it.

## Step 4 — Offer to test and package

Following this project's existing workflow, offer to:
1. Walk through 1-2 representative test scenarios (new feature request, bug report) and narrate which agent(s) would fire and why, so the user can sanity-check the chain before trusting it on real work.
2. If the user wants this bundled as a distributable `.skill` file for their own profile (as opposed to just files dropped into this one project), use `skill-creator`'s `scripts/package_skill.py` on the `create-agent-system` skill folder itself — not on the generated `.claude/agents/` output, which lives in the target project, not as a skill.

## Design notes (why it's built this way)

- **Automatic chaining is description-driven, not framework-enforced.** Claude Code's main thread decides whether to invoke a subagent based on that subagent's `description` field and explicit hand-off instructions in the calling agent's own prompt. That's why every template's system prompt ends with an explicit "when you finish, invoke X" instruction, and why descriptions use directive language like "use PROACTIVELY after Implementer finishes" — this is the standard pattern for getting reliable automatic hand-offs in Claude Code.
- **Read-only agents (Reviewer, Security Auditor, Debugger) never get Write/Edit/Bash-that-mutates.** This is deliberate: an agent that can both find a problem and silently fix it removes the independent-check property that makes review meaningful. They report; Implementer fixes.
- **Security Auditor is not part of the linear chain.** It's triggered contextually (auth/input/secret/crypto changes) rather than on every single diff, to avoid it becoming noise on unrelated changes.

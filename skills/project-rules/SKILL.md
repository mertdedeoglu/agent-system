---
name: project-rules
description: Sets up and maintains a living, project-specific rules file (project-rules.md) capturing corrections the user or a reviewer agent gives about code quality, style, or architecture, then applies those rules on every future code task. Use when the user wants to set up / initialize project rules, or wants existing dev/review agents wired to learn from corrections automatically. Also trigger on first code task in a project with no project-rules.md yet (offer to scan/bootstrap it), on any correction to generated code ("böyle yazma / bir daha bunu yapma / don't do X, do Y instead"), and whenever a reviewer/QA agent flags a quality or convention issue. Distinct from clean-code-standards (universal/global); this is always project-local, and can patch existing subagent definitions (e.g. .claude/agents/*.md) so they read/write project-rules.md themselves.
---

# Project Rules

A self-improving, project-scoped memory of code conventions. This skill has two jobs that run in a loop:

1. **Apply**: before writing/editing/reviewing code in a project, load that project's rules and follow them.
2. **Learn**: whenever the user or a reviewer agent gives corrective feedback about code, distill it into a durable rule and save it.

This skill is intentionally project-local — never write rules to a global/user-level file. Each project keeps its own `project-rules.md`.

## Where rules live

At the root of the current project: `project-rules.md`

If it doesn't exist yet, it will be created the first time a rule is captured (see "Bootstrapping" below for the optional initial-scan case).

## File format

Plain markdown, grouped by category, each rule as a single bullet with enough context to be self-contained months later. Keep entries short and imperative — a rule, not a story.

```markdown
# Project Rules

Auto-maintained by the project-rules skill. Do not remove entries silently — if a rule seems outdated, ask the user before deleting it.

## Naming & Style
- Use `camelCase` for variables, `PascalCase` for React components. (2026-08-30)
- Never abbreviate variable names beyond 2-3 letters (e.g. no `usr`, `cfg`). (2026-08-30)

## Architecture
- API calls go through `src/lib/api.ts` only — no direct `fetch` calls in components. (2026-08-30)

## Testing
- Every new API route needs a matching test in `tests/routes/`. (2026-08-30)

## Reviewer Feedback
- (rules captured from reviewer-agent comments go here, same format)
```

Categories are flexible — add new ones as needed (e.g. `Error Handling`, `Performance`, `Dependencies`). Don't force a rule into a category it doesn't fit; create a new section instead.

## Workflow A — Apply rules (runs before any code task)

1. Check if `project-rules.md` exists at the project root.
2. If it exists, read it in full and treat every rule as a hard constraint for this task — same priority tier as project style configs (`.eslintrc`, `pyproject.toml`, etc.) under `clean-code-standards`. Project-rules entries win over generic clean-code defaults when they conflict, since they represent explicit user intent for *this* project.
3. If it doesn't exist, proceed normally (optionally offer to bootstrap — see below) — do not block the task waiting for one.

## Workflow B — Capture a new rule (runs whenever feedback is detected)

Trigger this whenever, mid-conversation or in a later turn, one of these happens:
- The user corrects code you (or another agent) just wrote — e.g. "böyle yazma", "bunu şöyle yap", "don't do it that way, do X instead", "bir daha bu hatayı yapma", "always/never do Y in this project".
- The user states a standing preference or convention, even outside of a correction — e.g. "bu projede X kullanıyoruz", "we always structure services this way".
- A reviewer/QA/lint-style subagent flags a code quality, convention, or architecture issue in its output.

When triggered:

1. **Distill, don't transcribe.** Turn the specific complaint into a general, reusable rule. "Bu değişkeni `data` diye adlandırma, çok belirsiz" → *"Avoid generic variable names like `data`, `temp`, `info` — name for what the value represents."* Generalize enough to prevent the whole class of mistake, not just this one instance.
2. **Deduplicate.** Read the existing `project-rules.md` first. If a similar rule already exists, tighten/merge it instead of adding a near-duplicate. If this is the second+ time the same rule had to be restated, that's a signal the existing phrasing is too weak — sharpen it.
3. **Categorize** under an existing section if it fits, otherwise create a new one.
4. **Write the rule** as one bullet, imperative mood, with today's date.
5. **Confirm briefly** to the user what was captured (one line, e.g. "Not aldım: değişken isimlerinde X kuralı") — don't make a big production of it, just a quiet acknowledgment so they know it stuck.

Do not ask permission before saving a rule — capturing corrections is the whole point of this skill and should be zero-friction, matching the project's other skills. Only ask before *deleting* or *overriding* an existing rule outright.

## Workflow C — Setup (run when the user asks to set up / initialize project rules)

This is the "install" step. Run it once per project, and re-run if the user explicitly asks to re-scan.

1. **Scan the codebase.** Read a representative sample of existing files across the languages/frameworks present — enough to infer real conventions, not just skim. Cross-reference what you observe with `clean-code-standards` and any style configs present (`.eslintrc`, `.prettierrc`, `pyproject.toml`, `.editorconfig`, etc.).
2. **Create `project-rules.md`** at the project root, seeded with rules that describe the *existing, observed* conventions of this codebase (not aspirational ones you'd prefer). Use the format above. Always show the user the seeded list right after writing it, since these are inferred rather than explicitly stated by the user — flag anything you're unsure about rather than asserting it as a hard rule.
3. **Wire up existing agents** (see below) so the learning loop is automatic project-wide, not just in this chat.
4. Confirm what was created/patched in a short summary — file path, number of seed rules, which agent files were updated.

## Wiring into existing dev/review agents

If the project has its own subagent definitions — most commonly Claude Code subagents at `.claude/agents/*.md`, but also check for other agent-config conventions the project uses (e.g. a `agents/` folder, CI review-bot configs, custom prompt files referenced from tooling) — patch each relevant one (developer agents, reviewer/QA agents, linter agents) so it participates in the loop without needing this skill re-explained to it each time.

For each agent file found:

1. Read it first — don't blindly overwrite. Check whether it already references `project-rules.md`; if so, skip or update rather than duplicating.
2. Append a small, self-contained block (adapt wording to match the agent's existing style/voice) that instructs it to:
   - Read `project-rules.md` at the project root before starting work, if it exists, and treat its entries as binding constraints alongside any other standards it already follows.
   - Whenever it flags a code-quality/convention issue in review, or whenever the user corrects its output, distill that feedback into a new rule and append it to `project-rules.md` — same distill/deduplicate/categorize approach described in Workflow B above, not a verbatim transcript of the complaint.
3. Save the change and note which file was patched in your summary to the user.

If no agent definition files are found, say so plainly and just proceed with `project-rules.md` on its own — this skill's own Workflow A/B still cover the current session regardless of whether other agents exist.

Never invent or assume agent files exist — always verify by listing/reading the project directory first.

## Relationship to other skills

- `clean-code-standards`: universal baseline, applies everywhere. `project-rules` sits on top of it and is project-specific; on conflict, project-rules wins for that project.
- Do not write anything to a global/user-level file from this skill. If the user wants a rule to apply everywhere, tell them that belongs in `clean-code-standards` or a global memory instead, and don't add it here.

# MAESTRO 🎼 — model-orchestration bundle

**Date:** 2026-07-23
**Status:** Design approved, pending spec review

## Purpose

Let the user run **Opus** as their normal Claude Code driver while using *every*
Claude model at its best. Opus stays the head: it invokes skills (superpowers
etc.), delegates sub-work to subagents on the most suitable model tier, and
consults **Fable** as an adaptive advisor/supervisor when a task benefits.

MAESTRO is the **Opus-headed, Fable-advised** counterpart to the existing
OCTOPUS (which is Fable-headed). OCTOPUS is left **completely untouched**;
MAESTRO ships its own files and shares nothing with it.

Metaphor: Opus is the **maestro** (conductor); the tiered arms are the
**players** (haiku = fast/mechanical, sonnet = workhorse section, opus =
soloist for hard passages); Fable is the **composer/advisor** consulted on
interpretation and to sign off on hard movements.

## Activation (hybrid)

Two ways it engages, so "use Opus normally" is true by default *and* there's a
full-power mode for big tasks:

1. **Always-on doctrine** — a short snippet installed into `~/.claude/CLAUDE.md`.
   Makes every ordinary Opus session route models the smart/adaptive way with
   no command to remember.
2. **`/maestro <task>` command** — explicit full-power orchestration for big
   multi-part tasks: plan → (optional Fable plan critique) → parallel tiered
   arms → independent review → Fable supervisor sign-off on risky results →
   e2e + report.

## Model-routing matrix (the doctrine)

The single source of truth for "which model does what". Used verbatim in both
the always-on doctrine and the `/maestro` planning step.

| Situation | Who does it |
|-----------|-------------|
| Trivial / small change, tightly scoped | **Opus inline** (no subagent) |
| Bulk mechanical, low-risk, fully specified (renames, boilerplate, config/doc edits, repetitive changes) | **haiku** arm |
| Normal feature / fix work; large independent research or codebase sweep | **sonnet** arm (the default arm tier) |
| Genuinely hard brief (subtle algorithm, tricky concurrency, high-ambiguity cross-cutting) that sonnet would likely burn revise rounds on | **opus** arm (rare; justify it) |
| Plan critique, approach tradeoff, "what am I missing", risk read, sign-off on a risky/complex result | **fable-advisor** (advise or supervise) |

**Delegation threshold (always-on mode):** delegate to an arm **only** when
there is real parallelism (2+ genuinely independent chunks) **or** a clear
tier-fit win (bulk mechanical → haiku; big independent research → sonnet).
Otherwise Opus does it inline. Overhead is real — don't spawn an arm to save
Opus one edit.

**Fable threshold (always-on mode):** consult `fable-advisor` only on
genuinely hard / risky / high-ambiguity calls (architecture forks,
security/data-loss-adjacent decisions, "is this approach sound", plan sanity
check on a large change). Not for routine work.

## Components

### A. Always-on doctrine — `the-maestro-doctrine.md`

A portable Markdown snippet shipped in the repo, meant to be pasted/installed
into `~/.claude/CLAUDE.md` (global memory, loaded every session). Content:

- One-paragraph statement of the topology (Opus head, tiered arms, Fable advisor).
- The model-routing matrix (above), condensed.
- The delegation threshold and the Fable threshold.
- A pointer that `/maestro` is available for full-power orchestration.

It must be **short** (a routing reflex, not an essay) and must not fight the
superpowers `using-superpowers` wiring — it reinforces "reach for skills
first", then adds model routing on top.

### B. `/maestro <task>` command — `.claude/commands/maestro.md`

Frontmatter: `description`, `argument-hint: <task to build>`.

Protocol (Opus is THE MAESTRO; never writes feature code itself in this mode):

1. **Scout & plan (Opus).** Trace the real flow, existing conventions, files
   each part touches. Split into the **fewest** independent **briefs**. Each
   brief carries: Goal · Files/area (+overlap notes) · Acceptance criteria
   (concrete, checkable) · Constraints · **Model tier** (haiku/sonnet/opus per
   the matrix) · **Skills** the arm should use (e.g. relevant superpowers /
   project skills). Print the plan before dispatching.
2. **Optional Fable plan critique.** If the task is large / architecturally
   risky / ambiguous, dispatch `fable-advisor` in **advise** mode on the plan
   *before* building. Fold its guidance into the briefs. Skip for
   straightforward tasks (adaptive — scaled to benefit).
3. **Build (parallel arms).** One `maestro-arm` per brief, `model:` set to the
   brief's tier, dispatched in a **single message** for true parallelism. Use
   `isolation: worktree` when briefs may overlap; otherwise working-tree
   parallel is fine for disjoint files. Pass each arm its full brief verbatim.
4. **Review (fresh context).** When an arm reports done, dispatch
   `maestro-reviewer` on the **real diff** (`model: sonnet` default; `haiku`
   allowed for a mechanical brief; never opus). APPROVE, or REVISE + specific
   feedback. Revise loop: send feedback back to the **same** arm (context
   intact), max **2** rounds; then **one** `opus` escalation with a fresh arm +
   full review history; if that still fails, stop and surface to the user.
5. **Fable supervisor sign-off.** Before declaring done, if the change is
   risky / complex / high-blast-radius, dispatch `fable-advisor` in
   **supervise** mode on the integrated result. APPROVE → finish. REVISE →
   route each point back to the owning arm (counts toward its revise budget).
   Skip for low-risk changes (adaptive).
6. **e2e + report (Opus).** Integrate (merge worktrees if used), run real
   build · test · lint. Small integration glue edits allowed; no new feature
   code. Report: plan (task → briefs, tier used, skills used) · which arm built
   each · reviewer verdicts (rounds + any opus escalation) · Fable
   advise/supervise verdicts · e2e results · files changed · anything open.

### C. Agents

**`maestro-arm`** (`.claude/agents/maestro-arm.md`)
- Frontmatter: `name`, `description`, `tools: Read, Write, Edit, Grep, Glob, Bash, TodoWrite`, `model: sonnet` (default; head overrides per-call).
- An executor: builds exactly one brief, nothing more. Scope discipline, follow
  existing conventions, meet acceptance criteria, use the skills the brief
  names, no data-loss/validation shortcuts at trust boundaries. Revise rounds
  keep original context. Terse factual report back (Done/Blocked · files
  changed · how to verify · deviations).

**`maestro-reviewer`** (`.claude/agents/maestro-reviewer.md`)
- Frontmatter: `name`, `description`, `tools: Read, Grep, Glob, Bash, TodoWrite`, `model: sonnet`.
- Fresh-context reviewer: reads the **real diff** (not the arm's report),
  checks every acceptance criterion (runs runnable ones), looks for real
  defects only (correctness, edge cases, broke existing behaviour, violated
  constraint, security/data-loss, out-of-scope). Verdict: **APPROVE** (one line
  on what was verified) or **REVISE** (`file:line — what's wrong — what's
  needed`). No nitpicks, no rubber stamps.

**`fable-advisor`** (`.claude/agents/fable-advisor.md`)
- Frontmatter: `name`, `description`, `tools: Read, Grep, Glob, Bash`, `model: fable`. **No Write/Edit** — it judges, never edits.
- Two modes, selected by how the head prompts it:
  - **Advise** — given a plan / approach / decision + context, returns
    high-level judgment: risks, what's missing, better approaches, tradeoffs.
    Advice only; the head decides.
  - **Supervise** — given a brief/goal + the real result (diff), returns
    **APPROVE** (one line on what convinced it) or **REVISE** (specific,
    actionable points). Reads the actual diff, never signs off from a summary.
- Independence is the point: it reasons from the code and the goal, not from
  the head's intent. Terse, senior, high-signal.

## Files

New (nothing existing is modified):

```
docs/superpowers/specs/2026-07-23-maestro-model-orchestration-design.md  (this)
the-maestro-doctrine.md                    (doctrine snippet, repo root)
.claude/commands/maestro.md                (the /maestro head)
.claude/agents/maestro-arm.md              (executor, tier-parametric)
.claude/agents/maestro-reviewer.md         (fresh-context reviewer)
.claude/agents/fable-advisor.md            (Fable advisor/supervisor, read-only)
README.md                                  (add a MAESTRO section next to OCTOPUS)
```

## Installation (documented in README)

- **Always-on doctrine:** append `the-maestro-doctrine.md` into `~/.claude/CLAUDE.md`.
- **Command + agents everywhere:** copy `.claude/commands/maestro.md` and
  `.claude/agents/maestro-*.md` + `fable-advisor.md` into `~/.claude/`.
- Works in-repo without copying.

## Verification

Bundle is prose/config, so verification is structural + a live smoke test:

- **Structural:** every agent file has valid frontmatter (`name` matches
  filename, `model` is a real id: `haiku`/`sonnet`/`opus`/`fable`, `tools` is a
  valid list); `/maestro` frontmatter has `description` + `argument-hint`;
  model tiers named in the command/doctrine match the matrix.
- **Smoke test (documented, run once by the user):** `/maestro` on a small
  two-part task → confirm the plan prints with per-brief tiers, arms run in
  parallel on the assigned models, reviewer gates on the real diff, and (on a
  deliberately risky brief) `fable-advisor` is consulted.

## Out of scope (YAGNI)

- Modifying OCTOPUS or sharing agents with it.
- Any non-Claude model routing.
- Auto-installing into `~/.claude/` (documented manual step; no installer script).
- A config file for tiers — the matrix lives in the prose; change the prose to
  change routing.

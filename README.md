# My Claude Skills

## 🐙 OCTOPUS

One head that thinks, eight arms that build. Fable plans & orchestrates —
Sonnet executes & reviews.

```
/octopus <task to build>
```

The **head** (run on Fable 5) scouts the codebase and splits the task into
independent briefs, then delegates to Sonnet **arms** (`octopus-executor`)
running in parallel (worktrees if needed). Each brief is gated by an
independent fresh-context Sonnet **reviewer** (`octopus-reviewer`) that reads
the real diff against acceptance criteria — approve, or revise (max 2 rounds,
back to the same arm with context preserved). The head does the final
build/test/integrate and reports plan · sub-tasks · verdicts · results.

Planning and review run on the capable model; implementation runs on the fast,
cheap one — without losing quality, gaining it from independent review.

| File | Role |
|------|------|
| `.claude/commands/octopus.md` | `/octopus` — the head / orchestrator |
| `.claude/agents/octopus-executor.md` | the arms (Sonnet) |
| `.claude/agents/octopus-reviewer.md` | the reviewer (Sonnet, fresh context) |

Works when you're inside this repo. To use `/octopus` everywhere, copy the
files into `~/.claude/`:

```bash
cp .claude/commands/octopus.md ~/.claude/commands/
cp .claude/agents/octopus-*.md ~/.claude/agents/
```

## 🎼 MAESTRO

Opus conducts, tiered arms build, Fable advises. The **Opus-headed,
Fable-advised** counterpart to OCTOPUS — use Opus as your normal Code driver
while every Claude model works at its best.

Two ways it engages:

**1. Always-on doctrine.** Install `the-maestro-doctrine.md` into your global
`~/.claude/CLAUDE.md`. Normal Opus sessions then route models the smart way:
do small work inline, delegate to a **haiku** arm for bulk mechanical work or a
**sonnet** arm for big independent research, and consult **Fable**
(`fable-advisor`) only on the genuinely hard/risky calls.

**2. `/maestro <task>` command** — full-power orchestration for big multi-part
tasks:

```
/maestro <task to build>
```

Opus scouts and splits the task into tiered **briefs**, optionally runs the
plan past Fable, dispatches **arms** (`maestro-arm`) in parallel on the model
each brief needs, gates each on an independent fresh-context **reviewer**
(`maestro-reviewer`, max 2 revise rounds + one opus escalation), gets a
**Fable sign-off** (`fable-advisor`) on the risky results, then does the final
build/test/integrate and reports plan · sub-tasks · verdicts · results.

| File | Role |
|------|------|
| `the-maestro-doctrine.md` | always-on model-routing doctrine (→ `~/.claude/CLAUDE.md`) |
| `.claude/commands/maestro.md` | `/maestro` — the Opus head / conductor |
| `.claude/agents/maestro-arm.md` | the arms (tier-parametric executor) |
| `.claude/agents/maestro-reviewer.md` | the reviewer (fresh context) |
| `.claude/agents/fable-advisor.md` | Fable advisor/supervisor (read-only) |

Works inside this repo. To use everywhere, install globally:

```bash
cat the-maestro-doctrine.md >> ~/.claude/CLAUDE.md
mkdir -p ~/.claude/commands ~/.claude/agents
cp .claude/commands/maestro.md .claude/commands/ensemble.md ~/.claude/commands/
cp .claude/agents/maestro-arm.md .claude/agents/maestro-reviewer.md .claude/agents/fable-advisor.md ~/.claude/agents/
```

## 🎻 ENSEMBLE

The model-agnostic, complexity-aware merge of OCTOPUS and MAESTRO. Triages a
task first — inline, one brief, or full parallel orchestration — then does
exactly that much work, no more.

```
/ensemble <task to build>
```

The head runs on whatever model is driving the session (no Opus/Fable
assumption) and triages the task on two axes: a **size test** (one
coherent diff a single reviewer can check → simple; 2+ independently
reviewable units → complex) and a **risk test** (trust boundary, auth,
migration, irreversible op, real concurrency → mandatory Fable
**SUPERVISE** sign-off, on any path). A stricter trivial sub-case skips
subagents entirely and edits inline. Simple tasks get one brief, one
**arm** (`maestro-arm`), one **reviewer** (`maestro-reviewer`) pass — the
brief's required **Verify** field stands in for a separate e2e step.
Complex tasks get the full MAESTRO protocol inlined: scout & split →
optional Fable **ADVISE** on the plan → parallel tiered arms (worktree
isolation for overlap) → per-brief reviewer gate (max 2 revise rounds + one
opus escalation) → Fable **SUPERVISE** sign-off → head-run e2e build/test/
integrate.

| File | Role |
|------|------|
| `.claude/commands/ensemble.md` | `/ensemble` — the head, triage + both protocols inlined |

No new agent files to install — `/ensemble` reuses `maestro-arm`,
`maestro-reviewer`, and `fable-advisor` unchanged, everything MAESTRO
already lists above. `/ensemble` is the **recommended general-purpose entry
point** going forward; `/maestro` and `/octopus` remain available for
direct invocation of their specific fixed shapes.

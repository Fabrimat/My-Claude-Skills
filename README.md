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
cp .claude/commands/maestro.md ~/.claude/commands/
cp .claude/agents/maestro-arm.md .claude/agents/maestro-reviewer.md .claude/agents/fable-advisor.md ~/.claude/agents/
```

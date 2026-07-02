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

# agent-retro

A Claude Code plugin that gives AI agents a persistent learning loop. After completing tasks, run a retrospective to capture hard-won lessons. Before starting new tasks, the agent automatically recalls relevant past experiences to avoid repeating mistakes.

## How it works

**Two modes:**

1. **Retro mode** (`/retro`) — Run after a task to reflect on what was inefficient, what failed, and what workarounds were discovered. Stores structured learnings in `~/.claude/memory/experiences/`.

2. **Recall mode** (automatic) — Before starting complex tasks, the agent checks the experience index for relevant past lessons and applies them proactively.

**Architecture:**

```
~/.claude/memory/experiences/
├── index.md          # Quick-scan index (one-liner per experience, tagged)
├── git-workflow.md   # Domain-specific experience files
├── testing.md
├── debugging.md
└── ...
```

The two-tier design (lightweight index → detailed domain files) keeps token usage low. The agent reads the index (~50-100 lines) to find matches, then loads only the relevant detail files.

## Install

```bash
claude plugin add -- pkyosx/claude-plugin-retro
```

Or for local development:

```bash
claude --plugin-dir /path/to/claude-plugin-retro
```

## Usage

### After a painful debugging session:

```
/retro
```

Or just describe what happened:

> "I just spent 30 minutes figuring out that cmux browser console list returns empty. Save this lesson."

The agent will:
1. Extract the lesson into a structured entry (Context, Problem, Root Cause, Solution, Rule)
2. Store it in the appropriate domain file
3. Update the index for future recall
4. Show you what it's saving for confirmation

### Before starting a new task:

The agent automatically checks the experience index when you describe a task that matches past challenges. No action needed — it just works.

> "Check for JavaScript errors on our staging site using cmux browser."

The agent will find the relevant experience and use JS hooks from the start, instead of wasting time on the `console list` dead end.

## Experience entry format

```markdown
## [2024-01-15] Brief descriptive title

**Tags:** tool1, tool2, concept1
**Context:** What was being attempted
**Problem:** What went wrong (be specific)
**Root Cause:** Why it happened
**Solution:** What actually worked
**Rule:** One-line actionable guideline for next time
```

## License

MIT

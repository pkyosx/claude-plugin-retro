---
name: agent-retro
description: >
  Agent learning loop — retrospective and experience recall. This skill has two modes:

  **Retro mode (`/retro`):** Run after completing a task to reflect on what was inefficient, what failed,
  what workarounds were discovered, and store structured learnings for future sessions. Use this whenever
  the user says "retro", "retrospective", "what did we learn", "save this lesson", or after a session
  with notable struggles, retries, or dead ends.

  **Recall mode (proactive):** Before starting implementation work on a complex task, ALWAYS consult the
  experience index at ~/.claude/memory/experiences/index.md to check if past sessions encountered similar
  challenges. This is especially important when the task involves tools, patterns, or domains where past
  sessions hit problems. If the user describes a requirement and you recognize it might match a past
  hard-won lesson, check experiences BEFORE diving in. Even if the user doesn't ask — if you're about to
  do something that has a relevant experience entry, read it first. Think of this as "checking your notes
  before the exam."
---

# Agent Retro — Learning Loop

This skill gives you a persistent memory of hard-won lessons. It has two modes:
- **Retro**: reflect on what happened, extract learnings, store them
- **Recall**: before starting work, check if past experiences are relevant

The experience store lives at `~/.claude/memory/experiences/` and persists across all projects and sessions.

## First-Time Setup

If `~/.claude/memory/experiences/` does not exist, create it along with an empty index:

```bash
mkdir -p ~/.claude/memory/experiences
```

Then create `~/.claude/memory/experiences/index.md` with:

```markdown
# Experience Index

One-liner per experience for quick scanning. Format:
`- [file.md#section-anchor] tags: tag1, tag2 — Summary`

## Entries
```

## Retro Mode

### When to run

- **Manual**: User invokes `/retro` after a task
- **Auto-suggest**: If you notice during a session that you hit significant struggles (3+ retries on
  the same approach, workarounds for tool limitations, non-obvious solutions that took multiple attempts),
  suggest running a retro at the end: "That was a tricky one — want me to run `/retro` to capture what
  we learned?"

### How to run a retrospective

1. **Review the session.** Look back through the conversation for:
   - Commands or approaches that failed and had to be retried
   - Workarounds for tool limitations or unexpected behavior
   - Non-obvious solutions that took multiple attempts to discover
   - Patterns that worked well and should be repeated
   - Time sinks where a different approach would have been faster

2. **Draft experience entries.** For each significant learning, write a structured entry:

   ```markdown
   ## [YYYY-MM-DD] Brief descriptive title

   **Tags:** tool1, tool2, concept1, concept2
   **Context:** What was being attempted (1-2 sentences)
   **Problem:** What went wrong or was inefficient (be specific)
   **Root Cause:** Why it happened (the underlying reason, not just symptoms)
   **Solution:** What actually worked (exact commands, code, or approach)
   **Rule:** One-line actionable guideline for next time
   ```

3. **Categorize and store.** Each entry goes into a domain file under `~/.claude/memory/experiences/`.

   Organize by **tool or domain**, not by date or project. This makes retrieval efficient — when you're
   about to use a tool, you check that tool's experience file. Examples:

   - `git-workflow.md` — git, PR, branching lessons
   - `testing.md` — testing patterns and pitfalls
   - `debugging.md` — debugging techniques and gotchas
   - `api-integration.md` — external API patterns
   - `general.md` — cross-cutting workflow lessons

   Create new domain files as needed. If an experience spans multiple domains, put it in the most
   relevant one and add cross-reference tags.

4. **Update the index.** After adding entries, update `~/.claude/memory/experiences/index.md` with a
   one-liner per entry. The index is the scout's quick-scan file — it should be fast to read and
   pattern-match against.

   Format:
   ```markdown
   - [filename.md#section-anchor] tags: tag1, tag2, tag3 — One-line summary of the lesson
   ```

5. **Present to user.** Show the user what you're about to save. Let them confirm, edit, or skip entries
   before writing to disk.

### What makes a good experience entry

**Good entries are specific and actionable.** They capture the non-obvious — things that someone seeing
the task for the first time wouldn't know.

Good:
> **Rule:** When checking console errors via cmux browser, `console list` and `errors list` may return
> empty. Always install JS hooks via `eval` to monkey-patch console.error/warn and capture window.onerror.

Bad:
> **Rule:** Check for console errors carefully.

Good:
> **Rule:** mock.ANY and StrEnum show false diffs in pytest assertion output — only trust the
> "Differing items" section, ignore "Full diff" for those types.

Bad:
> **Rule:** Be careful with test assertions.

**Skip trivial lessons.** Don't store things that are obvious or well-documented. Focus on things that
cost real time to figure out.

## Recall Mode

### When to recall

- **Before complex tasks**: When the user describes a requirement that could involve past challenges,
  check the experience index before starting.
- **When you recognize a pattern**: If a task involves a tool/domain where you know experiences exist
  (because you can see the index), load the relevant file proactively.
- **On struggle detection**: If you're in the middle of a task and hitting walls, check if there's a
  relevant experience you missed.

### How to recall

1. **Read the index** at `~/.claude/memory/experiences/index.md`. This is lightweight — just one-liners
   with tags. Scan it for entries that match the current task's tools, concepts, or error patterns.

2. **Load relevant files.** If the index has matches, read only the specific experience files (not all
   of them). Use the section anchors from the index to jump to the right entry if the file is large.

3. **Apply the lessons.** Use the **Rule** and **Solution** fields to inform your approach. Don't just
   blindly follow them — they're guidelines from past context that may not perfectly fit the current
   situation. But they should save you from repeating known mistakes.

4. **Cite the experience.** When a past experience influences your approach, briefly mention it:
   "Based on a past experience with X, I'll use Y approach instead of Z."

### Token efficiency

The two-tier design (index → detail files) is intentional:
- The index is small enough to always read (~50-100 lines for dozens of experiences)
- Detail files are loaded only when relevant
- This avoids loading thousands of tokens of experiences that don't apply

If the experience store grows very large (100+ entries), consider having a subagent do the recall
scan in the background while you start on the obvious parts of the task.

## Experience Store Structure

```
~/.claude/memory/experiences/
├── index.md              # One-liner per experience, tagged for quick scanning
├── git-workflow.md       # Git workflow lessons
├── testing.md            # Testing patterns and pitfalls
├── debugging.md          # Debugging techniques
├── general.md            # Cross-cutting lessons
└── ...                   # New domain files as needed
```

## Bootstrapping

If the experience store doesn't exist yet or is empty, that's fine — there's nothing to recall. After
the first `/retro`, entries will start accumulating. The value compounds over time as the store grows.

If the user has existing notes in CLAUDE.md or memory files that represent hard-won lessons, suggest
migrating the relevant ones into the experience store format during the first retro.

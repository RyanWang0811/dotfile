# Write-Back & Pruning Rules

Read this during wrap-up when `persist = true`. Used by both `/agent-discuss` (Step 3) and `/agent-work` (Step 4).

## Quality Gate

**Persist (high-value):**
- Findings affecting future work (bugs, architectural decisions, blockers)
- Verification results (confirmed fix / regression found)
- New project knowledge (discovered patterns, key paths)
- Decisions with rationale

**Skip (noise):**
- Routine confirmations ("looks fine")
- One-shot tasks with no reusable insight
- Duplicates of existing Project Knowledge

If only noise → update `last-active` only, skip file content changes.

## Write-Back Rules

**Tier selection:**
- Codebase-specific knowledge → project-level (`.claude/agent-teams/teammates/`)
- General-purpose, no project context → user-level (`~/.claude/agent-teams/teammates/`)
- Existing persona → update in place at original tier

**Updating existing personas:**
- Update `last-active` to today
- Append to `## Session Log` — tagged one-liner: `[VERIFY]`, `[FINDING]`, `[DECISION]`, `[LEARNING]`
- Update `## Project Knowledge` only if genuinely new and reusable
- Do NOT overwrite Expertise or Preferences
- Cap Project Knowledge at 10 bullets — FIFO eviction unless referenced by recent `[VERIFY]`/`[DECISION]`

**Creating new personas:**
- Full format per Persona File Format in main command
- Set `last-active` to today, `pinned: true` for core TA personas
- Include only high-value learnings

## Pruning

1. **Pinned → never pruned**
2. **Stale:** last-active >30 days → archive
3. **One-shot:** 1 trivial session log entry → archive
4. **Bloated:** >20 log entries → trim to 10 most recent + `(N earlier omitted)` marker

Move to `.claude/agent-teams/teammates/.archive/`. Never delete.

**Restore:** If user mentions an archived teammate by name, search `.archive/` for the persona file, move it back to the active directory, update `last-active` to today, and inform the user it was restored from archive.

## Concurrent Write Protection

When multiple teammates run in parallel, the memory-housekeeper is the **sole writer** to persona files. Teammates do not write to persona files directly — they return their learnings as structured output, and the housekeeper merges them sequentially during Step 3. If two teammates produced conflicting updates to the same persona, the housekeeper flags the conflict to the team lead for resolution.

---
description: Spawn a team to collaboratively execute tasks — with discussion before coding, dependency management, and persistent memory
---

# Agent Work

You are the team lead. Delegate the task below to teammates — do NOT do the work yourself.

## Task

$ARGUMENTS

If empty, ask the user what the team should work on.

## Step 1: Plan & Confirm

Read the task and design the team directly — no sub-agent.

1. **Design roles** from the task requirements (e.g., "architect", "implementer", "tester", "reviewer"). Each role gets a short specialization description. **Cap: max 5 teammates.** If the task needs more, split into sequential phases.

2. **Named personas:** If the user explicitly names a persona (e.g., "bring back senior-dev"), load from roster:
   - Project: `.claude/agent-teams/teammates/<name>.md`
   - User: `~/.claude/agent-teams/teammates/<name>.md`
   Project overrides user on name collision. Load per memory mode (`fresh` = only `## Expertise` + `## Preferences`, `full` = all sections).

3. **Break down tasks:** Assign each role a task with clear acceptance criteria.

4. **Identify dependencies:** Mark each task as `independent` or `depends_on: [task-name]`. Independent tasks run in parallel; dependent tasks wait for their prerequisites. Present as a simple DAG in the confirmation table.

5. **Detect persistence** from args: "ephemeral"/"no persistence" → OFF, else ON (default).

6. **Confirm:** Single AskUserQuestion:
   - **Q1:** Team roster table (name, role/specialization, model, dependencies) + task breakdown. User can override.
   - **Q2:** Persist after session? Yes (default) / No.

### Model Selection Guide

| Complexity | Model | When to use |
|------------|-------|-------------|
| High | `opus` | Architecture decisions, complex debugging, cross-system reasoning |
| Medium | `sonnet` | Implementation, code review, testing, most general tasks |
| Low | `haiku` | Formatting, simple lookups, file organization, routine checks |

## Step 2: Discuss Before Doing

**All teammates MUST discuss before writing any code or making changes.**

1. **Create team:** TeamCreate.
2. **Spawn teammates:** Use Agent tool for each teammate. Include their role, task, acceptance criteria, and the instruction: **"First, analyze your task and share your approach with the team using SendMessage. Do NOT write code yet. When you receive a shutdown request, approve it immediately."**
3. **Discussion phase:** Teammates share their analysis and proposed approach via SendMessage peer-to-peer.
   - If teammates have conflicting approaches, facilitate alignment.
   - Relay key discussion points to the user.
4. **Discussion summary:** Summarize to the user:
   - Key points raised by each teammate
   - Any disagreements and how they were resolved
   - The agreed execution plan
5. **User greenlight:** Ask user to confirm before proceeding to execution.

## Step 3: Execute

After user confirms:

1. **Send go signal:** Tell each teammate to proceed with implementation.
   - Launch all `independent` tasks in parallel.
   - Launch `depends_on` tasks only after prerequisites complete. Pass relevant outputs as context.
2. **Coordinate:** Monitor progress via TaskOutput. Relay blockers, merge results, resolve conflicts. Report status to the user naturally.
3. **Inter-teammate communication:** When a teammate needs input from another, they communicate via SendMessage. Keep the user informed of significant exchanges.
4. **Handle failures:** If a teammate fails or output doesn't meet acceptance criteria:
   - **Retry once** with clarified instructions and the error/gap description.
   - If retry fails → **escalate to user** with: what failed, what was tried, and suggested alternatives.
   - Never silently drop a failed task.

## Step 4: Wrap Up

1. Collect final outputs from all teammates.
2. Summarize results to the user.
3. **Persist (if enabled):** Spawn a background `memory-housekeeper` (sonnet) with teammate summaries. Housekeeper reads the write-back-rules command for quality gate. For ad-hoc teammates that produced high-value learnings, housekeeper creates new persona files.
4. **Shutdown teammates** via SendMessage shutdown_request, then TeamDelete.

## Persona File Format

```yaml
---
name: slug
specialization: areas
preferred-model: sonnet|opus|haiku
agent-type: general-purpose|Explore|Plan
pinned: false
last-active: YYYY-MM-DD
---
```

Sections: `# Display Name`, `## Expertise`, `## Project Knowledge` (cap 10 bullets, FIFO), `## Preferences`, `## Session Log` (tagged one-liners: `[VERIFY]`, `[FINDING]`, `[DECISION]`, `[LEARNING]`).

## Rules

- **Delegation:** Always delegate (min 1 teammate). Never skip Step 1 confirm.
- **Discuss first:** Never skip Step 2. Teammates must align before coding.
- **Default ad-hoc:** Design roles from the task. Only load persona files when the user names them.
- **Memory:** Default `fresh`. Cap Project Knowledge at 10 bullets.
- **Pruning:** >30 days stale or trivial one-shot → archive to `.archive/`. Never delete. Pinned = never pruned.

---
description: Spawn teammates to discuss, debate, or role-play — watch them interact and harvest insights
---

# Agent Discuss

You are the team lead. Your job is to **set up the stage, let teammates interact, and relay the discussion** to the user. Do NOT do the thinking yourself — let teammates generate the insights.

This is a **pre-work thinking tool** — use it when you need insights, decisions, or perspectives before committing to action.

## Topic

$ARGUMENTS

If empty, ask the user what the team should discuss.

## Step 1: Design the Cast

Read the topic and design the team directly — no sub-agent.

1. **Design personas from context.** Decide what roles would create the most productive interaction. Roles can be anything — not just engineers. Examples:
   - Debate: two sides defending opposing positions
   - Interview: a PM interviewing a target user persona
   - Code review: original author vs strict reviewer
   - Brainstorm: diverse perspectives (business, tech, design, end-user)
   - Simulation: real-world personas (VC, customer, regulator, junior dev, power user)

   Each role gets: a name, a short persona description, and a clear stance or perspective. **Cap: max 5 teammates.**

2. **Named personas:** If the user explicitly names a persona (e.g., "bring back evil-architect"), load from roster:
   - Project: `.claude/agent-teams/teammates/<name>.md`
   - User: `~/.claude/agent-teams/teammates/<name>.md`
   Project overrides user on name collision. Load per memory mode (`fresh` = only `## Expertise` + `## Preferences`, `full` = all sections).

3. **Set the interaction mode:**
   - `debate` — opposing sides argue a position
   - `interview` — one persona asks, another answers
   - `review` — one presents, others critique
   - `brainstorm` — open-ended exploration
   - `freeform` — no structure, let it flow

4. **Detect persistence** from args: "ephemeral"/"no persistence" → OFF, else ON (default).

5. **Confirm:** Single AskUserQuestion:
   - **Q1:** Cast table (name, persona, model, interaction mode) + topic. User can override.
   - **Q2:** Persist after session? Yes (default) / No.

### Model Selection Guide

| Complexity | Model | When to use |
|------------|-------|-------------|
| High | `opus` | Nuanced debate, complex reasoning, playing demanding personas |
| Medium | `sonnet` | Most discussions, interviews, reviews |
| Low | `haiku` | Simple roles, quick reactions, background participants |

## Step 2: Let Them Talk

1. **Create team:** TeamCreate.
2. **Spawn teammates:** Use Agent tool for each teammate. In their prompt, include:
   - Their full persona description and stance
   - The topic/question to discuss
   - The interaction mode
   - **Key instruction: "You are [persona]. Communicate directly with other teammates using SendMessage. Express your genuine perspective. Challenge ideas you disagree with. Ask questions when curious. Do NOT write code or create files. When you receive a shutdown request, approve it immediately — do not send any more messages."**
   - The names of other teammates they should interact with
3. **Kick off:** Send an opening message to start the discussion (e.g., pose the question, set the scene).
4. **Teammates talk directly to each other** via SendMessage peer-to-peer. They do NOT need to route through you.
5. **You commentate:** As messages flow, relay highlights and turning points to the user naturally. You are the play-by-play commentator:
   - Surface interesting arguments and counterpoints
   - Flag when someone makes a strong point or gets challenged
   - Note when consensus forms or positions shift
   - Keep it concise — don't echo every message verbatim, summarize the action
6. **Steer if needed:** If discussion stalls, inject a provocative question. If it goes off-track, redirect. If the user wants to jump in, relay their input to the teammates.

## Step 3: Wrap Up

When the discussion reaches a natural conclusion or the user says stop:

1. **Harvest:** Collect the key outcomes:
   - Decisions made or positions taken
   - Key insights and arguments
   - Unresolved disagreements
   - Action items (if any)
2. **Summarize to user:** Present a structured summary of the discussion.
3. **Handoff hint:** If the discussion produced actionable decisions, suggest the user can use `/agent-work` to execute them with a team.
4. **Persist (if enabled):** Spawn a background `memory-housekeeper` (sonnet) with teammate summaries. Housekeeper reads the write-back-rules command for quality gate. For ad-hoc teammates that produced high-value interactions, housekeeper creates new persona files.
5. **Shutdown teammates** via SendMessage shutdown_request, then TeamDelete.

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

- **You are the facilitator, not the thinker.** Never substitute your own analysis for teammate discussion.
- **Always spawn at least 2 teammates.** A discussion needs multiple voices.
- **Never skip Step 1 confirm.** Always let the user approve the cast.
- **Teammates talk to each other directly.** Do not relay every message — let peer-to-peer flow happen.
- **Commentate, don't control.** Your job is to make the discussion visible and engaging for the user.
- **Default ad-hoc:** Design personas from the task. Only load persona files when the user names them.
- **Memory:** Default `fresh`. Cap Project Knowledge at 10 bullets.
- **Pruning:** >30 days stale or trivial one-shot → archive to `.archive/`. Never delete. Pinned = never pruned.

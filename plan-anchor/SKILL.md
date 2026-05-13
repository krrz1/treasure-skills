---
name: plan-anchor
description: When the user says "/anchor", "/plan-anchor", "save state", "恢复上下文", "上下文丢了", or any phrase indicating they want to persist or recover task state across conversation turns. Use this skill whenever a user is in the middle of a multi-step development task and wants to anchor the current context to a file, or when they need to recover lost context after a compaction. Trigger proactively when the user mentions losing track of progress, forgetting earlier decisions, or needing to resume a complex task.
---

# Overview

This skill provides a lightweight state persistence layer for long, multi-turn development conversations. It stores the essential context (progress, decisions, constraints, pitfalls) in a `plan_state.md` file so that if the conversation context gets compacted, the user can recover by invoking this skill again.

# State File Location

The state file lives at:

```
docs/plan-<slug>/plan_state.md
```

- `slug` is a short kebab-case identifier for the task (e.g., `smtp-rate-limit`).
- If multiple state files exist, prefer the one matching the current conversation topic, or ask the user which to use.

# Three Operations

## 1. Anchor — Create or Update State

Trigger: user says `/anchor <slug>`, `/plan-anchor`, "save state", or asks to record progress.

**If the state file does not exist (new task):**
1. Ask for the `slug` if not provided.
2. Create `docs/plan-<slug>/` directory.
3. Create `plan_state.md` from the template in [references/state-template.md](references/state-template.md).
4. Fill in what is already known from the conversation:
   - Current progress / active step
   - Any decisions the user has already confirmed
   - Known tech stack or constraints
   - Open questions still pending
5. Show the user the created file and confirm it looks correct.

**If the state file already exists (update):**
1. Read the existing `plan_state.md`.
2. Compare it with the current conversation. Identify anything new since the last save:
   - New CONFIRMED decisions
   - New PENDING questions
   - New pitfalls encountered
   - Progress changes (which step is now active or done)
3. Update the file, incrementing the version number in the header.
4. Tell the user what was updated.

## 2. Update — Periodic Sync During Execution

Trigger: the user says "update anchor", or you have just completed a significant milestone (e.g., finished a plan step, merged a PR, resolved a blocking issue).

Action:
1. Read the current `plan_state.md`.
2. Append or edit entries to reflect the latest state.
3. Focus on facts that are costly to rediscover: decisions, constraints, workarounds, active files.
4. Do not overwrite the file blindly — merge new information with existing state.

## 3. Recover — Restore Context After Compaction

Trigger: the user says "恢复上下文", "context lost", "where were we", or you detect that earlier conversation context is missing (e.g., you no longer remember the plan steps that were agreed upon).

Action:
1. Find and read `docs/plan-*/plan_state.md`.
2. Present a concise recovery summary to the user in this exact format:

   ```
   [Context Recovery] Restored from plan_state.md

   Progress: Step X/Y — brief description
   Frozen decisions: #1 ..., #2 ...
   Pending: #3 ... (blocking / deferred)
   Active files: path/to/file.go
   ```

3. Ask: "Continue from here, or do you want to adjust anything?"
4. After the user confirms, resume work. Update `plan_state.md` if the user provides corrections.

# What to Store

Keep the state file lean. Only record information that is expensive to lose or rediscover:

- **CONFIRMED decisions**: numbers, thresholds, architecture choices, prohibitions (e.g., "do not modify User struct").
- **PENDING questions**: open items that affect future steps, with a note on whether they are blocking.
- **Known facts**: tech stack, data scale, performance targets — things that will not change during this task.
- **Pitfalls**: non-obvious failures or workarounds you encountered.
- **Task checklist**: which plan steps are done, which is active, which remain.

Do not store:
- Full code snippets (link to files instead).
- Transient debugging output.
- Information already in auto memory (user preferences, project-wide conventions).

# Format Rules

- Use the template from [references/state-template.md](references/state-template.md).
- Increment the version number (`State v{N}`) on every write.
- Keep each bullet concise — one line per item.
- Use `[BLOCKING]` and `[DEFERRED]` tags on PENDING items so recovery is clear about what can proceed.

---
name: mycelium-plan
description: Create implementation plan with task breakdown
argument-hint: "[task description] | --list | --switch <track_id>"
allowed-tools: ["Skill", "Read", "Write", "Edit", "Glob", "Grep", "AskUserQuestion"]
---

# Workflow Plan

Transform user request into structured, executable plan with TDD task breakdown. Also supports listing and switching between multiple plans.

## Your Task

### Route by argument

**If `--list`**: Jump to [List Plans](#list-plans).
**If `--switch <track_id>`**: Jump to [Switch Plan](#switch-plan).
**Otherwise**: Continue with [Create Plan](#create-plan).

---

### Create Plan

1. **Update session state** - Write `invocation_mode: "single"` to `.mycelium/state/session_state.json`

2. **Load the planning skill** - Use Skill tool to load `planning`

3. **Parse input**:
   - If user provided task description: Use it
   - If empty: Ask user for task description

4. **Provide context**:
   - Read `.mycelium/state/session_state.json`
   - Read `.mycelium/context/*.md` (product, tech-stack, workflow)
   - Read `CLAUDE.md` if exists

5. **Execute planning workflow** - Follow the loaded `planning` skill which handles:
   - Requirements clarification (Phase 1) using AskUserQuestion
   - Smart research gate (Phase 1.5) - grep codebase before web search
   - Capability discovery (Phase 2) - check available skills/agents
   - Task breakdown and plan creation (Phase 3) - TDD-driven tasks with dependencies

6. **Save plan** to `.mycelium/plans/YYYY-MM-DD-{track-id}.md`

7. **Register plan in session state**:
   - Read `session_state.json`
   - If a plan in `plans[]` has `status: "in_progress"`, set it to `"paused"` (both in `plans[]` AND in that plan file's YAML frontmatter `status` field)
   - Append new plan entry to `plans[]`:
     ```json
     {
       "track_id": "{track_id}",
       "plan_file": "YYYY-MM-DD-{track-id}.md",
       "status": "in_progress",
       "created": "{timestamp}",
       "total_tasks": {count},
       "completed_tasks": 0
     }
     ```
   - Set `current_track` to the new plan's `{ "id": "{track_id}", "type": "{type}", "plan_file": "..." }`

8. **Next step**: Suggest `/mycelium-work` to execute the plan

---

### List Plans

Display all plans from `session_state.plans[]` (fall back to globbing `.mycelium/plans/*.md` and reading frontmatter if `plans[]` is missing or empty).

**Output format:**
```
Plans:
 * multi-plan_20260211  in_progress  2/7 tasks  (active)
   auth_20260210        paused       0/5 tasks
   bugfix_20260209      completed    3/3 tasks
```

- `*` marks the active plan (matches `current_track.id`)
- Exclude `preview-` prefixed files unless `--all` is passed
- Sort by `created` descending (newest first)

---

### Switch Plan

Switch the active plan to `<track_id>`:

1. Read `session_state.json`
2. Find `<track_id>` in `plans[]`. If not found, check `.mycelium/plans/` for a matching file and register it first. If still not found, error: "Plan `<track_id>` not found."
3. Set the current active plan (the one with `status: "in_progress"`) to `"paused"` in both `plans[]` and its plan file frontmatter
4. Set the target plan to `"in_progress"` in both `plans[]` and its plan file frontmatter
5. Update `current_track` to point to the target plan
6. Show confirmation: "Switched to plan `<track_id>`"
7. Suggest `/mycelium-work` or `/mycelium-continue` to resume

---

## Skills Used

- **planning**: Core planning workflow (clarify → research → discover → plan)
- **context**: For loading project knowledge
- **tdd**: Referenced during task breakdown

## Quick Example

```bash
# Create a new plan
/mycelium-plan "Add user authentication"

# List all plans
/mycelium-plan --list

# Switch to a different plan
/mycelium-plan --switch auth_20260210

# Create another plan (previous one auto-pauses)
/mycelium-plan "Optimize database queries"
```

## Important

- Plans are LIVING DOCUMENTS - updated in-place during execution
- All tasks follow TDD: tests before implementation
- Tasks have explicit dependencies (blockedBy/blocks)
- Default to parallel execution - minimize dependencies
- **Creating a new plan auto-pauses the previous active plan** - no plans are lost
- **Backward compatible** - works when `plans[]` doesn't exist (falls back to globbing `.mycelium/plans/`)

---
name: mycelium-status
description: Display current workflow progress and state
argument-hint: "[--verbose]"
allowed-tools: ["Read", "Bash", "Glob"]
---

# Workflow Status

Display current workflow state and progress dashboard.

## Your Task

1. **Load current state**:
   - Read `.workflow/state/session_state.json`
   - Read active plan from `.workflow/plans/`
   - Get git status and recent commits

2. **Discover capabilities** (extract from current session context - do NOT hardcode or guess):

   **Skills** - Check ALL of these sources and merge (deduplicate by name):
   - Primary: The system-reminder block listing skills available for the Skill tool. Extract every `plugin:skill-name` entry.
   - Secondary: The Skill tool's own description or metadata if it lists available skills.
   - Note: The system-reminder may NOT list all registered skills due to context optimization. Some plugin skills (e.g., git:commit-and-push) may be registered but not injected into the prompt. Acknowledge any known gaps.

   **Agents** - Read the Task tool description COMPLETELY. Extract EVERY agent listed under "Available agent types and the tools they have access to". This includes:
   - Built-in agents (Bash, general-purpose, Explore, Plan, claude-code-guide, statusline-setup, etc.)
   - Plugin agents (e.g., code-simplifier:*, mycelium:*, etc.)
   - IMPORTANT: Do not skip any agent. Read the FULL list carefully. Common missed agents: `claude-code-guide`, `statusline-setup`.

   **MCP Tools** - Check for any MCP server tools available in the current session (listed as additional tools from MCP servers in the system prompt or tool list).

3. **Display dashboard**:
   ```
   📊 Mycelium Status

   Current Track: {track_id}
   Phase: {current_phase}

   Tasks:
   - ✅ Completed: {count} ({percentage}%)
   - 🔄 In Progress: {count}
   - ⏳ Pending: {count}
   - 🚫 Blocked: {count}

   Available Skills: {count} skills discovered
   Available Agents: {count} agents discovered
   Available MCP Tools: {count} MCP tools discovered

   Recent Activity:
   - {recent commits}

   Next Action: {suggested_command}
   ```

4. **Suggest next action** based on state:
   - No plan → `/mycelium-plan`
   - Plan exists, tasks pending → `/mycelium-work`
   - All tasks complete → `/mycelium-review`
   - Review complete → `/mycelium-capture`
   - Blockers detected → Address blockers or use `/mycelium-continue`

5. **Verbose mode** (`--verbose`):
   - Detailed task breakdown by phase
   - Full list of discovered skills with descriptions
   - Full list of discovered agents with capabilities
   - Full list of discovered MCP tools with descriptions
   - Git status and stashes
   - Test coverage metrics
   - Context usage estimate
   - Recent solutions captured

## Quick Example

```bash
/mycelium-status       # Standard view
/mycelium-status -v    # Verbose view with details
```

## Important

- **Read-only operation** - Does not modify any state
- **Fast execution** - Should complete in <2 seconds
- **Actionable** - Always shows clear next steps
- **Visual** - Uses box drawing and symbols for clarity

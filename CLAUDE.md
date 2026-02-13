# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mycelium is a meta-workflow orchestration plugin for Claude Code that implements systematic development with compounding knowledge capture. It creates an intelligent substrate that learns from each development session, making subsequent work progressively easier through pattern recognition and knowledge accumulation.

## Development Commands

```bash
# Testing
npm test                    # Run all tests (uses --experimental-vm-modules for ESM support)
npm run test:watch          # Watch mode
npm run test:coverage       # Generate coverage report (target: >80%)

# Linting
npm run lint                # ESLint on lib/ directory

# Git operations
git worktree add .worktrees/<name> -b <branch>  # Create parallel feature worktree
git worktree remove .worktrees/<name>           # Cleanup after merge
```

## Architecture

### Three-Layer System

1. **Skills Layer** - User-invocable slash commands and internal workflow enforcement
   - User-facing skills (`/mycelium-*`): Self-contained workflow orchestrators
   - Internal skills (TDD, verification, context, recovery): Reusable domain expertise
   - Location: `skills/*/SKILL.md`

2. **Agents Layer** - Specialized analysis and knowledge management
   - `learning-agent`: Pattern detection, knowledge capture
   - `spec-compliance-reviewer`: Requirements verification
   - `code-quality-reviewer`: Security, performance, architecture analysis
   - Location: `agents/*.md`

3. **Library Layer** - Programmatic infrastructure (no orchestration)
   - Foundation: validation, templating, state management, pattern detection
   - Scheduler: dependency graphs, task scheduling, worktree tracking, merge analysis
   - Discovery: multi-scope capability scanning (agents, skills, MCPs)
   - Location: `lib/**/*.js`

**Philosophy:** Skills and agents handle AI-powered workflow logic. Libraries provide data structures, algorithms, and infrastructure only.

### Workflow Phases

The system implements a **12-phase workflow** with true phase isolation - one skill per phase:

**Bootstrap & Context:**
- **Phase -1**: Project Bootstrap (`mycelium-setup`) - Initialize `.mycelium/` structure via interactive Q&A
- **Phase 0**: Context Loading (`mycelium-context-load`) - Load project context + discover all capabilities (skills, agents, MCPs) → cache in state.json
- **Phase 1**: Clarify Request (`mycelium-clarify`) - Ask clarifying questions ONE at a time, using cached capabilities
- **Phase 1.5**: Research (optional, within `mycelium-clarify`) - WebSearch/WebFetch for unfamiliar tech

**Planning & Implementation:**
- **Phase 2**: Planning & Assignment (`mycelium-plan`) - Decompose request → tasks, assign agent/skills/model per task
- **Phase 3**: Implementation (`mycelium-work`) - Execute tasks with TDD enforcement (RED → GREEN → REFACTOR)
- **Phase 4.5**: Verification (internal, `verification` skill) - Evidence-based testing with actual output
- **Phase 4.5B**: Context Sync (auto at 80%, `context` skill) - Summarize + spawn fresh agent if needed

**Review & Finalization:**
- **Phase 5**: Two-Stage Review (`mycelium-review`) - Stage 1: Spec compliance (gate) + Stage 2: Code quality (parallel agents)
- **Phase 6**: Finalization (`mycelium-finalize`) - Create git commit with Co-Author + pull request
- **Phase 6E**: Pattern Detection (`mycelium-patterns`) - Find 3+ similar solutions, update critical-patterns.md, recommend skill generation
- **Phase 6F**: Store Knowledge (`mycelium-capture`) - Save to solutions/, learned/, preferences.yaml

**Orchestration:**
- Full workflow: `/mycelium-go` chains all phases automatically (Phase 0 → 6F)
- Resume workflow: `/mycelium-continue` dispatches to current phase based on state.json

### Phase-to-Skill Mapping

Each phase is managed by exactly one skill for clean separation of concerns:

| Phase | Skill | Type | Responsibility |
|-------|-------|------|----------------|
| -1 | `mycelium-setup` | User-invocable | Bootstrap project structure |
| 0 | `mycelium-context-load` | User-invocable | Load context, discover capabilities |
| 1 | `mycelium-clarify` | User-invocable | Clarify requirements, determine research |
| 1.5 | (within `mycelium-clarify`) | Internal | Optional external research |
| 2 | `mycelium-plan` | User-invocable | Plan tasks, assign capabilities |
| 3 | `mycelium-work` | User-invocable | Execute tasks with TDD |
| 4.5 | `verification` | Internal | Evidence-based validation |
| 4.5B | `context` | Internal | Context management (auto) |
| 5 | `mycelium-review` | User-invocable | Two-stage review |
| 6 | `mycelium-finalize` | User-invocable | Commit + PR creation |
| 6E | `mycelium-patterns` | User-invocable | Pattern detection |
| 6F | `mycelium-capture` | User-invocable | Knowledge storage |

**Phase Handoff Protocol:**
- Each skill updates `current_phase` in state.json upon completion
- If `invocation_mode == "full"`: skill auto-invokes next phase skill
- If `invocation_mode == "single"`: skill suggests next phase, then stops

**Example workflow chain (full mode):**
```
mycelium-go (sets mode=full)
  → mycelium-context-load (Phase 0)
    → mycelium-clarify (Phase 1)
      → mycelium-plan (Phase 2)
        → mycelium-work (Phase 3, includes 4.5 + 4.5B)
          → mycelium-review (Phase 5)
            → mycelium-finalize (Phase 6)
              → mycelium-patterns (Phase 6E)
                → mycelium-capture (Phase 6F)
                  → workflow_complete = true
```

### Knowledge Compounding System

The `.mycelium/` directory structure creates an institutional memory:

```
.mycelium/
├── context/           # Project understanding (product.md, tech-stack.md, workflow.md)
├── plans/             # Implementation plans with task breakdown
├── solutions/         # Documented solutions
│   └── patterns/
│       └── critical-patterns.md  # Auto-promoted patterns (3+ occurrences)
├── learned/           # Learning store
│   ├── decisions/     # Architectural decisions with context
│   ├── conventions/   # Detected code patterns
│   ├── preferences.yaml  # User preferences from corrections
│   ├── anti-patterns/ # Mistakes to avoid
│   └── effective-prompts/  # Successful approaches
├── state.json         # Session state + discovered capabilities cache
└── progress.md        # Human-readable progress tracking
```

**Feedback loops:**
- Solutions → Patterns (3+ occurrences) → critical-patterns.md → Skills (auto-generation)
- User corrections → preferences.yaml → Future behavior adaptation
- Context > 80% → Summarize + spawn fresh agent with compressed context

### Hierarchical Task Decomposition

All work follows this structure:

```
User Request
  ↓
Features (user-facing capabilities)
  ↓
Tasks (2-5 minute atomic units with TDD)
```

**Git Strategy:**
- Single feature → `git checkout -b feature/name`
- Multiple features → Git worktrees in `.worktrees/` (gitignored)
  - Each worktree gets its own branch and fresh agent context
  - Enables true parallel development with isolation

## Key Conventions

### Iron Law of TDD

**Non-negotiable**: Tests MUST be written first. Implementation MUST NOT proceed without failing tests.

The 11-step TDD cycle (enforced by `skills/tdd/SKILL.md`):
1. Write the test first
2. Run the test - watch it fail
3. Write minimal implementation
4. Run tests - watch it pass
5. Run full suite - ensure no regressions
6. Refactor for clarity
7. Run tests again
8. Document edge cases
9. Test edge cases
10. Final verification
11. Commit with tests

**Why:** AI-generated code can appear correct but fail on edge cases or integration. TDD proves correctness.

### Evidence-Based Verification

Never claim something "should work" - provide actual test output as proof.

The verification skill requires:
- Execute all tests and show results
- Check coverage ≥80% (configurable by maturity mode)
- Run linters and show output
- Verify build succeeds
- Evidence, not assertions

### Capability Discovery and Caching

Discovery happens in Phase 0 (SessionStart hook) and caches results in `state.json`:

```javascript
// lib/discovery/cache-manager.js handles TTL-based caching
discovered_capabilities: {
  agents: [...],      // From all plugin scopes
  skills: [...],      // Including user-invocable and internal
  mcps: [...],        // MCP servers providing tools
  last_updated: timestamp,
  ttl: 3600000       // 1 hour default
}
```

**Scopes searched** (in order):
1. `local` - `./.claude/` (project-specific)
2. `project` - Project plugins
3. `user` - `~/.claude/plugins/` (user-wide)
4. `global` - System-wide plugins

**When to refresh:**
- SessionStart hook (automatic)
- Cache age > TTL
- Explicit invalidation

### State Management

All workflow state lives in `.mycelium/state.json` with JSON Schema validation:

```javascript
{
  current_session: { id, started_at },
  active_tracks: [{ track_id, status, tasks }],
  current_phase: "context_loading" | "clarify_request" | "planning" | "implementation" |
                 "verification" | "context_sync" | "review" | "finalization" |
                 "pattern_detection" | "store_knowledge" | "completed",
  checkpoints: { context_loading_complete, planning_complete, implementation_complete, ... },
  discovered_capabilities: { agents, skills, mcps },
  active_worktrees: [{ path, branch, track_id }],
  metrics: { tasks_completed, context_resets, interventions }
}
```

Use `lib/state-manager.js` for dot-notation updates:
```javascript
await updateStateField('.mycelium/state.json', 'current_phase', 'implementation');
```

### Skill Architecture Patterns

**User-facing skills** (e.g., `mycelium-go`, `mycelium-plan`):
- Full workflow logic in SKILL.md
- Self-contained orchestration
- Can invoke other skills via `Skill` tool
- Can spawn agents via `Task` tool
- **New**: Progressive disclosure via `references/` directories for complex skills
- **New**: Examples and Troubleshooting sections for top-priority skills

**Phase Isolation Principle**:
- **One skill manages one phase** - No multi-phase skills
- Each skill updates `current_phase` in state.json upon completion
- Skills chain via phase handoff protocol (full mode) or suggest next skill (single mode)
- `invocation_mode` determines chaining behavior: `full` (auto-chain) vs `single` (stop after phase)

**Internal skills** (e.g., `tdd`, `verification`):
- Set `user-invocable: false` in frontmatter
- Provide reusable domain expertise
- Called by other skills, never directly by users

**Agent definitions** (e.g., `learning-agent.md`):
- Frontmatter: name, description, model, tools, color
- Examples in frontmatter showing when to use
- Detailed instructions in body
- Spawned via Task tool with `subagent_type`

**Skill Metadata** (all skills as of v0.9.0):
- `license: MIT` - Open source license
- `version: 0.9.0` - Semantic versioning matching plugin
- `metadata` object with: author, category, tags, documentation URL
- Descriptions follow "WHAT + WHEN + trigger phrases" format per [Agent Skills Guide](https://agentskills.io)

### Pattern Detection and Skill Generation

The learning agent tracks solution patterns:

1. Solutions documented in `.mycelium/solutions/*.md` with YAML frontmatter
2. Pattern detector finds 3+ occurrences with similar tags/problem_type
3. Auto-promotes to `solutions/patterns/critical-patterns.md`
4. Recommends skill generation for recurring patterns
5. Generated skills follow AgentSkills.io standard

### Dependency Management

Tasks can specify blocking relationships for parallel execution:

```yaml
Task 1.1:
  blockedBy: []      # Can start immediately
  blocks: [1.2, 1.3] # These wait for 1.1

Task 1.2:
  blockedBy: [1.1]
  blocks: [2.1]
```

The scheduler (`lib/scheduler/task-scheduler.js`) uses topological sort to determine execution order.

### Model Tier Selection

Tasks can specify model preference:

```yaml
Task 1.1: Complex architecture design
  model: opus

Task 1.2: Standard CRUD implementation
  model: sonnet  # Default

Task 1.3: Fast code review
  model: haiku
```

## Project-Specific Patterns

### Progressive Disclosure System

Following the [Agent Skills standard](https://agentskills.io), Mycelium skills use a 3-level progressive disclosure system:

**Level 1 (YAML Frontmatter):** Always loaded in Claude's system prompt
- Name, description (with trigger phrases), license, version, metadata
- Helps Claude decide when to load the skill without loading full content
- Kept concise (<1024 chars for description)

**Level 2 (SKILL.md Body):** Loaded when Claude thinks skill is relevant
- Core workflow instructions
- Essential steps and decision gates
- Quick examples
- Kept focused (target 2000-3000 words, max 5000)

**Level 3 (references/ Files):** Loaded only as needed
- Detailed examples in `references/examples/`
- Troubleshooting guides in `references/troubleshooting.md`
- Advanced usage patterns in `references/advanced-usage.md`
- Complex skills have `references/README.md` for navigation

**Skills with references/ structure:**
- mycelium-go (examples/, advanced-usage/)
- mycelium-review (review-criteria/)
- mycelium-capture
- mycelium-plan
- recovery (protocols/)
- context

### Templates System

Templates use `{{variable}}` syntax with nesting and filters:

```javascript
// lib/template-renderer.js
await render('{{user.name | uppercase}}', { user: { name: 'alice' } });
// Result: "ALICE"
```

All templates in `templates/` directory with subdirectories:
- `project/` - Bootstrap templates (5 files)
- `plans/` - Plan structure
- `solutions/` - Solution documentation format
- `state/` - Progress tracking
- `gitignore/` - Language-specific .gitignore files

### Schema Validation

All workflow files validate against JSON schemas in `schemas/`:

```javascript
// lib/schema-validator.js
import { validateConfig } from './lib/schema-validator.js';

const result = await validateConfig(data, schemaPath);
if (!result.valid) {
  console.error('Validation errors:', result.errors);
}
```

Critical schemas:
- `session-state.schema.json` - State file structure
- `plan-frontmatter.schema.json` - Plan YAML frontmatter
- `solution-frontmatter.schema.json` - Solution YAML frontmatter
- `enums.json` - Shared enum definitions

### Hooks

Two lifecycle hooks (defined in `hooks/hooks.json`):

1. **SessionStart** (`hooks/scripts/session-start.sh`):
   - Load project context
   - Discover capabilities (scan all plugin scopes)
   - Cache discovered capabilities in state.json
   - Load institutional knowledge from `.mycelium/learned/`

2. **Stop** (`hooks/scripts/save-state.sh`):
   - Persist session state
   - Backup state.json
   - Update metrics

## Testing Approach

### Test Organization

```
test/
├── unit/              # Library unit tests
├── integration/       # Workflow integration tests
└── fixtures/          # Test data and mock plugins
```

### Running Tests

- Tests use ES modules (type: "module" in package.json)
- Jest requires `--experimental-vm-modules` flag
- Run with `--runInBand` to avoid parallelism issues with state files
- Use `test/fixtures/` for mock data, never modify actual `.mycelium/` in tests

### Coverage Requirements

Target coverage per maturity mode (set in `.mycelium/context/workflow.md`):
- Prototype: 50%
- Development: 80%
- Production: 90%
- Regulated: 95%

## Common Workflows

### Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add frontmatter with name, description, user-invocable, allowed-tools
3. Write workflow logic in markdown body
4. Add examples if creating an agent definition
5. Test via `/skill-name` invocation
6. No registration needed - auto-discovered by capability scanner

### Adding a Library Utility

1. Write tests first in `test/unit/`
2. Implement in `lib/` (use ES modules)
3. Add JSDoc documentation
4. Export functions explicitly
5. Update `lib/README.md` if adding new module
6. Ensure >80% coverage

### Creating an Agent

1. Create `agents/<agent-name>.md`
2. Frontmatter: name, description, model, tools, color
3. Add examples showing when to use (see `learning-agent.md`)
4. Write detailed instructions
5. Spawn via Task tool: `subagent_type: "mycelium:<agent-name>"`

## Important Gotchas

1. **Never skip TDD** - Even for "simple" changes. Tests first, always.

2. **Git worktrees are gitignored** - `.worktrees/` never committed. Each worktree is temporary workspace.

3. **Context sync at 80%** - When context usage > 80%, Phase 4.5B triggers automatic summarization and fresh agent spawn.

4. **Capability cache has TTL** - Cached capabilities expire after 1 hour. Stale cache = missing new plugins.

5. **State files are JSON** - Use `lib/state-manager.js` for updates to ensure atomic writes and backups.

6. **Skills are markdown** - Skills use SKILL.md, not JSON. Frontmatter in YAML, body in markdown.

7. **Parallel features = separate worktrees** - Multiple features must use worktrees, not just branches, for context isolation.

8. **Patterns need 3+ occurrences** - Pattern detector only promotes to critical-patterns.md after finding 3+ similar solutions.

9. **Evidence required** - Never say "tests pass" without showing actual test output. Verification skill enforces this.

10. **Two-stage review is sequential** - Stage 1 (spec compliance) must pass before Stage 2 (code quality) runs. Stage 1 is a gate.

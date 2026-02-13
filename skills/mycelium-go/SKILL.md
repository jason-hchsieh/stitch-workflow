---
name: mycelium-go
description: Executes complete autonomous development workflow from planning to knowledge capture. Use when user says "build [feature]", "implement [functionality]", "fix [bug]", "debug [issue]", "investigate [problem]", "optimize [component]", or asks technical questions requiring systematic solution. Handles feature development, debugging, investigation, and optimization with full plan → work → review → capture cycle.
license: MIT
version: 0.9.0
argument-hint: "[task/question/issue description] [--interactive]"
allowed-tools: ["Skill", "Read", "Write", "Edit", "Bash", "Glob", "Grep", "Task", "AskUserQuestion"]
metadata:
  author: Jason Hsieh
  category: workflow
  tags: [workflow-orchestration, tdd, systematic-development, compound-engineering]
  documentation: https://github.com/jason-hchsieh/mycelium
---

# Workflow Go

Execute the complete mycelium workflow autonomously from planning to knowledge capture. Handles feature development, bug debugging, technical questions, investigation, and optimization.

## Your Task

1. **Parse arguments**:
   - `task description`: What to build, fix, debug, investigate, or answer
   - `--interactive`: Enable human approval at each phase (default: autonomous)

2. **Update session state** - Write `invocation_mode: "full"` to `.mycelium/state.json`

3. **Execute full workflow** - Follow the phases below (plan → work → review → capture)

4. **Final report** - Summarize completed work, test results, captured learnings

## Continue Mode

When invoked from `/mycelium-continue`, the workflow resumes from a saved checkpoint rather than starting fresh.

### Phase Mapping

Map `current_phase` in `state.json` to named phases:

| `current_phase` value | Phase name | Phase number |
|-----------------------|------------|-------------|
| `planning` | Plan | 1 |
| `implementation` | Work | 2 |
| `review` | Review | 3 |
| `capture` | Capture | 4 |

### Start-From Logic

1. Read `current_phase` and `checkpoints` from `state.json`
2. Skip any phase already marked complete in checkpoints (e.g., `planning_complete` timestamp exists → skip Plan)
3. Begin at the current phase, resuming from its checkpoint (e.g., `last_task_completed: "1.2"` → start at task 1.3)
4. Chain through all remaining phases to completion

### Mid-Phase Resumption

- Read `.mycelium/progress.md` for completed work summary
- Check plan markers (`[x]` = done, `[~]` = in progress, `[ ]` = pending)
- Resume the `[~]` or next `[ ]` task within the current phase
- Verify test baseline passes before continuing work

---

## Execution Modes

### Autonomous Mode (Default)

**Characteristics**:
- Minimal user interaction
- Auto-proceeds when path is clear
- Only stops for critical decisions
- Fast execution

**When to stop**:
- Ambiguous requirements (multiple valid interpretations)
- High-risk changes (security, payments, data)
- P1 issues in review
- Tests failing after 3 attempts
- Missing information that cannot be inferred

**When to auto-proceed**:
- Requirements are clear and specific
- Plan has no blockers or dependencies
- All tests pass
- Only P2/P3 review issues (note them, continue)
- Patterns exist in codebase

### Interactive Mode (`--interactive`)

**Characteristics**:
- Approval required at each phase
- User oversight throughout
- Slower but more controlled
- Good for learning or high-stakes changes

**When to ask**:
- After plan creation: "Approve plan?"
- After implementation: "Proceed to review?"
- After review: "Fix issues or accept?"
- After capture: "Learnings captured, continue?"

---

## Phase 0: Context Loading

**Invoke mycelium-context-load:**

```javascript
// Set full workflow mode
state.invocation_mode = "full"
state.current_phase = "context_loading"

// Start workflow with Phase 0
invoke("mycelium-context-load")
```

**What Phase 0 does:**
- Loads project context (product.md, tech-stack.md, workflow.md, CLAUDE.md)
- Loads institutional knowledge (solutions/, learned/)
- Discovers ALL capabilities (skills, agents, MCPs)
- Caches everything in state.json `discovered_capabilities`
- Chains to Phase 1 (Clarify Request)

**Output:**
- Context loaded and available
- Capabilities cached for use by planning phase
- Workflow continues automatically to Phase 1

---

## Phase 1: Clarify Request

**Invoked automatically by Phase 0 (via chain).**

**What Phase 1 does** (see mycelium-clarify):
- Asks clarifying questions ONE at a time
- Uses cached capabilities from Phase 0
- Determines if Phase 1.5 (Research) needed
- Chains to Phase 1.5 or Phase 2

**Decision gate** (Autonomous):
```
Requirements clear? (yes/no)
- If yes: Auto-proceed to Phase 2 (Planning)
- If no: Ask clarifying questions

Example:
  "Add user authentication" is AMBIGUOUS
  - Which auth method? (OAuth, JWT, session)
  - Registration included?
  - Password requirements?

  → STOP and ask
```

**Decision gate** (Interactive):
```
Questions answered?

[Show clarified requirements]

Approve? (yes/no/modify)
```

---

## Phase 1.5: Research (Optional)

**Invoked by Phase 1 if needed.**

**What Phase 1.5 does** (within mycelium-clarify):
- WebSearch for unfamiliar technology
- WebFetch for specific documentation
- Summarizes findings
- Chains to Phase 2 (Planning)

---

## Phase 2: Planning & Assignment

**Invoked automatically by Phase 1 (via chain).**

**What Phase 2 does** (see mycelium-plan):
1. Load cached capabilities from state.json
2. Decompose request into tasks with dependencies
3. Assign agent/skills/model to each task
4. Verify all assignments exist in cache
5. Create plan file
6. Chain to Phase 3 (Implementation)

**Decision gate** (Autonomous):
```
Requirements clear? (yes/no)
- If yes: Auto-proceed
- If no: Ask clarifying questions

Example:
  "Add user authentication" is AMBIGUOUS
  - Which auth method? (OAuth, JWT, session)
  - Registration included?
  - Password requirements?

  → STOP and ask
```

**Decision gate** (Interactive):
```
Plan created: 8 tasks across 3 phases

[Show task breakdown]

Approve plan? (yes/no/modify)
```

**Output**:
- Plan saved to `.mycelium/plans/YYYY-MM-DD-{track_id}.md`
- Track ID stored in state.json
- Plan registered in `session_state.plans[]` (auto-pauses any previously active plan)
- Chains to Phase 3 (Implementation)

**Error handling**:
- If planning fails: Report error, STOP
- If user rejects plan: Iterate or exit
- If requirements unclear: Ask questions, don't assume

---

## Phase 3: Implementation

**Invoked automatically by Phase 2 (via chain).**

### Pre-flight Checks

**Verify baseline**:
```bash
# Run existing tests
{test_command}

# Must all pass before new work
if [ $? -ne 0 ]; then
  echo "Baseline tests failing"
  echo "Fix existing tests before new work"
  STOP
fi
```

**Check uncommitted changes**:
```bash
git status --porcelain

# If dirty working tree:
# Autonomous: Stash changes
# Interactive: Ask "Stash changes? (yes/no)"
```

### Invoke TDD and Verification Skills

**Load**:
- `tdd` (mandatory)
- `verification` (mandatory)

**Execute implementation**:

For EACH task in plan (or parallel if dependencies allow):

1. **Mark task in-progress**:
   ```markdown
   [~] Task 1.1: Setup auth module
   ```

2. **Search for patterns**:
   ```bash
   # Find similar implementations
   grep -r "auth\|authentication" src/

   # Check for learned patterns
   cat .mycelium/solutions/patterns/critical-patterns.md
   ```

3. **Apply TDD cycle** (from tdd skill):
   ```
   RED → GREEN → REFACTOR

   1. Write failing test
   2. Verify RED (test fails)
   3. Write minimal implementation
   4. Verify GREEN (test passes)
   5. Refactor if needed
   ```

4. **Verify with evidence**:
   ```bash
   # Run tests
   {test_command}

   # Show output
   # Verify exit code = 0
   # Check coverage
   ```

5. **Commit**:
   ```bash
   git add {files}
   git commit -m "Task 1.1: Setup auth module

   - Add User model with password hashing
   - Add JWT token generation

   Tests: All pass (32/32)
   Coverage: 85%

   Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
   ```

6. **Mark task complete**:
   ```markdown
   [x] Task 1.1: Setup auth module `abc1234`
   ```

### Decision Gates

**Autonomous mode**:
- Tests pass → Auto-proceed
- Tests fail (attempt 1-3) → Debug and fix
- Tests fail (attempt 4+) → STOP, report blocker
- Pattern exists → Follow pattern
- No pattern → Create new, document

**Interactive mode**:
```
Task 1.1 complete (tests pass, coverage 85%)

Next: Task 1.2 (Add login endpoint)

Continue? (yes/no/pause)
```

### Parallel Execution

If tasks have no dependencies (`blockedBy: []`):

```
Tasks ready for parallel execution:
- Task 2.1: Add login endpoint
- Task 2.2: Add logout endpoint
- Task 2.3: Add token refresh

Creating worktrees...
├── .worktrees/auth_20260204_2.1/
├── .worktrees/auth_20260204_2.2/
└── .worktrees/auth_20260204_2.3/

Executing in parallel...
```

Use Task tool to dispatch subagents for each task.

**Merge strategy**:
1. Wait for task completion
2. Verify tests in worktree
3. Merge to main
4. Run full test suite
5. If conflicts: Resolve, retest
6. Remove worktree
7. Unblock dependent tasks

### Progress Reporting

**Autonomous mode**:
```
Working...
   Task 1.1: Auth module (tests pass)
   Task 1.2: Login endpoint (tests pass)
   Task 1.3: Session management (in progress)
```

**Interactive mode**:
```
Tasks 1.1 - 1.3 complete (3/8)

Progress: 37%

Continue to Task 1.4? (yes/no/status)
```

### Error Recovery

**Test failures**:
```
Attempt 1: Fix attempt #1
Attempt 2: Fix attempt #2
Attempt 3: Fix attempt #3
Attempt 4: STOP

Tests still failing after 3 fix attempts

Issue: {description}
Last error: {error message}

Blocker detected. Stopping for user input.

Options:
1. Debug manually
2. Simplify approach
3. Skip this task (not recommended)
```

**Blocked on external dependency**:
```
Task 2.3 blocked: Waiting for API key from user

Cannot proceed automatically.
Please provide API key and resume.
```

---

## Phase 4.5: Verification

**Invoked automatically within Phase 3 (mycelium-work).**

**What Phase 4.5 does** (verification skill, loaded by mycelium-work):
- Run all tests (show actual output)
- Check coverage ≥80%
- Run linters
- Verify build
- Evidence-based validation (NO "should work")

---

## Phase 4.5B: Context Sync

**Auto-invoked when context >80% (within mycelium-work).**

**What Phase 4.5B does** (context skill):
- Summarize work ≤500 tokens
- Update progress.md
- Spawn fresh agent with compressed context
- Continue work without context bloat

---

## Phase 5: Review

**Invoked automatically after Phase 3 completes.**

### Invoke Review Skill

**Load**: `mycelium-review` skill

**Execute two-stage review**:

#### Stage 1: Spec Compliance

```
Reviewing (Stage 1: Spec Compliance)...
   All tasks completed
   Acceptance criteria met
   Tests passing
   Coverage: 85% (target: 80%)
```

**Decision gate**:
- PASS → Proceed to Stage 2
- CONDITIONAL PASS → Note issues, proceed to Stage 2
- FAIL → STOP, report required fixes

#### Stage 2: Quality Review

```
Reviewing (Stage 2: Quality)...
   Security: 0 P1, 1 P2 issues
   Performance: 0 P1, 2 P2 issues
   Architecture: 0 P1, 0 P2 issues
   Code Quality: 0 P1, 1 P2 issues
   Simplicity: 0 P1, 3 P3 issues
```

**Issue summary**:
```
Quality Score: 8.5/10

Issues:
- 0 critical (P1)
- 4 important (P2)
- 3 minor (P3)
```

### Decision Gates

**Autonomous mode**:

```
P1 issues? (yes/no)
- Yes → STOP, must fix
- No → Auto-proceed

Example:
  0 P1, 4 P2, 3 P3 → Continue
  (P2/P3 noted for future improvement)
```

**Interactive mode**:
```
Review complete: 0 P1, 4 P2, 3 P3

Quality: 8.5/10

P2 Issues:
1. Weak password requirements
2. N+1 query in user list
3. Missing rate limiting
4. Error messages expose internal details

Fix now or accept? (fix/accept/selective)
```

### Fixing Issues

If P1 issues or user chooses to fix:

**Create fix tasks**:
```markdown
### Fix: SQL Injection in login

**Priority:** P1
**File:** src/auth/login.ts:45
**Fix:** Use parameterized queries
**Effort:** 15 min
```

**Execute fixes**:
- Follow TDD cycle
- Verify fix doesn't break existing tests
- Re-run affected review agents
- Confirm issue resolved
- Chain to Phase 6 (Finalization)

---

## Phase 6: Finalization

**Invoked automatically after Phase 5 completes.**

### Invoke Finalize Skill

**Load**: `mycelium-finalize` skill

**What Phase 6 does:**
1. Create git commit with Co-Author
2. Create pull request (via gh/glab/tea or manual)
3. Chain to Phase 6E (Pattern Detection)

**Output:**
- Commit SHA saved to state.json
- PR URL saved to state.json
- Chains automatically to Phase 6E

---

## Phase 6E: Pattern Detection

**Invoked automatically after Phase 6 completes.**

### Invoke Patterns Skill

**Load**: `mycelium-patterns` skill

**What Phase 6E does:**
1. Scan `.mycelium/solutions/**/*` for similar solutions
2. Identify patterns with 3+ occurrences
3. Update `critical-patterns.md` with new patterns
4. Recommend skill generation for 5+ occurrences
5. Chain to Phase 6F (Store Knowledge)

**Output:**
- Patterns detected and documented
- Skill candidates recommended
- Chains automatically to Phase 6F

---

## Phase 6F: Store Learned Knowledge

**Invoked automatically after Phase 6E completes.**

### Invoke Capture Skill

**Load**: `mycelium-capture` skill

**What Phase 6F does:**
1. Store solutions to `.mycelium/solutions/{category}/`
2. Capture decisions to `.mycelium/learned/decisions/`
3. Capture conventions to `.mycelium/learned/conventions/`
4. Update preferences in `.mycelium/learned/preferences.yaml`
5. Track anti-patterns in `.mycelium/learned/anti-patterns/`
6. Record effective prompts in `.mycelium/learned/effective-prompts/`
7. Mark workflow complete (`current_phase: "completed"`, `workflow_complete: true`)

**Important:** This phase does NOT handle:
- ❌ Commit creation (done in Phase 6)
- ❌ PR creation (done in Phase 6)
- ❌ Pattern detection (done in Phase 6E)

**Output:**
- Knowledge stored in appropriate locations
- Workflow marked complete
- No further phase chaining (workflow ends)

**Output example:**
```
✅ Knowledge captured:
   • 3 solutions stored
   • 2 decisions documented
   • 1 convention captured

Workflow complete!
```

---

## Final Report

### Generate Summary

```
Workflow Complete!

Task: Add user authentication with JWT

Phase 0: Context Loading
   ✓ Project context loaded
   ✓ 15 solutions discovered
   ✓ 13 skills, 7 agents cached

Phase 1: Clarify Request
   ✓ Requirements clarified
   ✓ No external research needed

Phase 2: Planning & Assignment
   ✓ Created 8 tasks across 3 phases
   ✓ Capabilities assigned (agent/skills/model per task)
   ✓ TDD test strategy defined

Phase 3: Implementation
   ✓ 8/8 tasks completed
   ✓ All tests passing (45 tests)
   ✓ Coverage: 85% (target: 80%)

Phases 4.5 & 4.5B: Verification & Context Sync
   ✓ Tests verified with evidence
   ✓ Context managed (no bloat)

Phase 5: Review
   ✓ Spec compliance: PASS
   ✓ Quality score: 8.5/10
   ✓ 0 P1, 4 P2, 3 P3 issues

Phase 6: Finalization
   ✓ Commit: abc1234
   ✓ PR: #123 created

Phase 6E: Pattern Detection
   ✓ 2 patterns detected
   ✓ critical-patterns.md updated

Phase 6F: Store Knowledge
   ✓ 3 solutions stored
   ✓ 2 decisions documented
   ✓ Knowledge base updated

Next: Deploy or continue with next feature?
```

### Next Steps

**Suggest**:
- Merge to main (if on feature branch)
- Create PR (if using pull requests)
- Deploy (if ready)
- Start next task: `/mycelium-go [next-task]`

---

## State Management

Throughout execution, maintain state in `state.json`:

```json
{
  "current_track": "auth_20260204",
  "current_phase": "implementation",
  "current_task": "1.3",
  "mode": "autonomous",
  "plans": [
    {
      "track_id": "auth_20260204",
      "plan_file": "2026-02-04-auth_20260204.md",
      "status": "in_progress",
      "created": "2026-02-04T10:15:00Z",
      "total_tasks": 8,
      "completed_tasks": 2
    }
  ],
  "checkpoints": {
    "planning_complete": "2026-02-04T10:15:00Z",
    "implementation_started": "2026-02-04T10:20:00Z",
    "last_task_completed": "1.2",
    "last_commit": "abc1234"
  },
  "blockers": [],
  "decisions_made": [
    {
      "decision": "Use JWT for authentication",
      "rationale": "Stateless, scalable",
      "timestamp": "2026-02-04T10:12:00Z"
    }
  ]
}
```

**Save state**:
- After each phase completion (update `plans[]` entry with task progress)
- After each task completion
- Before asking user questions
- On error/blocker

**Resume support**:
If interrupted, can resume from last checkpoint using `/mycelium-continue`. Use `/mycelium-continue --full` to force full workflow mode regardless of how the workflow was originally started.

---

## Decision Matrix

### When to STOP and Ask (Both Modes)

| Scenario | Action | Example |
|----------|--------|---------|
| Ambiguous requirements | ASK | "Add auth" → Which method? |
| High-risk change | ASK | Changing payment logic |
| P1 review issues | STOP | SQL injection found |
| Tests fail 3+ times | STOP | Cannot fix automatically |
| Missing credentials | ASK | Need API key |
| Architectural decision | ASK | Monolith vs microservices |

### When to AUTO-PROCEED (Autonomous Only)

| Scenario | Action | Example |
|----------|--------|---------|
| Clear requirements | GO | "Add JWT auth with bcrypt" |
| Pattern exists | GO | Found similar auth in codebase |
| Tests passing | GO | All green, proceed |
| P2/P3 issues only | GO | Note issues, continue |
| Standard task | GO | CRUD operations |

---

## Error Handling

**Planning fails**:
```
Planning failed: {reason}

Cannot proceed without plan.
Stopping.

Suggested: Clarify requirements and retry
```

**Implementation blocked**:
```
Implementation blocked: {reason}

Completed: 5/8 tasks
Blocked on: Task 1.6 (missing API key)

Saving state...
Resume with: /mycelium-continue
```

**Review finds P1 issues**:
```
Review FAILED: 2 critical (P1) issues

P1 Issues:
1. SQL injection vulnerability
2. Hardcoded credentials

These MUST be fixed before merge.

Stopping for fixes.
```

**Knowledge capture fails**:
```
Knowledge capture failed: {reason}

Work is complete but learnings not captured.

Options:
1. Retry capture: /mycelium-capture
2. Continue anyway (learnings lost)
```

---

## Performance Optimization

**Parallel execution**:
- Use worktrees for independent tasks
- Dispatch multiple subagents
- Merge results as they complete

**Smart caching**:
- Load context once, reuse
- Cache grep results
- Reuse test runs if code unchanged

**Progress streaming**:
- Update UI incrementally
- Don't wait for phase completion
- Show real-time progress

---

## Skills Used

- **mycelium-plan**: Requirements → tasks
- **tdd**: RED → GREEN → REFACTOR enforcement
- **verification**: Evidence-based validation
- **mycelium-review**: Two-stage quality check
- **mycelium-capture**: Knowledge extraction

## Quick Examples

```bash
# Feature development
/mycelium-go "Add user authentication"

# Debugging
/mycelium-go "Fix memory leak in session handler"

# Investigation / Question
/mycelium-go "Why does the API return 500 on concurrent requests?"

# Optimization
/mycelium-go "Optimize database queries" --interactive

# Detailed description
/mycelium-go "Add pagination to user list API with page size limits"
```

## Mode Comparison

**Autonomous** (default):
```
Planning... done
Working... done (5 tasks, parallel)
Reviewing... done (0 P1, 2 P2, 3 P3)
Capturing... done
Complete!
```

**Interactive** (`--interactive`):
```
Plan created
Approve? → yes
Tasks complete
Review? → yes
2 P2 issues
Fix now? → accept
Learnings captured
Complete!
```

## Success Criteria

Workflow is successful when:
- [ ] Plan created and approved
- [ ] All tasks completed with passing tests
- [ ] Coverage meets target
- [ ] No P1 issues in review
- [ ] Learnings captured
- [ ] State saved for resume capability
- [ ] User informed of completion and next steps

## Important

- **Saves state frequently** - Enable resume on interruption
- **Evidence-based** - All decisions verified with actual output
- **TDD mandatory** - No code without tests first
- **Stops on P1** - Critical issues block completion
- **Captures knowledge** - Builds compounding intelligence
- **Respect user preferences** - Check workflow.md for policies
- **Be transparent** - Show what's happening
- **Fail gracefully** - Clear error messages, suggest fixes
- **Resume with `/mycelium-continue`** - If interrupted, `/mycelium-continue` resumes all remaining phases automatically (since `/mycelium-go` sets `invocation_mode: "full"`)
- **`--full` flag** - When resuming a single-phase skill (e.g., `/mycelium-work`), use `/mycelium-continue --full` to run all remaining phases instead of just finishing the current one

## References

- [`.mycelium/` directory structure][mycelium-dir]
- [Session state docs][session-state-docs]
- [Session state schema][session-state-schema]
- [Plan frontmatter schema][plan-schema]
- [Solution frontmatter schema][solution-schema]
- [Progress state schema][progress-schema]
- [Metrics schema][metrics-schema]
- [Enum definitions][enums]

[mycelium-dir]: ../../docs/mycelium-directory.md
[session-state-docs]: ../../docs/session-state.md
[session-state-schema]: ../../schemas/session-state.schema.json
[plan-schema]: ../../schemas/plan-frontmatter.schema.json
[solution-schema]: ../../schemas/solution-frontmatter.schema.json
[progress-schema]: ../../schemas/progress-state.schema.json
[metrics-schema]: ../../schemas/metrics.schema.json
[enums]: ../../schemas/enums.json

## Examples

### Example 1: Building a New Feature
**User request:** "build user authentication with JWT"

**Workflow:**
1. **Planning** - Clarifies: OAuth vs JWT (user chose JWT), registration included, password requirements
2. **Creates plan** - 8 tasks across 3 phases with TDD breakdown
3. **Implementation** - Executes tasks with RED→GREEN→REFACTOR cycle
4. **Review** - Two-stage review finds 0 P1, 2 P2 issues (noted for later)
5. **Capture** - Extracts JWT pattern, saves to solutions library

**Result:** Feature implemented, tested (85% coverage), reviewed, and knowledge captured in 45 minutes.

### Example 2: Debugging an Issue
**User request:** "fix memory leak in session handler"

**Workflow:**
1. **Investigation** - Profiles memory usage, identifies leak source
2. **Planning** - Creates focused plan: reproduce, fix, verify, prevent
3. **Implementation** - Adds test reproducing leak, fixes issue, verifies
4. **Review** - Confirms fix, checks for similar patterns
5. **Capture** - Documents leak pattern and prevention in anti-patterns

**Result:** Memory leak fixed, test added, pattern documented for future prevention.

### Example 3: Technical Question
**User request:** "Why does the API return 500 on concurrent requests?"

**Workflow:**
1. **Investigation** - Analyzes logs, identifies race condition
2. **Explanation** - Provides root cause analysis  
3. **Solution plan** - Proposes locking strategy
4. **Implementation** - User approves, implements fix
5. **Capture** - Documents concurrency pattern

**Result:** Question answered with actionable solution and implementation completed.

## Troubleshooting

**Error:** "Planning failed: Requirements unclear"
**Cause:** Ambiguous request with multiple valid interpretations
**Solution:** mycelium-go stops to ask clarifying questions. Answer them to proceed.

**Error:** "Tests failing after 3 attempts"
**Cause:** Implementation approach has fundamental issue
**Solution:** Check error logs in progress.md. Consider simplifying approach or using `/recovery` for systematic debugging.

**Error:** "Review FAILED: 2 critical (P1) issues"
**Cause:** Security vulnerabilities or critical bugs found
**Solution:** P1 issues must be fixed before merge. Review findings, implement fixes, re-run review.

**Error:** "No .mycelium directory found"
**Cause:** Project not initialized
**Solution:** Run `/mycelium-setup` first to bootstrap project structure.

**Blocked:** "Task 2.3 blocked: Waiting for API key"
**Cause:** External dependency missing
**Solution:** Provide required information, then run `/mycelium-continue` to resume.

# Lib Utilities Refactoring Plan

## Current State: 23 utilities

## Recommended State: 13 utilities + lean versions

---

## Phase 1: Remove Redundant Learning Utilities

**DELETE these 9 files** (functionality moved to learning-agent):

```bash
rm lib/learning/pattern-analyzer.js      # → learning-agent analyzes
rm lib/learning/skill-generator.js        # → /workflow:create-skill
rm lib/learning/project-plugin-manager.js # → /workflow:create-skill
rm lib/learning/skill-validator.js        # → skill-reviewer agent
rm lib/learning/decision-capture.js       # → learning-agent
rm lib/learning/convention-detector.js    # → learning-agent
rm lib/learning/preference-learner.js     # → learning-agent
rm lib/learning/anti-pattern-tracker.js   # → learning-agent
rm lib/learning/prompt-evaluator.js       # → learning-agent

rmdir lib/learning  # Empty directory
```

**Update learning-agent.md** to explicitly list these responsibilities.

---

## Phase 2: Remove Agent Spawning

**DELETE:**
```bash
rm lib/agents/agent-spawner.js  # → Use Task tool natively
rmdir lib/agents  # Empty directory
```

**Update commands** to use Task tool directly:
```markdown
# In /workflow:work command:
Instead of: call lib/agents/agent-spawner.js
Use: Task tool with subagent_type
```

---

## Phase 3: Refactor Scheduler Utilities

**KEEP with modifications:**

### worktree-pool.js → worktree-tracker.js
```javascript
// BEFORE: Creation, removal, lifecycle management
// AFTER: State tracking only

module.exports = {
  // ✅ Keep: State tracking
  getActiveWorktrees() { ... },
  isWorktreeActive(path) { ... },
  getWorktreeStatus(path) { ... },

  // ❌ Remove: Let commands handle operations
  // createWorktree() - use /workflow:worktree-create
  // removeWorktree() - use /workflow:worktree-cleanup
  // setupWorktree() - use command
};
```

### merge-coordinator.js → merge-analyzer.js
```javascript
// BEFORE: Execute merges, handle conflicts
// AFTER: Analyze conflicts, provide recommendations

module.exports = {
  // ✅ Keep: Analysis
  detectConflicts(branch1, branch2) { ... },
  analyzeConflictComplexity(conflicts) { ... },
  suggestMergeStrategy(analysis) { ... },

  // ❌ Remove: Let Claude Code execute
  // executeMerge() - use Bash tool
  // resolveConflict() - Claude resolves
  // applyMergeStrategy() - Claude applies
};
```

---

## Phase 4: Keep Core Infrastructure (13 utilities)

**NO CHANGES needed for:**

### Core (4)
- `schema-validator.js`
- `state-manager.js`
- `template-renderer.js`
- `pattern-detector.js`

### Discovery (5)
- `capability-scanner.js`
- `agent-discovery.js`
- `skill-discovery.js`
- `mcp-discovery.js`
- `cache-manager.js`

### Scheduler (4)
- `task-scheduler.js`
- `dependency-graph.js`
- `worktree-tracker.js` (renamed from worktree-pool.js)
- `merge-analyzer.js` (renamed from merge-coordinator.js)

---

## Final Structure

```
lib/
├── schema-validator.js         ✅ Keep as-is
├── state-manager.js            ✅ Keep as-is
├── template-renderer.js        ✅ Keep as-is
├── pattern-detector.js         ✅ Keep as-is
│
├── discovery/                  ✅ Keep all (5 files)
│   ├── capability-scanner.js
│   ├── agent-discovery.js
│   ├── skill-discovery.js
│   ├── mcp-discovery.js
│   └── cache-manager.js
│
└── scheduler/                  ✅ Keep with renames (4 files)
    ├── task-scheduler.js
    ├── dependency-graph.js
    ├── worktree-tracker.js     # Renamed, slimmed down
    └── merge-analyzer.js        # Renamed, analysis only
```

**Total: 13 utilities** (down from 23)

---

## Migration Guide

### For Learning Operations

**Before:**
```javascript
const skillGen = require('./lib/learning/skill-generator');
await skillGen.generateSkill(pattern);
```

**After:**
```markdown
# In /workflow:compound command:
Detect pattern → Recommend skill generation
User runs: /workflow:create-skill
learning-agent generates the skill
```

### For Agent Spawning

**Before:**
```javascript
const spawner = require('./lib/agents/agent-spawner');
await spawner.spawnAgent({
  type: 'general-purpose',
  task: taskDef
});
```

**After:**
```markdown
# In /workflow:work command:
Use Task tool directly:

Task({
  subagent_type: "general-purpose",
  description: "Execute task X",
  prompt: taskDef.description
})
```

### For Worktree Operations

**Before:**
```javascript
const pool = require('./lib/scheduler/worktree-pool');
await pool.createWorktree(path, branch);
await pool.setupWorktree(path);
```

**After:**
```markdown
# In /workflow:work command:
Use Bash tool:

git worktree add .worktrees/feature-name -b feature/name
cd .worktrees/feature-name
npm install  # Setup dependencies
```

---

## Benefits

### Reduced Complexity
- 23 utilities → 13 utilities
- 10 fewer files to maintain
- Clearer separation: infrastructure vs. orchestration

### Better Use of Claude Code
- Agent spawning: Native Task tool
- Learning operations: learning-agent (AI-powered)
- Worktree operations: Bash tool (standard git)
- Skill generation: /workflow:create-skill command

### Clearer Architecture
- **lib/**: Programmatic infrastructure only
- **agents/**: AI-powered autonomous work
- **commands/**: Orchestration and workflows
- **hooks/**: Event-driven automation

### Easier Maintenance
- Fewer JavaScript modules to debug
- Claude Code handles complexity
- Infrastructure stays simple
- Updates happen in commands/agents, not lib/

---

## Implementation Steps

1. **Review with user** - Confirm this plan
2. **Update agents** - Ensure learning-agent covers all functionality
3. **Update commands** - Replace lib calls with Task/Bash tools
4. **Delete 10 utilities** - Remove redundant files
5. **Rename 2 utilities** - worktree-tracker, merge-analyzer
6. **Update documentation** - README, design.md
7. **Test workflows** - Ensure nothing breaks
8. **Commit changes** - Document migration

---

## Questions to Resolve

1. Should we keep `pattern-detector.js` or move to learning-agent?
   - Recommendation: Keep (used by multiple components)

2. Should worktree-tracker.js exist or just use git commands?
   - Recommendation: Keep for state tracking (faster than parsing git)

3. Should merge-analyzer.js exist or let Claude analyze conflicts?
   - Recommendation: Keep (programmatic conflict detection is faster)

---

**Estimated Impact:**
- Files deleted: 10
- Files renamed: 2
- Files unchanged: 11
- Lines of code removed: ~3,000
- Complexity reduction: ~40%

---
name: mycelium-setup
description: Bootstraps new or existing projects with .mycelium/ workflow structure using progressive disclosure. Follows critical interactive rules (one question per turn, suggested answers, max 5 per section). Use when user says "setup mycelium", "initialize project", or when starting a new project. Git is optional but recommended for parallel features.
license: MIT
version: 0.10.0
allowed-tools: ["Skill", "Read", "Write", "Edit", "Bash", "Glob", "Grep", "AskUserQuestion"]
metadata:
  author: Jason Hsieh
  category: workflow
  tags: [setup, initialization, project-bootstrap, progressive-disclosure]
  documentation: https://github.com/jason-hchsieh/mycelium
---

# Workflow Setup

Bootstrap a new or existing project with the mycelium workflow structure.

## Your Task

### Setup Workflow

This workflow guides you through bootstrapping a project with the mycelium workflow structure, detecting project type, gathering configuration through interactive questions, creating the `.mycelium/` directory structure, and initializing git if needed.

---

## Phase 1: Bootstrap

### Step 0: Pre-flight Check

Before starting setup, check if `.mycelium/` already exists and assess its completeness.

#### If .mycelium/ does not exist
→ Continue to Step 1 (fresh setup)

#### If .mycelium/ exists - Check completeness

**Minimum required files:**
- `.mycelium/state.json`
- `.mycelium/context/product.md`
- `.mycelium/context/tech-stack.md`
- `.mycelium/context/workflow.md`
- `.mycelium/solutions/patterns/critical-patterns.md`

See [.mycelium/ directory structure][mycelium-dir] for complete specification.

**If all required files present (complete structure):**
```
✅ Mycelium structure already exists and is complete.

Structure:
  ✓ .mycelium/state.json
  ✓ .mycelium/context/product.md
  ✓ .mycelium/context/tech-stack.md
  ✓ .mycelium/context/workflow.md
  ✓ .mycelium/solutions/patterns/critical-patterns.md
  ✓ .mycelium/plans/
  ✓ .mycelium/solutions/

Next steps:
  • Review context files: .mycelium/context/
  • Start planning: /mycelium-plan [description]
  • Full workflow: /mycelium-go [description]
```
→ **EXIT** (setup already complete)

**If partially complete (missing required files):**
```
⚠️  Incomplete .mycelium/ structure found.

Present:
  ✓ .mycelium/state.json
  ✓ .mycelium/context/product.md

Missing:
  ✗ .mycelium/context/tech-stack.md
  ✗ .mycelium/context/workflow.md
  ✗ .mycelium/solutions/patterns/critical-patterns.md

Options:
  1. Complete the missing files (recommended)
  2. Start fresh (deletes existing .mycelium/)
  3. Exit (manual fix required)

Enter 1-3:
```

**Handle user choice:**
- **Option 1 (Complete)**: Continue to Step 4 (Create Directory Structure) to create only missing files
- **Option 2 (Start fresh)**: `rm -rf .mycelium/`, then continue to Step 1
- **Option 3 (Exit)**: Show guidance on how to manually fix, then EXIT

---

### Step 1: Check for Existing Setup

Check if [setup_state.json][setup-state-schema] exists and determine next action.

```
┌─────────────────────────────────────┐
│ Does setup_state.json exist?       │
└─────────┬───────────────────────────┘
          │
    ┌─────┴─────┐
    NO          YES
    │           │
    │     ┌─────┴──────────────────────┐
    │     │ Check status field         │
    │     └─────┬──────────────────────┘
    │           │
    │     ┌─────┴─────┐
    │     │           │
    │  "completed"  "in_progress"
    │     │           │
    │     │           │
┌───▼─────▼───┐   ┌──▼─────────────────┐
│ Path A:     │   │ Path C:            │
│ Start fresh │   │ Resume from        │
│ setup       │   │ checkpoint         │
│             │   │                    │
│ → Step 2    │   │ Show progress      │
│             │   │ → current_section  │
└─────────────┘   └────────────────────┘
                  │
  ┌───────────────┘
  │ Path B:
  │ Setup complete
  │
  │ Info: Already done
  │ → EXIT
  └────────────────
```

#### Path A: No setup_state.json
→ Fresh setup, continue to **Step 2**

#### Path B: setup_state.json exists with status="completed"
```
✅ Setup already complete!

Completed: {completed_at}
Files created: {count} files

Next steps:
  /mycelium-plan [description]
  /mycelium-go [description]
```
→ **EXIT**

#### Path C: setup_state.json exists with status="in_progress"
```
⏸️  Resuming interrupted setup...

Progress:
  ✓ {completed_sections}
  → {current_section} (question {current_question})

Last updated: {last_updated}
```

**Options:**
1. **Resume** - Continue from where you left off
2. **Restart** - Delete state and start fresh
3. **Exit** - Manual intervention needed

Ask user choice → handle accordingly:
- **Option 1 (Resume)**: Load answers, continue from `current_section` and `current_question`
- **Option 2 (Restart)**: Delete `.mycelium/setup_state.json`, continue to **Step 2**
- **Option 3 (Exit)**: Show current state details, **EXIT**

---

### Step 2: Detect Project Type

Analyze the current directory to determine if this is a new or existing project.

#### Greenfield Detection

**Indicators** (new project):
- Empty directory or only has README
- No package.json, go.mod, Cargo.toml, requirements.txt, pyproject.toml
- No src/, lib/, app/, pkg/ directories
- No existing source code

**Action**: Run full interactive setup with all questions

#### Brownfield Detection

**Indicators** (existing project):
- Has package.json, go.mod, requirements.txt, Cargo.toml, etc.
- Has existing source code in src/, lib/, app/, etc.
- Has test files
- Has existing git repository

**Action**:
- Extract context from existing files
- Pre-populate answers by analyzing:
  - package.json → project name, dependencies, scripts
  - README.md → description, purpose
  - Existing code → language, framework, patterns
  - .gitignore → deployment target hints
- Ask user to confirm/correct detected values
- Only ask for information that cannot be inferred

---

### Step 3: Interactive Setup

**CRITICAL RULES:**
1. ✅ Ask **ONE** question per turn
2. ✅ Wait for user response before proceeding
3. ✅ Offer 2-4 suggested answers + "Type your own" option
4. ✅ Maximum 5 questions per section
5. ✅ Update [setup_state.json][setup-state-schema] after EACH successful answer
6. ✅ Validate file write succeeded before continuing

Ask questions ONE AT A TIME following this pattern:

#### PRODUCT Section (what & why)

**Question 1 of 4: Project Identity**
```
What is this project?

Suggested answers:
  1. TaskFlow - A workflow automation tool
  2. DataHub - A data integration platform
  3. CodeReview - An automated code review system
  4. Type your own answer

Enter 1-4:
```

**After user responds:**
1. If 1-3: Extract project name and description
2. If 4: Prompt for custom "ProjectName - Description"
3. Save to [setup_state.json][setup-state-schema]:
   ```json
   {
     "current_section": "product",
     "current_question": 2,
     "completed_sections": [],
     "answers": {
       "project_identity": "TaskFlow - A workflow automation tool"
     }
   }
   ```
4. Verify write succeeded: `const state = await readFile('setup_state.json')`
5. If write failed: Error "❌ Failed to save answer, cannot continue"
6. Continue to Question 2

**Question 2 of 4: Problem Statement**
```
What problem does this project solve?

Suggested answers:
  1. Manual processes waste developer time
  2. Lack of integration between tools
  3. Poor visibility into workflows
  4. Type your own answer

Enter 1-4:
```

**After user responds:** Save answer, update `current_question: 3`, verify, continue

**Question 3 of 4: Target Users**
```
Who are the target users?

Suggested answers:
  1. Software development teams (5-50 people)
  2. DevOps engineers
  3. Engineering managers
  4. Type your own answer

Enter 1-4:
```

**After user responds:** Save answer, update `current_question: 4`, verify, continue

**After Question 3 (Target Users) complete:**
Update state:
```json
{
  "current_section": "tech_stack",
  "current_question": 1,
  "completed_sections": ["product"]
}
```

**Note:** Goals are optional and can be added later to `.mycelium/context/product.md`

#### TECH STACK Section (with what)

**For brownfield projects:** Detect language/framework from files first, then confirm with user.

**Question 1 of 4: Primary Language**
```
What programming language?

Suggested answers:
  1. TypeScript
  2. Python
  3. Go
  4. Type your own (JavaScript, Rust, Java, etc.)

Enter 1-4:
```

**After user responds:** Save answer, update `current_question: 2`, verify, continue

**Question 2 of 4: Framework**
```
What framework or library?

Suggested answers:
  1. React + Next.js
  2. FastAPI (Python)
  3. Gin (Go)
  4. Type your own

Enter 1-4:
```

**After Question 2 (Framework) complete:**
Update state:
```json
{
  "current_section": "workflow",
  "current_question": 1,
  "completed_sections": ["product", "tech_stack"]
}
```

**Note:** Database and infrastructure are optional and can be added later to `.mycelium/context/tech-stack.md`

#### WORKFLOW Section (how to work)

**Question 1 of 1: TDD Strictness**
```
How strict should TDD enforcement be?

Suggested answers:
  1. strict - All code follows RED-GREEN-REFACTOR (recommended)
  2. flexible - TDD for new features, optional for fixes
  3. none - No TDD enforcement
  4. Type your own approach

Enter 1-4:
```

Default: `strict`

**After user responds:**
1. Save answer, update:
   ```json
   {
     "current_section": null,
     "current_question": null,
     "completed_sections": ["product", "tech_stack", "workflow"],
     "answers": {
       "tdd_strictness": "strict",
       "commit_style": "conventional",
       "review_required": "recommended",
       "coverage_target": 80
     }
   }
   ```
2. Automatically apply recommended defaults:
   - Commit style: `conventional` (feat/fix/chore format)
   - Code review: `recommended` (for major changes)
   - Coverage target: `80%`
3. Verify, continue to Step 4 (Create Directory Structure)

**Note:** Advanced settings (commit style, review requirements, coverage) use recommended defaults. Edit `.mycelium/context/workflow.md` to customize later.

---

### Step 4: Create Directory Structure

Once all answers collected, create this structure:

```
project/
├── CLAUDE.md                    # Quick context file
├── .mycelium/                   # AI workflow artifacts
│   ├── context/                 # Project information
│   │   ├── product.md
│   │   ├── tech-stack.md
│   │   └── workflow.md
│   ├── plans/                   # Living plan documents
│   ├── solutions/               # Captured learnings
│   │   ├── performance-issues/
│   │   ├── database-issues/
│   │   ├── security-issues/
│   │   ├── ui-bugs/
│   │   ├── integration-issues/
│   │   └── patterns/
│   │       └── critical-patterns.md
│   ├── state.json               # Session state
│   └── setup_state.json         # Setup progress
└── docs/                        # User documentation (if not exists)
```

See [.mycelium/ directory structure][mycelium-dir] for complete documentation of this structure and all file purposes.

#### File Templates

**CLAUDE.md**:
```markdown
# {project_name}

{product_description}

## Quick Context

**What**: {problem_statement}
**Who**: {target_users}
**Stack**: {primary_language} + {frameworks}

## Key Goals

{list_of_goals}

## Getting Started

See `.mycelium/context/` for detailed project information.

## Workflow

- `/mycelium-plan` - Plan new work
- `/mycelium-work` - Execute tasks
- `/mycelium-review` - Review code
- `/mycelium-capture` - Capture learnings
```

**.mycelium/context/product.md**:
```markdown
---
product_name: {name}
description: {description}
target_users: {users}
created: {timestamp}
---

# Product Context

## Problem Statement

{problem}

## Goals

{goals}

## Success Criteria

{how_we_measure_success}
```

**.mycelium/context/tech-stack.md**:
```markdown
---
primary_language: {language}
frameworks: [{frameworks}]
database: {database}
infrastructure: {infrastructure}
updated: {timestamp}
---

# Tech Stack

## Languages

{list}

## Frameworks & Libraries

{list}

## Database

{details}

## Infrastructure

{details}

## Development Tools

{linters, formatters, test frameworks}
```

**.mycelium/context/workflow.md**:
```markdown
---
tdd_strictness: {strict|flexible|none}
commit_style: {conventional|descriptive|atomic}
review_required: {required|recommended|optional}
coverage_target: {percentage}
---

# Workflow Configuration

## TDD Policy

{description_based_on_strictness}

## Commit Guidelines

{description_based_on_style}

## Code Review

{description_based_on_requirement}

## Quality Standards

- Test coverage: ≥{target}%
- No linting errors
- All tests passing
```

**.mycelium/solutions/patterns/critical-patterns.md**:
```markdown
# Critical Patterns

Patterns that MUST be followed in this codebase.

## When to Update This File

After discovering:
- Security-critical patterns
- Performance-critical patterns
- Data integrity patterns
- Architectural constraints

## Patterns

(Will be populated as patterns are discovered)
```

---

### Step 5: Initialize Git (OPTIONAL but RECOMMENDED)

Git is **strongly recommended** for the mycelium workflow but not required.

#### With Git Enabled

**Benefits:**
- ✅ Version control for all changes
- ✅ Parallel implementation via `git worktree`
- ✅ Automatic commit creation with Co-Author attribution
- ✅ Branch-based feature development
- ✅ Easy rollback of changes
- ✅ Track history of solutions and patterns

#### Without Git

**Limitations:**
- ❌ No worktree support (cannot run tasks in parallel)
- ❌ No automatic commits
- ❌ No version control or rollback
- ⚠️  Single-threaded implementation only
- ⚠️  Manual tracking of changes required

**Setup still works without git** - all core workflow features function, but parallelization and version control are unavailable.

#### Initialize Git

```bash
# Check if git repository exists
if [ ! -d .git ]; then
  echo "Git repository not found."
  echo ""
  echo "Initialize git now? (recommended)"
  echo "  • Enables parallel implementation"
  echo "  • Automatic version control"
  echo "  • Easier rollback"
  echo ""
  # Ask user (use AskUserQuestion tool)
  # If yes:
  git init
  echo "✓ Initialized git repository"
  git checkout -b main
  # Save to setup_state.json: git_enabled: true
  # If no:
  echo "⚠️  Continuing without git (parallel features disabled)"
  # Save to setup_state.json: git_enabled: false
else
  echo "✓ Git repository found"
  # Verify main/master branch exists
  git branch --show-current || git checkout -b main
  # Save to setup_state.json: git_enabled: true
fi

# If git_enabled, create/update .gitignore
```

#### Create/Update .gitignore (if git_enabled)

Use language-specific templates from `templates/gitignore/`:

**Detect language and load appropriate template:**
```bash
# Node.js/TypeScript
if [ -f package.json ]; then
  cat templates/gitignore/node.gitignore >> .gitignore
fi

# Python
if [ -f requirements.txt ] || [ -f pyproject.toml ]; then
  cat templates/gitignore/python.gitignore >> .gitignore
fi

# Go
if [ -f go.mod ]; then
  cat templates/gitignore/go.gitignore >> .gitignore
fi

# Rust
if [ -f Cargo.toml ]; then
  cat templates/gitignore/rust.gitignore >> .gitignore
fi
```

**Always add mycelium-specific entries:**
```gitignore
# Mycelium
.worktrees/
.mycelium/state.json
```

**Available templates:**
- [Node.js gitignore][node-gitignore]
- [Python gitignore][python-gitignore]
- [Go gitignore][go-gitignore]
- [Rust gitignore][rust-gitignore]

#### Create Initial Commit (if git_enabled)

```bash
git add .mycelium/ CLAUDE.md .gitignore
git commit -m "Initialize project with mycelium workflow

- Add .mycelium/ directory structure
- Add CLAUDE.md quick context
- Configure git and .gitignore

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

### Step 6: Import Existing Configs (Brownfield Only)

If project uses linters/formatters:

**Detect**:
- .eslintrc.* → ESLint
- .prettierrc.* → Prettier
- pyproject.toml → Black, Ruff
- .golangci.yml → golangci-lint
- rustfmt.toml → rustfmt

**Action**:
- Note their locations in tech-stack.md
- Do NOT create new configs
- Do NOT modify existing configs
- Just document what exists

**Example addition to tech-stack.md**:
```markdown
## Code Quality Tools

- ESLint: `.eslintrc.json` (existing)
- Prettier: `.prettierrc` (existing)
- TypeScript: `tsconfig.json` (existing)
```

---

### Step 7: Finalize Setup

**Update [setup_state.json][setup-state-schema]**:
```json
{
  "status": "completed",
  "project_type": "greenfield",
  "completed_at": "2026-02-04T10:30:00Z",
  "files_created": [
    "CLAUDE.md",
    ".mycelium/context/product.md",
    ".mycelium/context/tech-stack.md",
    ".mycelium/context/workflow.md",
    ".mycelium/solutions/patterns/critical-patterns.md",
    ".gitignore"
  ]
}
```

**Initialize state.json**:
```json
{
  "project_name": "{name}",
  "initialized_at": "2026-02-04T10:30:00Z",
  "current_track": null,
  "discovered_capabilities": {
    "skills": [],
    "agents": [],
    "patterns": []
  },
  "active_worktrees": [],
  "stats": {
    "tracks_completed": 0,
    "solutions_captured": 0,
    "patterns_discovered": 0
  }
}
```

**Output summary**:
```
✅ Mycelium setup complete!

Created:
  ✓ CLAUDE.md - Quick context
  ✓ .mycelium/context/ - Project information
  ✓ .mycelium/plans/ - Living plans
  ✓ .mycelium/solutions/ - Knowledge base
  ✓ Git repository initialized

Configuration:
  • Language: {language}
  • TDD: {strictness}
  • Coverage: ≥{target}%

Next steps:
  1. Review CLAUDE.md and .mycelium/context/
  2. When you have a task: /mycelium-plan [description]
  3. Or for full automation: /mycelium-go [description]
```

---

## State Management

The [setup_state.json][setup-state-schema] enables resume functionality by tracking setup progress.

**State structure:**

```json
{
  "status": "in_progress",
  "project_type": "greenfield|brownfield",
  "current_section": "product|tech_stack|workflow",
  "current_question": 2,
  "completed_sections": ["product"],
  "answers": {
    "product_name": "MyApp",
    "product_description": "...",
    "problem_statement": "...",
    "target_users": "..."
  },
  "files_created": [".mycelium/context/product.md"],
  "started_at": "2026-02-04T09:00:00Z",
  "last_updated": "2026-02-04T09:15:00Z"
}
```

**Save after EACH answer** to enable interruption and resume.

---

## Resume Support

When resuming (state shows "in_progress"):

1. Automatically detect in-progress setup from [setup_state.json][setup-state-schema]
2. Show progress:
   ```
   Resuming setup...

   Completed:
     ✓ Product section (4 questions)

   In progress:
     • Tech Stack section (question 2 of 4)

   Continuing...
   ```
3. Jump to current_section and current_question
4. Do NOT re-ask answered questions
5. Continue until completion

---

## Error Handling

**Git init fails**:
- Show error message
- Offer to continue without git (with feature limitations)
- If user declines git: set git_enabled=false, continue setup
- If user wants git: suggest resolving git issues, then re-run setup

**Directory not writable**:
- Check permissions
- Inform user about permission issues
- Suggest solutions

**User answers unclear**:
- Ask for clarification
- Do NOT assume or guess
- Provide examples of good answers

**Interrupted**:
- State is saved after each answer in [setup_state.json][setup-state-schema]
- Running `/mycelium-setup` again automatically resumes from checkpoint
- No data loss

---

## Greenfield vs Brownfield Differences

| Aspect | Greenfield | Brownfield |
|--------|-----------|------------|
| Questions | All 12 questions | Only undetectable info |
| Detection | Manual entry | Auto-detect from files |
| Confirmation | Not needed | Confirm detected values |
| Git | Create new repo | Use existing |
| Configs | None created | Import existing |

---

## Quick Example

```bash
/mycelium-setup  # Setup project (auto-resumes if interrupted)
```

## Important Notes

- **Git is recommended but optional** - Mycelium works without git, but parallel features and version control require it
- **One question at a time** - Interactive setup asks questions sequentially
- **State saved after each answer** - Automatically resumes if interrupted
- **Brownfield detection** - Auto-detects project info from existing files
- Ask ONE question at a time, not all at once
- Save state after EACH answer (enables resume)
- For brownfield: Show detected values, ask confirmation
- Be conversational but concise
- Do NOT proceed to planning - setup only creates structure
- Do NOT create application code - only workflow infrastructure
- Without git: no worktrees, no auto-commits, single-threaded only
- All file paths are absolute when saved to state

---

## Next Steps After Setup

1. **Review** created files (CLAUDE.md, .mycelium/context/)
2. **Customize** if needed (edit context files)
3. **Start work**:
   - Manual: `/mycelium-plan [task]`
   - Autonomous: `/mycelium-go [task]`

## Skills Used

- **context**: For extracting information from existing projects (brownfield)

## References

- [`.mycelium/` directory structure][mycelium-dir]
- [Session state docs][session-state-docs]
- [Session state schema][session-state-schema]
- [Setup state schema][setup-state-schema]
- [CLAUDE.md template][claude-template]
- [Product template][product-template]
- [Tech stack template][tech-stack-template]
- [Workflow template][workflow-template]
- [Critical patterns template][patterns-template]
- [Node.js gitignore][node-gitignore]
- [Python gitignore][python-gitignore]
- [Go gitignore][go-gitignore]
- [Rust gitignore][rust-gitignore]
- [Enum definitions][enums]

[mycelium-dir]: ../../docs/mycelium-directory.md
[session-state-docs]: ../../docs/session-state.md
[session-state-schema]: ../../schemas/session-state.schema.json
[setup-state-schema]: ../../schemas/setup-state.schema.json
[claude-template]: ../../templates/project/CLAUDE.md.template
[product-template]: ../../templates/project/product.md.template
[tech-stack-template]: ../../templates/project/tech-stack.md.template
[workflow-template]: ../../templates/project/workflow.md.template
[patterns-template]: ../../templates/project/critical-patterns.md.template
[node-gitignore]: ../../templates/gitignore/node.gitignore
[python-gitignore]: ../../templates/gitignore/python.gitignore
[go-gitignore]: ../../templates/gitignore/go.gitignore
[rust-gitignore]: ../../templates/gitignore/rust.gitignore
[enums]: ../../schemas/enums.json

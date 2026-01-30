# ORC Commands Reference

Complete reference for all ORC commands.

## Analysis Commands

### `/orc analyze`

Performs pre-flight codebase analysis to understand conventions, patterns, and project structure.

**Variants:**
- `/orc analyze` - Run analysis (uses cache if valid)
- `/orc analyze --force` - Force re-analysis, ignore cache
- `/orc analyze --focus <areas>` - Focus on specific areas

**Focus Areas:**
- `conventions` - Naming, formatting, file organization
- `dependencies` - Runtime and dev dependencies
- `patterns` - Data access, error handling, auth, validation
- `security` - Auth patterns, secrets, vulnerabilities
- `api` - REST/GraphQL style, response formats
- `testing` - Framework, style, mocking approach
- `infrastructure` - Docker, CI/CD, hosting

**Behavior:**
- Automatically runs before planning if profile missing or stale
- Caches results in `.orc/plan/codebase_profile.json`
- Cache expires after 24 hours or significant git changes

**Example:**
```
/orc analyze

🔍 Analyzing Codebase...

Project Type: typescript
Frameworks: express, prisma, react

📁 Structure: feature-based
📝 Conventions: kebab-case files, camelCase functions
🔧 Patterns: repository, zod validation, jwt auth
🧪 Testing: jest (describe-it style)
🔒 Security: helmet, cors configured

✅ Analysis Complete (247 files, 92% confidence)
```

---

## Planning Commands

### `/orc plan <goal>`

Creates a new plan or appends to an existing plan.

**Arguments:**
- `<goal>` - High-level description of what to build

**Behavior:**
- If no plan exists: Creates new plan with Epic→Feature→Story hierarchy
- If plan exists: Prompts to Append or Replace

**Example:**
```
/orc plan "Build REST API with JWT authentication and user management"
```

**Output:**
```
📋 Plan Created: Build REST API with JWT authentication

Epics (2):
  E1: User Authentication [3 features, 9 stories]
  E2: User Management [2 features, 6 stories]

Total: 5 features, 15 stories
Run /orc approve to proceed
```

---

### `/orc show`

Displays the current plan summary.

**Variants:**
- `/orc show` - Full plan overview
- `/orc show <epic-id>` - Detailed epic view (e.g., `/orc show E1`)
- `/orc show deviations` - List all deviations pending review

**Output includes:**
- Epic/feature/story counts and status
- Progress indicators
- Current execution position
- Confidence score
- Pending deviations

---

### `/orc approve`

Approves plan items for execution.

**Variants:**
- `/orc approve` - Approve all pending epics
- `/orc approve <epic-id>` - Approve specific epic
- `/orc approve deviation <id>` - Approve specific deviation
- `/orc approve deviations` - Approve all deviations

**Note:** Approval is **mandatory** before execution can begin.

---

## Execution Commands

### `/orc run`

Executes approved epics.

**Variants:**
- `/orc run` - Execute all approved epics
- `/orc run <epic-id>` - Execute specific epic

**Behavior:**
- Processes epics in priority order
- Executes independent stories in parallel
- Validates each story against acceptance criteria
- Auto-retries failed stories (up to 3 attempts)
- Saves checkpoints at feature/epic boundaries

**Output:**
Verbose progress showing:
- REASON: Planning and pattern matching
- ACT: File operations
- OBSERVE: Type/lint/import checks
- VERIFY: Test and acceptance criteria results

---

### `/orc next`

Executes only the next priority epic, then pauses.

**Use case:** Incremental execution with review between epics.

---

### `/orc stop`

Gracefully stops execution.

**Behavior:**
- Waits for current story to complete
- Saves checkpoint
- Does not abort mid-story

**Resume with:** `/orc resume`

---

### `/orc resume`

Resumes execution from the last checkpoint.

**Behavior:**
- Loads latest checkpoint
- Validates codebase state
- Continues from first incomplete story

---

### `/orc retry <story-id>`

Retries a blocked story.

**Arguments:**
- `<story-id>` - Story to retry (e.g., `E1-F2-S3`)

**Behavior:**
- Resets attempt counter
- Re-executes with fresh context
- Unblocks dependent stories if successful

**Use after:** Fixing external dependencies (database, API, etc.)

---

## Learning Commands

### `/orc patterns`

Displays learned patterns.

**Variants:**
- `/orc patterns` - All patterns
- `/orc patterns <category>` - By category (auth, api, database, testing)
- `/orc patterns search <query>` - Search patterns

**Shows:**
- Pattern ID, name, confidence
- Success/failure counts
- Code examples
- Anti-patterns

---

### `/orc learn`

Forces immediate pattern extraction.

**Behavior:**
- Analyzes completed stories
- Extracts new patterns
- Updates existing pattern confidence
- Records anti-patterns
- Generates embeddings

**Normally automatic** after feature completion.

---

## Utility Commands

### `/orc status`

Shows current ORC state.

**Output:**
- Session ID and phase
- Progress bars
- Health indicators
- Recent activity
- Blocked items and warnings

---

### `/orc clear`

Clears plan and state.

**Behavior:**
- Prompts for confirmation
- Deletes plan, checkpoints, state
- **Preserves** learned patterns

**Variant:**
- `/orc clear --all` - Also deletes learnings (requires typing "DELETE ALL")

---

## Command Quick Reference

| Command | Purpose |
|---------|---------|
| `/orc analyze` | Analyze codebase |
| `/orc analyze --force` | Force re-analysis |
| `/orc plan <goal>` | Create/append plan |
| `/orc show` | View plan |
| `/orc show E1` | View epic E1 |
| `/orc show deviations` | View deviations |
| `/orc approve` | Approve all |
| `/orc approve E1` | Approve E1 |
| `/orc run` | Execute all |
| `/orc run E1` | Execute E1 |
| `/orc next` | Execute next epic |
| `/orc stop` | Stop execution |
| `/orc resume` | Resume execution |
| `/orc retry E1-F2-S3` | Retry story |
| `/orc patterns` | View patterns |
| `/orc learn` | Extract patterns |
| `/orc status` | View status |
| `/orc clear` | Clear state |

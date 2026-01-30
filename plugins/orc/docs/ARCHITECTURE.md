# ORC Architecture

Technical architecture overview of the ORC system.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                            │
│        (State management, phase control, contract enforcement)  │
└─────────────────────────────────────────────────────────────────┘
                              │
    ┌─────────────────────────┼─────────────────────────┐
    ▼                         ▼                         ▼
┌─────────┐             ┌─────────┐               ┌─────────┐
│ANALYZER │             │ PLANNER │               │IMPLEMENT│
│         │────────────►│         │               │   ER    │
└────┬────┘  profile    └────┬────┘               └────┬────┘
     │                       │                         │
     ▼                       ▼                         ▼
┌─────────┐             ┌─────────┐               ┌─────────┐
│Architect│             │Architect│               │Validator│
│DevOps   │             │Product  │               │Security │
│Security │             │DevOps   │               │Database │
│Database │             │Biz Anlst│               │Frontend │
└─────────┘             │   ...   │               │   ...   │
(Specialists)           └─────────┘               └─────────┘
                        (Specialists)             (Specialists)
                                                       │
                              ┌─────────────────────────┘
                              ▼
                        ┌─────────┐               ┌─────────┐
                        │ REVIEWER│──────────────►│ LEARNER │
                        │         │               │         │
                        └─────────┘               └─────────┘
```

## Core Components

### 1. Orchestrator

The central coordination layer responsible for:

- **State Management**: Maintaining execution state across phases
- **Phase Control**: Managing transitions between plan/execute/review/learn
- **Contract Enforcement**: Validating all agent inputs/outputs against JSON schemas
- **Delegation**: Spawning agents and managing their lifecycle
- **Checkpoint Management**: Saving/restoring execution state

**Key Responsibilities:**
- Never implements code directly
- Validates all agent handoffs
- Manages parallel execution
- Handles interruption/recovery

### 2. Analyzer

Pre-flight codebase analysis for context-aware planning:

- **Project Detection**: Language, framework, platform identification
- **Convention Extraction**: Naming patterns, file organization, code style
- **Dependency Analysis**: Runtime and dev dependencies, package manager
- **Pattern Detection**: Data access, error handling, auth, validation
- **API Style Analysis**: REST/GraphQL, response formats, versioning
- **Test Pattern Detection**: Framework, style, mocking approach
- **Security Analysis**: Auth patterns, secrets management, vulnerabilities
- **Infrastructure Analysis**: Docker, CI/CD, hosting configuration

**Output**: Codebase profile that guides all planning decisions

### 3. Planner

Decomposes goals into structured plans:

- **Profile Loading**: Uses Analyzer's codebase profile
- **Goal Analysis**: Understanding user intent
- **Hierarchy Generation**: Creating Epic→Feature→Story structure
- **Pattern Matching**: Finding relevant patterns from learnings
- **Dependency Analysis**: Identifying parallelizable work
- **Convention Alignment**: Ensuring all plans follow existing codebase patterns

**Output**: Complete plan ready for approval, aligned with codebase

### 4. Implementer

Executes story implementations using ReAct pattern:

- **REASON**: Analyze requirements, load patterns and codebase profile, plan steps
- **ACT**: Create/modify files following existing conventions, implement features
- **OBSERVE**: Verify operations, check types/imports, validate convention adherence

**Key Features:**
- Strict approach adherence
- Codebase convention compliance
- Deviation documentation
- Specialist spawning

### 5. Validator

Verifies implementations against criteria:

- **Gate Checks**: Tests, types, lint, security
- **Acceptance Criteria**: Evidence collection
- **Compliance**: Approach, file, and convention verification
- **Convention Validation**: Naming, style, organization alignment

**All gates must pass** for story completion.

### 6. Reviewer

Final quality gate with Reflexion pattern:

- **Cross-Story Analysis**: Consistency checks
- **Deviation Analysis**: Impact assessment
- **Security Review**: Vulnerability scanning
- **Convention Audit**: Verify codebase alignment
- **Reflexion**: What worked, what didn't, improvements

### 7. Learner

Extracts and maintains patterns:

- **Pattern Extraction**: From successful stories
- **Anti-Pattern Detection**: From failures
- **Convention Learning**: Successful convention adherence patterns
- **Embedding Generation**: For semantic matching
- **Confidence Management**: Decay and updates

## Data Flow

### Analyze Phase
```
User Goal
    │
    ▼
Orchestrator
    │
    ├─→ Check cached profile
    │      │
    │      ├─→ Valid cache: Skip analysis
    │      │
    │      └─→ Stale/missing: Continue
    │
    ├─→ Analyzer
    │      │
    │      ├─→ [Specialists: Architect, DevOps, Security, Database]
    │      │
    │      └─→ Codebase Profile
    │
    ▼
.orc/plan/codebase_profile.json
```

### Plan Phase
```
User Goal + Codebase Profile
    │
    ▼
Orchestrator
    │
    ├─→ Planner (with profile context)
    │      │
    │      ├─→ [Specialists as needed]
    │      │
    │      └─→ Structured Plan (aligned with codebase)
    │
    ▼
.orc/plan/plan.json
.orc/plan/epics/E*.json
```

### Execute Phase
```
Approved Plan
    │
    ▼
Orchestrator
    │
    ├─→ Implementer (per story)
    │      │
    │      ├─→ [Specialists as needed]
    │      │
    │      └─→ Implementation Result
    │
    ├─→ Validator (per story)
    │      │
    │      └─→ Validation Result
    │
    ▼
Story Complete/Blocked
    │
    ▼
Feature Checkpoint
Epic Checkpoint
```

### Review Phase
```
Completed Epic
    │
    ▼
Orchestrator
    │
    ├─→ Reviewer
    │      │
    │      ├─→ [Security Specialist]
    │      │
    │      └─→ Review Result
    │
    ▼
Review Complete
User Review Items
```

### Learn Phase
```
Session Data
    │
    ▼
Orchestrator
    │
    ├─→ Learner
    │      │
    │      └─→ Extracted Patterns
    │
    ▼
.orc/plan/learnings.json
.orc/plan/embeddings.json
```

## State Management

### State File Structure
```json
{
  "session_id": "uuid",
  "goal": "string",
  "phase": "analyze|plan|execute|review|learn",
  "codebase_analyzed": true,
  "codebase_profile_at": "timestamp",
  "analysis_confidence": 0.92,
  "current_epic": "E1",
  "current_feature": "E1-F1",
  "current_story": "E1-F1-S2",
  "started_at": "timestamp",
  "last_activity": "timestamp",
  "epics_summary": {},
  "stories_summary": {},
  "deviations_pending_review": 0,
  "confidence": 0.95
}
```

### Checkpoint Strategy

Checkpoints created at:
1. **Epic Creation** - Plan captured
2. **Feature Completion** - All stories done
3. **Epic Completion** - All features done

Checkpoint contains:
- Full state snapshot
- Completed story results
- Deviation log
- Patterns used

### Recovery Protocol

On `/orc resume`:
1. Load latest checkpoint
2. Validate codebase matches
3. Identify incomplete work
4. Resume from next pending story

## Contract System

All agent communication uses typed JSON contracts:

### Schema Files
```
contracts/
├── _definitions.schema.json      # Shared definitions
├── codebase-profile.schema.json  # Pre-flight analysis output
├── epic.schema.json
├── feature.schema.json
├── story.schema.json
├── plan.schema.json
├── task-result.schema.json
├── deviation.schema.json
├── pattern.schema.json
└── learnings.schema.json
```

### Validation Flow
```
Agent Output
    │
    ▼
JSON Schema Validation
    │
    ├─→ Valid: Accept output
    │
    └─→ Invalid: Return error, request retry
```

## Pattern Matching

### Embedding-Based Matching
1. Story description → Embedding
2. Compare against pattern embeddings
3. Cosine similarity > 0.8 = match
4. Rank by confidence
5. Attach as `suggested_approach`

### Embedding Storage
```json
{
  "model": "text-embedding-3-small",
  "patterns": {
    "sp_bcrypt_001": [0.021, -0.034, ...]
  },
  "stories": {
    "E1-F1-S1": [0.033, -0.018, ...]
  }
}
```

## Parallel Execution

### Dependency Analysis
At plan time:
1. Build dependency graph
2. Topological sort
3. Identify parallel groups
4. Validate no conflicts

### Execution Groups
```
Feature with 4 stories:
  S1 (no deps)     ──────────►
  S2 (no deps)     ──────────►     (parallel)
  S3 (depends S1)       wait... ──────────►
  S4 (depends S2)       wait... ──────────►
```

## Error Handling

### Transient Errors
- Rate limits, network issues
- Retry with exponential backoff
- Max 3 retries

### Persistent Errors
- Invalid credentials, permissions
- Save checkpoint and exit
- Clear error message

### Task Failures
- Auto-fix loop (3 attempts)
- Try alternative patterns
- Mark blocked if still failing
- Continue non-dependent work

## Security Considerations

1. **No hardcoded secrets** - Use environment variables
2. **Deviation flagging** - Security changes require review
3. **Security scanning** - npm audit, safety check
4. **OWASP compliance** - Patterns follow best practices
5. **Audit logging** - All actions tracked in state

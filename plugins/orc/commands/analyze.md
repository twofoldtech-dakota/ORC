---
description: "Analyze codebase conventions and patterns"
argument-hint: "[--force] [--focus <areas>]"
allowed-tools: [Read, Glob, Grep, Bash]
---

# /orc:analyze Command

## Usage

```
/orc:analyze [options]
```

## Options

| Option | Description |
|--------|-------------|
| `--force` | Force re-analysis even if cached profile exists |
| `--focus <areas>` | Comma-separated areas to focus on (conventions,dependencies,patterns,security,api,testing,infrastructure) |
| `--quiet` | Minimal output, only show warnings |

## Description

Performs a deep pre-flight analysis of the existing codebase to understand conventions, patterns, dependencies, and project structure. This analysis informs the planning phase to ensure all generated code integrates seamlessly with existing code.

## Behavior

### Automatic Invocation
The analyze phase runs automatically before planning if:
1. No codebase profile exists
2. Existing profile is stale (>24 hours old or significant git changes)

### Manual Invocation
Run manually to:
1. Force refresh the profile
2. Focus analysis on specific areas
3. View current profile details

### Analysis Process

1. **Project Detection** - Identify language, framework, platform
2. **Convention Extraction** - Naming, formatting, import styles
3. **Dependency Analysis** - Runtime and dev dependencies
4. **Pattern Detection** - Data access, error handling, auth, validation
5. **API Style Analysis** - REST/GraphQL, response formats
6. **Test Pattern Detection** - Framework, style, mocking approach
7. **Security Analysis** - Auth, secrets, vulnerabilities
8. **Infrastructure Analysis** - Docker, CI/CD, hosting
9. **Project Structure** - Directory organization, key files

## Output Format

```
🔍 Analyzing Codebase...

Project Type: typescript
Frameworks: express, prisma, react

📁 Structure
  └─ Organization: feature-based
  └─ Source: src/
  └─ Tests: colocated
  └─ Monorepo: no

📝 Conventions
  └─ Files: kebab-case
  └─ Functions: camelCase
  └─ Classes: PascalCase
  └─ Imports: alias (@/)
  └─ Indent: 2 spaces
  └─ Quotes: single
  └─ Semicolons: yes

📦 Dependencies
  └─ Package Manager: pnpm
  └─ Runtime: 24 packages
  └─ Dev: 18 packages
  └─ Key: express, prisma, zod, jest

🔧 Patterns
  └─ Data Access: repository
  └─ Error Handling: error-middleware
  └─ Validation: zod
  └─ Auth: jwt
  └─ Logging: pino

🌐 API
  └─ Format: REST
  └─ Versioning: url (/v1/)
  └─ Pagination: cursor
  └─ Rate Limiting: yes

🧪 Testing
  └─ Framework: jest
  └─ Style: describe-it
  └─ Mocking: jest.mock
  └─ E2E: playwright

🔒 Security
  └─ CORS: configured
  └─ Headers: helmet
  └─ Rate Limiting: yes
  └─ Secrets: env-vars

💾 Database
  └─ Type: postgresql
  └─ ORM: prisma
  └─ Migrations: prisma migrate
  └─ Pooling: yes

⚙️  Build
  └─ Tool: tsc
  └─ Linter: eslint
  └─ Formatter: prettier
  └─ CI: github-actions

✅ Analysis Complete
  └─ Files scanned: 247
  └─ Confidence: 92%
  └─ Warnings: 0

Profile saved to .orc/plan/codebase_profile.json
```

## Cached Profile

When a valid cached profile exists:

```
🔍 Codebase Profile (cached)

Last analyzed: 2 hours ago
Confidence: 92%

Project: typescript (express, prisma, react)
Organization: feature-based

Use /orc:analyze --force to refresh

Run /orc:plan <goal> to create a plan
```

## Stale Profile

When profile is stale:

```
⚠️  Codebase profile is stale

Last analyzed: 26 hours ago
Changes detected:
  - 3 new dependencies added
  - 2 new directories created
  - eslint.config.js modified

Re-analyzing...
```

## Warnings

When analysis detects issues:

```
⚠️  Analysis Warnings:

1. Mixed naming conventions detected
   - Files: 60% kebab-case, 40% camelCase
   - Recommendation: Standardize to kebab-case

2. No test framework detected
   - No test files found
   - Recommendation: Add jest or vitest

3. Hardcoded values found
   - src/config.ts: potential hardcoded secret
   - Recommendation: Use environment variables

4. Low confidence for API patterns (45%)
   - Insufficient route definitions to analyze
   - Will use best-effort defaults
```

## Directory Updates

Creates/updates if not exists:
```
.orc/
├── plan/
│   ├── codebase_profile.json   # Analysis output
│   └── ...
└── ...
```

## State Updates

- Sets `codebase_analyzed` flag in state
- Records `codebase_profile_at` timestamp
- Updates `analysis_confidence` score

## Error Handling

| Error | Response |
|-------|----------|
| No project manifest | "Warning: No package.json/requirements.txt found. Using best-effort analysis." |
| Empty project | "Error: No source files found. Is this an empty project?" |
| Permission denied | "Error: Cannot read [file]. Check permissions." |
| Analysis timeout | "Warning: Analysis timed out. Returning partial results." |

## Examples

### Basic Analysis
```
> /orc:analyze

🔍 Analyzing Codebase...
[full output as shown above]
```

### Force Refresh
```
> /orc:analyze --force

🔍 Re-analyzing Codebase (forced refresh)...
[full output]
```

### Focused Analysis
```
> /orc:analyze --focus security,patterns

🔍 Analyzing Codebase (focus: security, patterns)...

🔧 Patterns
  └─ Data Access: repository
  └─ Error Handling: error-middleware
  └─ Validation: zod
  └─ Auth: jwt

🔒 Security
  └─ CORS: configured
  └─ Headers: helmet
  └─ Rate Limiting: yes
  └─ Secrets: env-vars

✅ Analysis Complete (focused)
```

### Quiet Mode
```
> /orc:analyze --quiet

✅ Analysis complete (247 files, 92% confidence)
⚠️  1 warning: Mixed naming conventions detected
```

## Integration with Planning

When running `/orc:plan`, the analyzer phase executes automatically:

```
> /orc:plan "Add user management API"

🔍 Analyzing Codebase... ✓ (cached, 92% confidence)

📋 Planning: Add user management API...
[planning output using codebase profile]
```

This ensures the planner:
1. Uses existing naming conventions
2. Follows established patterns
3. Places files in correct locations
4. Uses compatible dependencies
5. Matches test styles

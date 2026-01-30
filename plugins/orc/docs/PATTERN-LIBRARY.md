# ORC Pattern Library

Guide to using and extending the pattern library.

## Overview

Patterns are reusable implementation approaches extracted from successful story completions. They help ORC suggest proven solutions for similar tasks.

## Pattern Types

### Success Patterns (`sp_`)
Approaches that have worked well. Suggested to implementers for similar tasks.

### Anti-Patterns (`ap_`)
Approaches to avoid. Warn implementers away from known pitfalls.

## Using Patterns

### Automatic Matching
When a story is created, ORC automatically:
1. Generates embedding for story description
2. Compares against pattern embeddings
3. Attaches high-similarity patterns as `suggested_approach`

### Manual Reference
View patterns with:
```
/orc patterns                  # All patterns
/orc patterns auth             # By category
/orc patterns search jwt       # Search
```

## Pattern Structure

```markdown
# Pattern: Pattern Name

**ID**: `sp_category_001`
**Category**: Category Name
**Confidence**: 90%

## When to Use

Circumstances where this pattern applies.

## Keywords

`keyword1`, `keyword2`, `keyword3`

## Approach

Description of the implementation approach.

## Code Example

\```typescript
// Example code
\```

## Security Notes

Any security considerations.

## Related Patterns

- `sp_related_001` - Description
```

## Creating Patterns

### File Location
Place patterns in `patterns/<category>/`:
- `patterns/auth/` - Authentication patterns
- `patterns/api/` - API patterns
- `patterns/database/` - Database patterns
- `patterns/testing/` - Testing patterns

### Required Fields

| Field | Description |
|-------|-------------|
| ID | Unique identifier (`sp_` or `ap_` prefix) |
| Category | Grouping category |
| Confidence | Initial confidence (0-100%) |
| When to Use | Applicability criteria |
| Keywords | Trigger words for matching |
| Approach | Implementation description |
| Code Example | Working code sample |

### Pattern ID Format
```
sp_<category>_<sequence>   # Success pattern
ap_<category>_<sequence>   # Anti-pattern

Examples:
sp_bcrypt_001
sp_jwt_rs256_002
ap_sync_bcrypt_001
```

## Automatic Learning

Patterns are extracted automatically after feature completion:

1. **Success Patterns**
   - Story completed on first attempt
   - No deviations recorded
   - High test coverage

2. **Anti-Patterns**
   - Story required multiple attempts
   - Specific approach caused failures
   - Security issues detected

3. **Pattern Updates**
   - Existing patterns get success/failure counts
   - Confidence recalculated
   - Code examples updated

## Confidence Scoring

```
confidence = base_confidence * recency_factor * success_factor

recency_factor = 0.95 ^ months_since_last_use
success_factor = success_count / (success_count + failure_count)
```

### Confidence Thresholds
| Level | Range | Behavior |
|-------|-------|----------|
| High | 80-100% | Strongly suggested |
| Medium | 50-79% | Suggested with alternatives |
| Low | 30-49% | Mentioned, not primary |
| Very Low | <30% | Flagged for review/deletion |

## Pattern Categories

### Authentication (`auth/`)
- Password hashing
- JWT tokens
- Session management
- OAuth flows
- MFA implementation

### API (`api/`)
- REST endpoints
- Error handling
- Request validation
- Rate limiting
- Pagination

### Database (`database/`)
- Schema design
- Migrations
- Repository pattern
- Query optimization
- Transactions

### Testing (`testing/`)
- Unit test structure
- Integration testing
- E2E testing
- Mocking strategies
- Test data management

### Frontend (`frontend/`)
- Component patterns
- State management
- Form handling
- Accessibility
- Performance

### DevOps (`devops/`)
- CI/CD pipelines
- Docker configuration
- Kubernetes manifests
- Monitoring setup

## Extending the Library

### Adding New Categories

1. Create directory: `patterns/<category>/`
2. Add pattern files
3. Update documentation

### Importing External Patterns

Convert external patterns to ORC format:
1. Extract approach description
2. Identify keywords
3. Add code examples
4. Set initial confidence (typically 70%)

### Pattern Versioning

When approaches change:
1. Keep old pattern with lower confidence
2. Create new pattern with updated approach
3. Link via `superseded_by` field

## Best Practices

1. **Specific over general** - Patterns for specific scenarios work better
2. **Working code** - Examples must be copy-pasteable
3. **Keywords matter** - Good keywords improve matching
4. **Update regularly** - Patterns should reflect current best practices
5. **Include anti-patterns** - What not to do is as valuable as what to do

## Example: Adding a Pattern

1. Create file:
```bash
touch patterns/api/rate-limiting.md
```

2. Write pattern:
```markdown
# Pattern: Rate Limiting with Redis

**ID**: `sp_rate_limit_redis_001`
**Category**: API
**Confidence**: 85%

## When to Use
- API endpoints needing rate limiting
- Distributed systems requiring shared state
- High-traffic endpoints

## Keywords
`rate limit`, `throttle`, `redis`, `api`, `request limit`

## Approach
Use Redis with sliding window algorithm...

## Code Example
\```typescript
// Implementation...
\```
```

3. Pattern becomes available for matching

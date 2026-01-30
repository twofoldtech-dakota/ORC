# Creating Custom Specialists

Guide for creating custom specialist agents for ORC.

## Overview

Specialists are domain experts spawned by core agents when specific expertise is needed. They receive focused context and return structured recommendations.

## Specialist Structure

Create a new file in `agents/specialists/`:

```markdown
---
name: your-specialist
type: specialist
model: opus
tools: [Read, Glob, Grep, Bash]
spawned_by: [planner, implementer]
---

# Your Specialist Name

## Role

Brief description of the specialist's expertise and purpose.

## Expertise Areas

- Area 1
- Area 2
- Area 3

## Input Contract

```json
{
  "request_type": {
    "type": "string",
    "enum": ["type1", "type2", "type3"]
  },
  "context": {
    "type": "object",
    "properties": {
      "relevant_data": { "type": "string" }
    }
  },
  "specific_question": { "type": "string" }
}
```

## Execution Protocol

### Request Type 1
1. Step one
2. Step two
3. Step three

### Request Type 2
1. Step one
2. Step two

## Output Contract

```json
{
  "request_type": "type1",
  "analysis": {},
  "recommendations": []
}
```

## Common Patterns/Examples

[Include code examples, templates, best practices]

## Related Patterns

- `sp_related_001` - Description
```

## Required Sections

### Frontmatter (Required)
```yaml
---
name: unique-identifier
type: specialist
model: opus
tools: [list, of, tools]
spawned_by: [parent, agents]
---
```

### Role (Required)
One paragraph describing what this specialist does.

### Expertise Areas (Required)
Bullet list of specific areas of knowledge.

### Input Contract (Required)
JSON Schema defining what the specialist accepts:
- `request_type`: Enum of operation types
- `context`: Relevant data for the request
- `specific_question`: Optional free-form question

### Execution Protocol (Required)
Step-by-step instructions for each request type.

### Output Contract (Required)
JSON Schema defining what the specialist returns.

## Optional Sections

### Common Patterns
Code examples and templates the specialist commonly uses.

### Related Patterns
Links to ORC patterns this specialist relates to.

### Security Notes
For security-sensitive specialists.

### Best Practices
Domain-specific recommendations.

## Example: API Versioning Specialist

```markdown
---
name: api-versioning
type: specialist
model: opus
tools: [Read, Glob, Grep]
spawned_by: [planner, implementer]
---

# API Versioning Specialist

## Role

Provides expertise on API versioning strategies, migration paths, and backward compatibility for REST and GraphQL APIs.

## Expertise Areas

- URL path versioning (/v1/, /v2/)
- Header-based versioning
- Query parameter versioning
- Content negotiation
- Deprecation strategies
- Migration planning
- Backward compatibility

## Input Contract

```json
{
  "request_type": {
    "type": "string",
    "enum": ["strategy_recommendation", "migration_plan", "compatibility_check"]
  },
  "context": {
    "type": "object",
    "properties": {
      "current_api": { "type": "string" },
      "proposed_changes": { "type": "array" },
      "client_types": { "type": "array" }
    }
  }
}
```

## Execution Protocol

### Strategy Recommendation
1. Analyze current API structure
2. Evaluate client diversity
3. Assess change frequency
4. Recommend versioning approach
5. Document trade-offs

### Migration Plan
1. Identify breaking changes
2. Define version timeline
3. Plan deprecation notices
4. Create migration guide
5. Define sunset dates

## Output Contract

```json
{
  "request_type": "strategy_recommendation",
  "recommendation": {
    "strategy": "url_path|header|query",
    "rationale": "string",
    "implementation": {}
  },
  "migration_steps": [],
  "timeline": {}
}
```
```

## Registering the Specialist

Add to `plugin.json`:

```json
{
  "agents": {
    "specialists": [
      "agents/specialists/api-versioning.md"
    ]
  }
}
```

## Spawn Triggers

Specialists are spawned when:

1. **Keywords match** - Story contains relevant keywords
2. **Explicit marking** - Planner marks story needs specialist
3. **Agent request** - Core agent decides mid-task

Add keywords to your specialist that trigger automatic spawning.

## Testing Your Specialist

1. Create test stories that should trigger your specialist
2. Verify correct spawning
3. Test all request types
4. Validate output format
5. Check integration with parent agents

## Best Practices

1. **Focused scope** - One specialist, one domain
2. **Clear contracts** - Well-defined input/output
3. **Practical examples** - Include real code
4. **Actionable output** - Recommendations should be implementable
5. **Error guidance** - Include common pitfalls

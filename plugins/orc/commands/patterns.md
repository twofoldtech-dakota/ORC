---
description: "Show learned patterns"
argument-hint: "[<category>] [search <query>]"
allowed-tools: [Read]
---

# /orc:patterns Command

## Usage

```
/orc:patterns                  # Show all patterns
/orc:patterns <category>       # Show patterns by category
/orc:patterns search <query>   # Search patterns
```

## Description

Displays learned patterns from the learnings database. Patterns are used to suggest implementation approaches for new stories.

## Output

### All Patterns (`/orc:patterns`)

```
📚 Learned Patterns

Success Patterns (23):
  Authentication (5):
    ✓ sp_bcrypt_001    Password hashing with bcrypt    [94% confidence]
    ✓ sp_jwt_001       JWT token generation            [89% confidence]
    ✓ sp_jwt_rs256_001 JWT with RS256 algorithm        [92% confidence]
    ✓ sp_session_001   Session management              [85% confidence]
    ✓ sp_oauth_001     OAuth 2.0 flow                  [78% confidence]

  API (6):
    ✓ sp_rest_001      RESTful endpoint structure      [91% confidence]
    ✓ sp_validation_001 Zod request validation         [88% confidence]
    ✓ sp_error_001     Error handling middleware       [93% confidence]
    ✓ sp_pagination_001 Cursor-based pagination        [82% confidence]
    ✓ sp_rate_limit_001 Rate limiting                  [79% confidence]
    ✓ sp_cors_001      CORS configuration              [95% confidence]

  Database (4):
    ✓ sp_prisma_001    Prisma model structure          [90% confidence]
    ✓ sp_migration_001 Database migrations             [87% confidence]
    ✓ sp_repository_001 Repository pattern             [84% confidence]
    ✓ sp_transaction_001 Transaction handling          [81% confidence]

  ... (more categories)

Anti-Patterns (8):
    ✗ ap_sync_bcrypt   Sync bcrypt in handlers         [Never use]
    ✗ ap_jwt_none      JWT with 'none' algorithm       [Never use]
    ✗ ap_sql_concat    SQL string concatenation        [Never use]
    ... (more)

Low Confidence (needs review):
    ⚠ sp_email_001     Email service pattern           [25% confidence]

Commands:
  /orc:patterns auth         - Show auth patterns
  /orc:patterns search jwt   - Search for 'jwt'
```

### Category View (`/orc:patterns <category>`)

```
📚 Authentication Patterns

sp_bcrypt_001: Password hashing with bcrypt
  Confidence: 94%
  Success: 12 | Failures: 1
  Last used: 2025-01-30

  Applicable when: password, user model, authentication, hashing

  Approach:
    Use bcrypt.hash with cost factor 12, async methods only.
    Never use hashSync in request handlers.

  Example:
    const hash = await bcrypt.hash(password, 12);
    const valid = await bcrypt.compare(password, hash);

---

sp_jwt_001: JWT token generation
  Confidence: 89%
  Success: 8 | Failures: 1
  Last used: 2025-01-28

  Applicable when: jwt, token, authentication, api

  Approach:
    Use jsonwebtoken with short expiry (15m) for access tokens.
    Use refresh tokens with longer expiry (7d) stored securely.

  Example:
    const token = jwt.sign(payload, secret, { expiresIn: '15m' });

... (more patterns)
```

### Search (`/orc:patterns search <query>`)

```
📚 Search Results: "jwt"

Found 3 patterns:

1. sp_jwt_001: JWT token generation [89%]
   Use jsonwebtoken with short expiry for access tokens...

2. sp_jwt_rs256_001: JWT with RS256 algorithm [92%]
   Use RS256 for distributed systems with public key verification...

3. ap_jwt_none: JWT with 'none' algorithm [ANTI-PATTERN]
   Never use 'none' algorithm - allows signature bypass...
```

## Pattern Details

Each pattern includes:
- **ID**: Unique identifier
- **Name**: Human-readable name
- **Confidence**: Based on success rate and recency
- **Stats**: Success/failure counts
- **Applicable when**: Keywords that trigger matching
- **Approach**: Description of the pattern
- **Example**: Code example

## Error Handling

| Error | Response |
|-------|----------|
| No learnings | "No patterns learned yet. Patterns are extracted after feature completion." |
| Category not found | "Category '<cat>' not found. Available: auth, api, database, testing" |
| No search results | "No patterns found matching '<query>'." |

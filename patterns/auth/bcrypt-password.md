# Pattern: Bcrypt Password Hashing

**ID**: `sp_bcrypt_001`
**Category**: Authentication
**Confidence**: 94%

## When to Use

- User registration with password
- Password change functionality
- Any password storage requirement

## Keywords

`password`, `hash`, `bcrypt`, `authentication`, `user model`, `registration`

## Approach

Use bcrypt with async methods and cost factor 12 (or higher for sensitive systems). Never use synchronous methods in request handlers as they block the event loop.

## Code Example

```typescript
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

export async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return bcrypt.compare(password, hash);
}
```

## Usage in User Model

```typescript
import { hashPassword, verifyPassword } from '../lib/auth';

export class UserService {
  async createUser(email: string, password: string) {
    const passwordHash = await hashPassword(password);

    return prisma.user.create({
      data: {
        email,
        passwordHash,
      },
    });
  }

  async validateCredentials(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } });

    if (!user) {
      return null;
    }

    const valid = await verifyPassword(password, user.passwordHash);
    return valid ? user : null;
  }
}
```

## Anti-Pattern Warning

Never use `bcrypt.hashSync()` or `bcrypt.compareSync()` in API handlers:

```typescript
// ❌ BAD - blocks event loop
const hash = bcrypt.hashSync(password, 12);

// ✅ GOOD - async
const hash = await bcrypt.hash(password, 12);
```

## Security Notes

- Cost factor 12 provides good security vs performance balance
- Increase to 14+ for highly sensitive systems
- bcrypt automatically handles salting
- Store only the hash, never the plain password

## Related Patterns

- `sp_jwt_001` - JWT token generation
- `sp_session_001` - Session management

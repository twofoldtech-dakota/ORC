# Pattern: JWT Token Generation

**ID**: `sp_jwt_001`
**Category**: Authentication
**Confidence**: 89%

## When to Use

- API authentication
- Stateless session management
- Microservices authentication
- Mobile app authentication

## Keywords

`jwt`, `token`, `authentication`, `api`, `bearer`, `access token`, `refresh token`

## Approach

Use short-lived access tokens (15 minutes) with longer-lived refresh tokens (7 days). Store refresh tokens securely. Use HS256 for single-service or RS256 for distributed systems.

## Code Example

```typescript
import jwt from 'jsonwebtoken';

const ACCESS_TOKEN_SECRET = process.env.JWT_ACCESS_SECRET!;
const REFRESH_TOKEN_SECRET = process.env.JWT_REFRESH_SECRET!;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

interface TokenPayload {
  userId: string;
  email: string;
}

export function generateAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, ACCESS_TOKEN_SECRET, {
    expiresIn: ACCESS_TOKEN_EXPIRY,
    algorithm: 'HS256',
  });
}

export function generateRefreshToken(payload: TokenPayload): string {
  return jwt.sign(payload, REFRESH_TOKEN_SECRET, {
    expiresIn: REFRESH_TOKEN_EXPIRY,
    algorithm: 'HS256',
  });
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, ACCESS_TOKEN_SECRET) as TokenPayload;
}

export function verifyRefreshToken(token: string): TokenPayload {
  return jwt.verify(token, REFRESH_TOKEN_SECRET) as TokenPayload;
}
```

## Token Pair Generation

```typescript
export function generateTokenPair(user: User) {
  const payload = { userId: user.id, email: user.email };

  return {
    accessToken: generateAccessToken(payload),
    refreshToken: generateRefreshToken(payload),
    expiresIn: 900, // 15 minutes in seconds
  };
}
```

## Middleware Usage

```typescript
export async function authMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing token' });
  }

  const token = authHeader.slice(7);

  try {
    const payload = verifyAccessToken(token);
    req.user = payload;
    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

## Refresh Token Flow

```typescript
export async function refreshTokens(refreshToken: string) {
  try {
    const payload = verifyRefreshToken(refreshToken);

    // Optionally: Check if refresh token is in whitelist/database
    // Optionally: Rotate refresh token

    return generateTokenPair({ id: payload.userId, email: payload.email } as User);
  } catch (error) {
    throw new Error('Invalid refresh token');
  }
}
```

## Security Notes

- Use separate secrets for access and refresh tokens
- Store secrets in environment variables
- Consider token rotation for refresh tokens
- Implement token revocation for logout
- Use HTTPS only

## Related Patterns

- `sp_bcrypt_001` - Password hashing
- `sp_jwt_rs256_001` - JWT with RS256 for distributed systems

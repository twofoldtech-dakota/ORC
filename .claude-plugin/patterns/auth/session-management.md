# Pattern: Session Management

**ID**: `sp_session_001`
**Category**: Authentication
**Confidence**: 85%

## When to Use

- Server-rendered applications
- When you need server-side session control
- Applications requiring immediate session invalidation
- Traditional web applications

## Keywords

`session`, `cookie`, `authentication`, `login`, `logout`, `web app`

## Approach

Use secure HTTP-only cookies with server-side session storage. Consider Redis for distributed systems or database storage for persistence.

## Code Example (Express + Redis)

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient({
  url: process.env.REDIS_URL,
});

await redisClient.connect();

const sessionMiddleware = session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  name: 'sessionId', // Don't use default 'connect.sid'
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 1000 * 60 * 60 * 24, // 24 hours
  },
});

app.use(sessionMiddleware);
```

## Session Types

```typescript
declare module 'express-session' {
  interface SessionData {
    userId: string;
    email: string;
    role: string;
    createdAt: number;
  }
}
```

## Login Flow

```typescript
export async function login(req: Request, res: Response) {
  const { email, password } = req.body;

  const user = await UserService.validateCredentials(email, password);

  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Regenerate session to prevent fixation
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ error: 'Session error' });
    }

    req.session.userId = user.id;
    req.session.email = user.email;
    req.session.role = user.role;
    req.session.createdAt = Date.now();

    res.json({ success: true, user: { id: user.id, email: user.email } });
  });
}
```

## Logout Flow

```typescript
export async function logout(req: Request, res: Response) {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' });
    }

    res.clearCookie('sessionId');
    res.json({ success: true });
  });
}
```

## Auth Middleware

```typescript
export function requireAuth(req: Request, res: Response, next: NextFunction) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  // Optionally check session age
  const sessionAge = Date.now() - (req.session.createdAt || 0);
  const maxAge = 1000 * 60 * 60 * 24; // 24 hours

  if (sessionAge > maxAge) {
    return req.session.destroy(() => {
      res.status(401).json({ error: 'Session expired' });
    });
  }

  next();
}
```

## Security Notes

- Always use `httpOnly: true` to prevent XSS access
- Use `secure: true` in production (HTTPS only)
- Use `sameSite: 'strict'` or `'lax'` for CSRF protection
- Regenerate session ID after login
- Implement session timeout
- Consider session fixation protection

## Related Patterns

- `sp_bcrypt_001` - Password hashing
- `sp_jwt_001` - JWT for stateless auth

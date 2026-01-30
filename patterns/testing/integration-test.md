# Pattern: Integration Testing

**ID**: `sp_integration_test_001`
**Category**: Testing
**Confidence**: 86%

## When to Use

- Testing API endpoints
- Testing database operations
- Testing multiple components together
- Verifying system behavior

## Keywords

`integration test`, `api test`, `supertest`, `database test`, `e2e`

## Approach

Use real database (test instance) and make actual HTTP requests. Reset database state between tests. Test the full request/response cycle.

## Code Example

### Test Setup
```typescript
// src/test/setup.ts
import { beforeAll, afterAll, beforeEach } from 'vitest';
import { prisma } from '../lib/prisma';

beforeAll(async () => {
  // Connect to test database
  await prisma.$connect();
});

afterAll(async () => {
  await prisma.$disconnect();
});

beforeEach(async () => {
  // Clean database before each test
  await prisma.$transaction([
    prisma.post.deleteMany(),
    prisma.user.deleteMany(),
  ]);
});
```

### API Integration Tests
```typescript
// src/routes/users.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '../app';
import { prisma } from '../lib/prisma';
import { hashPassword } from '../lib/auth';

describe('POST /api/users', () => {
  it('should create a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      success: true,
      data: {
        email: 'test@example.com',
        name: 'Test User',
      },
    });
    expect(response.body.data).not.toHaveProperty('password');
    expect(response.body.data).not.toHaveProperty('passwordHash');

    // Verify database
    const user = await prisma.user.findUnique({
      where: { email: 'test@example.com' },
    });
    expect(user).not.toBeNull();
  });

  it('should return 409 for duplicate email', async () => {
    // Create existing user
    await prisma.user.create({
      data: {
        email: 'existing@example.com',
        passwordHash: await hashPassword('password'),
        name: 'Existing User',
      },
    });

    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'existing@example.com',
        password: 'password123',
        name: 'Another User',
      });

    expect(response.status).toBe(409);
    expect(response.body.success).toBe(false);
    expect(response.body.error.code).toBe('CONFLICT');
  });

  it('should return 400 for invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'invalid-email',
        password: 'password123',
        name: 'Test User',
      });

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });
});

describe('GET /api/users/:id', () => {
  let authToken: string;
  let userId: string;

  beforeEach(async () => {
    // Create test user and get auth token
    const user = await prisma.user.create({
      data: {
        email: 'auth@example.com',
        passwordHash: await hashPassword('password'),
        name: 'Auth User',
      },
    });
    userId = user.id;

    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send({ email: 'auth@example.com', password: 'password' });

    authToken = loginResponse.body.data.accessToken;
  });

  it('should return user by ID', async () => {
    const response = await request(app)
      .get(`/api/users/${userId}`)
      .set('Authorization', `Bearer ${authToken}`);

    expect(response.status).toBe(200);
    expect(response.body.data.id).toBe(userId);
  });

  it('should return 401 without auth', async () => {
    const response = await request(app)
      .get(`/api/users/${userId}`);

    expect(response.status).toBe(401);
  });

  it('should return 404 for non-existent user', async () => {
    const response = await request(app)
      .get('/api/users/00000000-0000-0000-0000-000000000000')
      .set('Authorization', `Bearer ${authToken}`);

    expect(response.status).toBe(404);
  });
});
```

### Database Integration Tests
```typescript
// src/repositories/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { userRepository } from './user';
import { prisma } from '../lib/prisma';

describe('UserRepository', () => {
  describe('create', () => {
    it('should create user in database', async () => {
      const user = await userRepository.create({
        email: 'test@example.com',
        passwordHash: 'hashed',
        name: 'Test User',
      });

      expect(user.id).toBeDefined();
      expect(user.email).toBe('test@example.com');

      // Verify in database
      const dbUser = await prisma.user.findUnique({
        where: { id: user.id },
      });
      expect(dbUser).not.toBeNull();
    });

    it('should throw on duplicate email', async () => {
      await userRepository.create({
        email: 'duplicate@example.com',
        passwordHash: 'hashed',
        name: 'First User',
      });

      await expect(
        userRepository.create({
          email: 'duplicate@example.com',
          passwordHash: 'hashed',
          name: 'Second User',
        })
      ).rejects.toThrow();
    });
  });

  describe('findMany', () => {
    beforeEach(async () => {
      // Create test data
      await prisma.user.createMany({
        data: Array.from({ length: 25 }, (_, i) => ({
          email: `user${i}@example.com`,
          passwordHash: 'hashed',
          name: `User ${i}`,
        })),
      });
    });

    it('should paginate results', async () => {
      const page1 = await userRepository.findMany({ page: 1, limit: 10 });
      expect(page1.data).toHaveLength(10);
      expect(page1.total).toBe(25);
      expect(page1.hasMore).toBe(true);

      const page3 = await userRepository.findMany({ page: 3, limit: 10 });
      expect(page3.data).toHaveLength(5);
      expect(page3.hasMore).toBe(false);
    });
  });
});
```

### Test Helpers
```typescript
// src/test/helpers/auth.ts
import request from 'supertest';
import { app } from '../../app';
import { prisma } from '../../lib/prisma';
import { hashPassword } from '../../lib/auth';

export async function createAuthenticatedUser(email = 'test@example.com') {
  const user = await prisma.user.create({
    data: {
      email,
      passwordHash: await hashPassword('password'),
      name: 'Test User',
    },
  });

  const response = await request(app)
    .post('/api/auth/login')
    .send({ email, password: 'password' });

  return {
    user,
    token: response.body.data.accessToken,
  };
}
```

## Test Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
});
```

## Related Patterns

- `sp_unit_test_001` - Unit testing
- `sp_e2e_test_001` - End-to-end testing

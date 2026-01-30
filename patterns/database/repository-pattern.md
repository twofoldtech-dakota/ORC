# Pattern: Repository Pattern

**ID**: `sp_repository_001`
**Category**: Database
**Confidence**: 84%

## When to Use

- Abstracting database access
- When you might switch ORMs/databases
- Complex query logic
- Unit testing with mocks

## Keywords

`repository`, `database`, `data access`, `prisma`, `orm`, `query`

## Approach

Create repository classes that encapsulate data access logic. Keep database-specific code isolated from business logic.

## Code Example

### Base Repository
```typescript
// src/repositories/base.ts
export interface PaginationOptions {
  page: number;
  limit: number;
}

export interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  hasMore: boolean;
}

export abstract class BaseRepository<T, CreateInput, UpdateInput> {
  abstract findById(id: string): Promise<T | null>;
  abstract findMany(options?: PaginationOptions): Promise<PaginatedResult<T>>;
  abstract create(data: CreateInput): Promise<T>;
  abstract update(id: string, data: UpdateInput): Promise<T>;
  abstract delete(id: string): Promise<void>;
}
```

### User Repository Implementation
```typescript
// src/repositories/user.ts
import { prisma } from '../lib/prisma';
import { BaseRepository, PaginationOptions, PaginatedResult } from './base';

interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

interface CreateUserInput {
  email: string;
  passwordHash: string;
  name: string;
}

interface UpdateUserInput {
  name?: string;
  bio?: string;
}

const userSelect = {
  id: true,
  email: true,
  name: true,
  createdAt: true,
} as const;

export class UserRepository extends BaseRepository<User, CreateUserInput, UpdateUserInput> {
  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id },
      select: userSelect,
    });
  }

  async findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { email },
      select: userSelect,
    });
  }

  async findByEmailWithPassword(email: string) {
    return prisma.user.findUnique({
      where: { email },
      select: {
        ...userSelect,
        passwordHash: true,
      },
    });
  }

  async findMany(options: PaginationOptions = { page: 1, limit: 20 }): Promise<PaginatedResult<User>> {
    const { page, limit } = options;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      prisma.user.findMany({
        select: userSelect,
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count(),
    ]);

    return {
      data,
      total,
      page,
      limit,
      hasMore: skip + data.length < total,
    };
  }

  async create(data: CreateUserInput): Promise<User> {
    return prisma.user.create({
      data,
      select: userSelect,
    });
  }

  async update(id: string, data: UpdateUserInput): Promise<User> {
    return prisma.user.update({
      where: { id },
      data,
      select: userSelect,
    });
  }

  async delete(id: string): Promise<void> {
    await prisma.user.delete({ where: { id } });
  }

  // Custom queries
  async search(query: string, options: PaginationOptions): Promise<PaginatedResult<User>> {
    const { page, limit } = options;
    const skip = (page - 1) * limit;

    const where = {
      OR: [
        { name: { contains: query, mode: 'insensitive' as const } },
        { email: { contains: query, mode: 'insensitive' as const } },
      ],
    };

    const [data, total] = await Promise.all([
      prisma.user.findMany({
        where,
        select: userSelect,
        skip,
        take: limit,
      }),
      prisma.user.count({ where }),
    ]);

    return {
      data,
      total,
      page,
      limit,
      hasMore: skip + data.length < total,
    };
  }
}

// Export singleton
export const userRepository = new UserRepository();
```

### Service Usage
```typescript
// src/services/user.ts
import { userRepository } from '../repositories/user';
import { hashPassword, verifyPassword } from '../lib/auth';
import { NotFoundError, ConflictError, UnauthorizedError } from '../lib/errors';

export class UserService {
  static async findById(id: string) {
    const user = await userRepository.findById(id);
    if (!user) throw NotFoundError('User not found');
    return user;
  }

  static async create(data: { email: string; password: string; name: string }) {
    const existing = await userRepository.findByEmail(data.email);
    if (existing) throw ConflictError('Email already registered');

    const passwordHash = await hashPassword(data.password);
    return userRepository.create({
      email: data.email,
      passwordHash,
      name: data.name,
    });
  }

  static async authenticate(email: string, password: string) {
    const user = await userRepository.findByEmailWithPassword(email);
    if (!user) throw UnauthorizedError('Invalid credentials');

    const valid = await verifyPassword(password, user.passwordHash);
    if (!valid) throw UnauthorizedError('Invalid credentials');

    const { passwordHash: _, ...userWithoutPassword } = user;
    return userWithoutPassword;
  }
}
```

## Testing with Mocks
```typescript
// src/repositories/__mocks__/user.ts
export const userRepository = {
  findById: jest.fn(),
  findByEmail: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
};
```

## Related Patterns

- `sp_prisma_001` - Prisma model structure
- `sp_transaction_001` - Transaction handling

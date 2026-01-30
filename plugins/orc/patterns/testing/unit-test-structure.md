# Pattern: Unit Test Structure

**ID**: `sp_unit_test_001`
**Category**: Testing
**Confidence**: 90%

## When to Use

- Testing individual functions/methods
- Testing service logic
- Testing utilities
- Any isolated unit of code

## Keywords

`test`, `unit test`, `jest`, `vitest`, `describe`, `it`, `expect`

## Approach

Use Arrange-Act-Assert (AAA) pattern. Group tests by functionality with `describe` blocks. Test both happy path and edge cases.

## Code Example

### Basic Structure
```typescript
// src/services/user.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { UserService } from './user';
import { userRepository } from '../repositories/user';
import { hashPassword } from '../lib/auth';
import { ConflictError, NotFoundError } from '../lib/errors';

// Mock dependencies
vi.mock('../repositories/user');
vi.mock('../lib/auth');

describe('UserService', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('create', () => {
    const validInput = {
      email: 'test@example.com',
      password: 'password123',
      name: 'Test User',
    };

    it('should create user with valid input', async () => {
      // Arrange
      vi.mocked(userRepository.findByEmail).mockResolvedValue(null);
      vi.mocked(hashPassword).mockResolvedValue('hashed_password');
      vi.mocked(userRepository.create).mockResolvedValue({
        id: '1',
        email: validInput.email,
        name: validInput.name,
        createdAt: new Date(),
      });

      // Act
      const result = await UserService.create(validInput);

      // Assert
      expect(result).toMatchObject({
        email: validInput.email,
        name: validInput.name,
      });
      expect(userRepository.create).toHaveBeenCalledWith({
        email: validInput.email,
        passwordHash: 'hashed_password',
        name: validInput.name,
      });
    });

    it('should throw ConflictError for duplicate email', async () => {
      // Arrange
      vi.mocked(userRepository.findByEmail).mockResolvedValue({
        id: '1',
        email: validInput.email,
        name: 'Existing User',
        createdAt: new Date(),
      });

      // Act & Assert
      await expect(UserService.create(validInput)).rejects.toThrow(ConflictError);
      expect(userRepository.create).not.toHaveBeenCalled();
    });
  });

  describe('findById', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser = {
        id: '1',
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date(),
      };
      vi.mocked(userRepository.findById).mockResolvedValue(mockUser);

      // Act
      const result = await UserService.findById('1');

      // Assert
      expect(result).toEqual(mockUser);
    });

    it('should throw NotFoundError when user not found', async () => {
      // Arrange
      vi.mocked(userRepository.findById).mockResolvedValue(null);

      // Act & Assert
      await expect(UserService.findById('nonexistent')).rejects.toThrow(NotFoundError);
    });
  });
});
```

### Testing Utilities
```typescript
// src/lib/utils.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate, slugify, truncate } from './utils';

describe('formatDate', () => {
  it('should format date in default locale', () => {
    const date = new Date('2025-01-30T10:00:00Z');
    expect(formatDate(date)).toBe('January 30, 2025');
  });

  it('should handle invalid date', () => {
    expect(formatDate(new Date('invalid'))).toBe('Invalid Date');
  });
});

describe('slugify', () => {
  it.each([
    ['Hello World', 'hello-world'],
    ['Multiple   Spaces', 'multiple-spaces'],
    ['Special @#$ Characters!', 'special-characters'],
    ['  Trim Whitespace  ', 'trim-whitespace'],
    ['MixedCASE', 'mixedcase'],
  ])('should convert "%s" to "%s"', (input, expected) => {
    expect(slugify(input)).toBe(expected);
  });
});

describe('truncate', () => {
  it('should not truncate short strings', () => {
    expect(truncate('short', 10)).toBe('short');
  });

  it('should truncate long strings with ellipsis', () => {
    expect(truncate('this is a long string', 10)).toBe('this is...');
  });

  it('should handle exact length', () => {
    expect(truncate('exact', 5)).toBe('exact');
  });
});
```

### Testing Async Code
```typescript
describe('async operations', () => {
  it('should handle async success', async () => {
    const result = await fetchData();
    expect(result).toBeDefined();
  });

  it('should handle async errors', async () => {
    await expect(fetchBadData()).rejects.toThrow('Failed to fetch');
  });

  it('should resolve within timeout', async () => {
    await expect(slowOperation()).resolves.toBe('done');
  }, 5000); // 5 second timeout
});
```

### Testing with Fixtures
```typescript
// src/test/fixtures/users.ts
export const testUsers = {
  admin: {
    id: '1',
    email: 'admin@example.com',
    name: 'Admin User',
    role: 'ADMIN',
  },
  regular: {
    id: '2',
    email: 'user@example.com',
    name: 'Regular User',
    role: 'USER',
  },
};

// Usage in tests
import { testUsers } from '../test/fixtures/users';

it('should allow admin access', async () => {
  vi.mocked(getCurrentUser).mockResolvedValue(testUsers.admin);
  // ...
});
```

## Test Organization

```
src/
├── services/
│   ├── user.ts
│   └── user.test.ts      # Colocated tests
├── lib/
│   ├── utils.ts
│   └── utils.test.ts
└── test/
    ├── fixtures/          # Test data
    ├── helpers/           # Test utilities
    └── setup.ts           # Global setup
```

## Related Patterns

- `sp_integration_test_001` - Integration testing
- `sp_mock_001` - Mocking strategies

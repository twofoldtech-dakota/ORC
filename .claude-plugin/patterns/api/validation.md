# Pattern: Request Validation with Zod

**ID**: `sp_validation_001`
**Category**: API
**Confidence**: 88%

## When to Use

- Validating API request bodies
- Validating query parameters
- Validating URL parameters
- Any input validation

## Keywords

`validation`, `zod`, `schema`, `request`, `input`, `body`, `params`, `query`

## Approach

Use Zod schemas for type-safe validation. Create reusable validation middleware and colocate schemas with routes.

## Code Example

### Validation Middleware
```typescript
// src/middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { AnyZodObject, ZodError } from 'zod';

interface ValidateOptions {
  body?: AnyZodObject;
  query?: AnyZodObject;
  params?: AnyZodObject;
}

export function validateRequest(schema: AnyZodObject | ValidateOptions) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      if ('body' in schema || 'query' in schema || 'params' in schema) {
        // Multiple schemas
        const options = schema as ValidateOptions;
        if (options.body) {
          req.body = await options.body.parseAsync(req.body);
        }
        if (options.query) {
          req.query = await options.query.parseAsync(req.query);
        }
        if (options.params) {
          req.params = await options.params.parseAsync(req.params);
        }
      } else {
        // Single schema for body
        await schema.parseAsync({
          body: req.body,
          query: req.query,
          params: req.params,
        });
      }
      next();
    } catch (error) {
      next(error);
    }
  };
}
```

### Schema Definitions
```typescript
// src/schemas/user.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  body: z.object({
    email: z.string().email('Invalid email address'),
    password: z
      .string()
      .min(8, 'Password must be at least 8 characters')
      .max(128, 'Password too long'),
    name: z
      .string()
      .min(1, 'Name is required')
      .max(100, 'Name too long'),
  }),
});

export const updateUserSchema = z.object({
  params: z.object({
    id: z.string().uuid('Invalid user ID'),
  }),
  body: z.object({
    name: z.string().min(1).max(100).optional(),
    bio: z.string().max(500).optional(),
  }),
});

export const getUserSchema = z.object({
  params: z.object({
    id: z.string().uuid('Invalid user ID'),
  }),
});

export const listUsersSchema = z.object({
  query: z.object({
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().min(1).max(100).default(20),
    search: z.string().optional(),
    sort: z.enum(['name', 'createdAt', '-name', '-createdAt']).default('createdAt'),
  }),
});

// Type inference
export type CreateUserInput = z.infer<typeof createUserSchema>['body'];
export type UpdateUserInput = z.infer<typeof updateUserSchema>['body'];
export type ListUsersQuery = z.infer<typeof listUsersSchema>['query'];
```

### Route Usage
```typescript
// src/routes/users.ts
import { Router } from 'express';
import { validateRequest } from '../middleware/validate';
import {
  createUserSchema,
  updateUserSchema,
  getUserSchema,
  listUsersSchema,
} from '../schemas/user';
import { UserController } from '../controllers/user';

const router = Router();

router.get('/', validateRequest(listUsersSchema), UserController.list);
router.get('/:id', validateRequest(getUserSchema), UserController.getById);
router.post('/', validateRequest(createUserSchema), UserController.create);
router.patch('/:id', validateRequest(updateUserSchema), UserController.update);

export { router as userRoutes };
```

### Common Schema Patterns
```typescript
// src/schemas/common.ts
import { z } from 'zod';

// Reusable validators
export const email = z.string().email();
export const password = z.string().min(8).max(128);
export const uuid = z.string().uuid();

// Pagination
export const paginationQuery = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
});

// Date range
export const dateRangeQuery = z.object({
  from: z.coerce.date().optional(),
  to: z.coerce.date().optional(),
}).refine(
  (data) => !data.from || !data.to || data.from <= data.to,
  { message: 'From date must be before to date' }
);

// Sort order
export const sortOrder = <T extends string>(fields: T[]) =>
  z.enum([
    ...fields,
    ...fields.map((f) => `-${f}` as const),
  ] as [string, ...string[]]);
```

## Error Response Format
```json
{
  "success": false,
  "error": {
    "message": "Validation failed",
    "code": "VALIDATION_ERROR",
    "details": [
      {
        "field": "body.email",
        "message": "Invalid email address"
      },
      {
        "field": "body.password",
        "message": "Password must be at least 8 characters"
      }
    ]
  }
}
```

## Related Patterns

- `sp_rest_001` - REST endpoint structure
- `sp_error_001` - Error handling

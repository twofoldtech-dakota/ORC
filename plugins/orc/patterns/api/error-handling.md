# Pattern: Error Handling Middleware

**ID**: `sp_error_001`
**Category**: API
**Confidence**: 93%

## When to Use

- Any Express/Fastify API
- Centralized error handling
- Consistent error responses

## Keywords

`error`, `error handling`, `middleware`, `exception`, `api`, `response`

## Approach

Create a custom error class and centralized error handling middleware. Catch all errors and return consistent, user-friendly responses while logging details for debugging.

## Code Example

### Custom Error Class
```typescript
// src/lib/errors.ts
export class AppError extends Error {
  public readonly statusCode: number;
  public readonly code: string;
  public readonly isOperational: boolean;

  constructor(
    message: string,
    statusCode: number = 500,
    code?: string,
    isOperational: boolean = true
  ) {
    super(message);
    this.statusCode = statusCode;
    this.code = code || this.generateCode(statusCode);
    this.isOperational = isOperational;

    Error.captureStackTrace(this, this.constructor);
  }

  private generateCode(statusCode: number): string {
    const codes: Record<number, string> = {
      400: 'BAD_REQUEST',
      401: 'UNAUTHORIZED',
      403: 'FORBIDDEN',
      404: 'NOT_FOUND',
      409: 'CONFLICT',
      422: 'UNPROCESSABLE_ENTITY',
      500: 'INTERNAL_ERROR',
    };
    return codes[statusCode] || 'UNKNOWN_ERROR';
  }
}

// Convenience factory functions
export const NotFoundError = (message = 'Resource not found') =>
  new AppError(message, 404, 'NOT_FOUND');

export const BadRequestError = (message: string) =>
  new AppError(message, 400, 'BAD_REQUEST');

export const UnauthorizedError = (message = 'Authentication required') =>
  new AppError(message, 401, 'UNAUTHORIZED');

export const ForbiddenError = (message = 'Access denied') =>
  new AppError(message, 403, 'FORBIDDEN');

export const ConflictError = (message: string) =>
  new AppError(message, 409, 'CONFLICT');
```

### Error Handler Middleware
```typescript
// src/middleware/error-handler.ts
import { Request, Response, NextFunction } from 'express';
import { ZodError } from 'zod';
import { AppError } from '../lib/errors';
import { logger } from '../lib/logger';

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Log error details
  logger.error({
    message: error.message,
    stack: error.stack,
    path: req.path,
    method: req.method,
  });

  // Handle AppError (operational errors)
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      success: false,
      error: {
        message: error.message,
        code: error.code,
      },
    });
  }

  // Handle Zod validation errors
  if (error instanceof ZodError) {
    return res.status(400).json({
      success: false,
      error: {
        message: 'Validation failed',
        code: 'VALIDATION_ERROR',
        details: error.errors.map((e) => ({
          field: e.path.join('.'),
          message: e.message,
        })),
      },
    });
  }

  // Handle Prisma errors
  if (error.name === 'PrismaClientKnownRequestError') {
    const prismaError = error as any;

    if (prismaError.code === 'P2002') {
      return res.status(409).json({
        success: false,
        error: {
          message: 'Resource already exists',
          code: 'DUPLICATE_ENTRY',
        },
      });
    }

    if (prismaError.code === 'P2025') {
      return res.status(404).json({
        success: false,
        error: {
          message: 'Resource not found',
          code: 'NOT_FOUND',
        },
      });
    }
  }

  // Handle unknown errors (don't leak details)
  return res.status(500).json({
    success: false,
    error: {
      message: 'An unexpected error occurred',
      code: 'INTERNAL_ERROR',
    },
  });
}
```

### Async Handler Wrapper
```typescript
// src/lib/async-handler.ts
import { Request, Response, NextFunction, RequestHandler } from 'express';

type AsyncRequestHandler = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<any>;

export function asyncHandler(fn: AsyncRequestHandler): RequestHandler {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Usage:
router.get('/:id', asyncHandler(async (req, res) => {
  const user = await UserService.findById(req.params.id);
  res.json({ success: true, data: user });
}));
```

### Register Middleware
```typescript
// src/app.ts
import express from 'express';
import { errorHandler } from './middleware/error-handler';

const app = express();

// ... routes ...

// Error handler must be last
app.use(errorHandler);
```

## Usage in Services
```typescript
import { NotFoundError, ConflictError } from '../lib/errors';

export class UserService {
  static async findById(id: string) {
    const user = await prisma.user.findUnique({ where: { id } });

    if (!user) {
      throw NotFoundError('User not found');
    }

    return user;
  }

  static async create(data: CreateUserInput) {
    const existing = await prisma.user.findUnique({
      where: { email: data.email },
    });

    if (existing) {
      throw ConflictError('Email already registered');
    }

    return prisma.user.create({ data });
  }
}
```

## Related Patterns

- `sp_rest_001` - REST endpoint structure
- `sp_validation_001` - Request validation

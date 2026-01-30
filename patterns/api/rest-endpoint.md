# Pattern: RESTful Endpoint Structure

**ID**: `sp_rest_001`
**Category**: API
**Confidence**: 91%

## When to Use

- Creating CRUD endpoints
- Building REST APIs
- Any HTTP API endpoint

## Keywords

`rest`, `api`, `endpoint`, `crud`, `route`, `controller`, `express`, `http`

## Approach

Follow REST conventions with proper HTTP methods, status codes, and consistent response format. Separate routes, controllers, and services.

## REST Conventions

| Operation | Method | Path | Success Status |
|-----------|--------|------|----------------|
| List | GET | /resources | 200 |
| Get one | GET | /resources/:id | 200 |
| Create | POST | /resources | 201 |
| Update | PUT/PATCH | /resources/:id | 200 |
| Delete | DELETE | /resources/:id | 204 |

## Code Example

### Route Definition
```typescript
// src/routes/users.ts
import { Router } from 'express';
import { UserController } from '../controllers/user';
import { validateRequest } from '../middleware/validate';
import { createUserSchema, updateUserSchema } from '../schemas/user';
import { requireAuth } from '../middleware/auth';

const router = Router();

router.get('/', requireAuth, UserController.list);
router.get('/:id', requireAuth, UserController.getById);
router.post('/', validateRequest(createUserSchema), UserController.create);
router.patch('/:id', requireAuth, validateRequest(updateUserSchema), UserController.update);
router.delete('/:id', requireAuth, UserController.delete);

export { router as userRoutes };
```

### Controller
```typescript
// src/controllers/user.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '../services/user';

export class UserController {
  static async list(req: Request, res: Response, next: NextFunction) {
    try {
      const { page = 1, limit = 20 } = req.query;
      const result = await UserService.list({ page: +page, limit: +limit });

      res.json({
        success: true,
        data: result.users,
        meta: {
          page: +page,
          limit: +limit,
          total: result.total,
        },
      });
    } catch (error) {
      next(error);
    }
  }

  static async getById(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await UserService.findById(req.params.id);

      res.json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  static async create(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await UserService.create(req.body);

      res.status(201).json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  static async update(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await UserService.update(req.params.id, req.body);

      res.json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  static async delete(req: Request, res: Response, next: NextFunction) {
    try {
      await UserService.delete(req.params.id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}
```

## Response Format

### Success Response
```json
{
  "success": true,
  "data": { },
  "meta": { }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "message": "User not found",
    "code": "USER_NOT_FOUND"
  }
}
```

### List Response with Pagination
```json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "hasMore": true
  }
}
```

## HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Authenticated but not allowed |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 500 | Server Error | Unexpected error |

## Related Patterns

- `sp_validation_001` - Request validation
- `sp_error_001` - Error handling

# Pattern: Database Migrations

**ID**: `sp_migration_001`
**Category**: Database
**Confidence**: 87%

## When to Use

- Any database schema change
- Adding new tables or columns
- Modifying existing schema
- Seeding initial data

## Keywords

`migration`, `database`, `schema`, `prisma`, `alter`, `table`, `column`

## Approach

Use versioned migrations with both up and down migrations. Test migrations on a copy of production data before deploying. Use transactions where supported.

## Code Example (Prisma)

### Schema Definition
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id           String   @id @default(uuid())
  email        String   @unique
  passwordHash String   @map("password_hash")
  name         String
  bio          String?
  role         Role     @default(USER)
  posts        Post[]
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt @map("updated_at")

  @@map("users")
  @@index([email])
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  String   @map("author_id")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("posts")
  @@index([authorId])
  @@index([published, createdAt])
}

enum Role {
  USER
  ADMIN
}
```

### Migration Commands
```bash
# Create migration from schema changes
npx prisma migrate dev --name add_user_bio

# Apply migrations in production
npx prisma migrate deploy

# Reset database (dev only)
npx prisma migrate reset

# Generate client without migration
npx prisma generate
```

### Generated Migration
```sql
-- prisma/migrations/20250130_add_user_bio/migration.sql

-- CreateTable
CREATE TABLE "users" (
    "id" UUID NOT NULL DEFAULT gen_random_uuid(),
    "email" VARCHAR(255) NOT NULL,
    "password_hash" VARCHAR(255) NOT NULL,
    "name" VARCHAR(100) NOT NULL,
    "bio" TEXT,
    "role" "Role" NOT NULL DEFAULT 'USER',
    "created_at" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "users_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "users_email_key" ON "users"("email");
CREATE INDEX "users_email_idx" ON "users"("email");
```

### Data Migration Script
```typescript
// prisma/migrations/scripts/backfill_user_names.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Find users without names (from legacy data)
  const users = await prisma.user.findMany({
    where: { name: '' },
  });

  console.log(`Found ${users.length} users to update`);

  // Update in batches
  const batchSize = 100;
  for (let i = 0; i < users.length; i += batchSize) {
    const batch = users.slice(i, i + batchSize);

    await Promise.all(
      batch.map((user) =>
        prisma.user.update({
          where: { id: user.id },
          data: { name: user.email.split('@')[0] },
        })
      )
    );

    console.log(`Updated ${Math.min(i + batchSize, users.length)}/${users.length}`);
  }
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### Seed Script
```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import { hashPassword } from '../src/lib/auth';

const prisma = new PrismaClient();

async function main() {
  // Create admin user
  const adminPassword = await hashPassword('admin123');

  await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: {
      email: 'admin@example.com',
      passwordHash: adminPassword,
      name: 'Admin',
      role: 'ADMIN',
    },
  });

  console.log('Seed completed');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### Package.json Scripts
```json
{
  "scripts": {
    "db:migrate": "prisma migrate dev",
    "db:migrate:prod": "prisma migrate deploy",
    "db:seed": "ts-node prisma/seed.ts",
    "db:reset": "prisma migrate reset",
    "db:studio": "prisma studio"
  }
}
```

## Migration Best Practices

1. **Always test on copy of prod data** before deploying
2. **Use transactions** for multi-step migrations
3. **Make migrations reversible** when possible
4. **Avoid breaking changes** - add columns as nullable first
5. **Separate schema and data migrations** for large changes
6. **Run migrations in CI** before deployment

## Zero-Downtime Migration Pattern

```sql
-- Step 1: Add nullable column
ALTER TABLE users ADD COLUMN new_field VARCHAR(255);

-- Step 2: Backfill data (application handles both old and new)
UPDATE users SET new_field = compute_value(old_field);

-- Step 3: Make non-nullable (after backfill complete)
ALTER TABLE users ALTER COLUMN new_field SET NOT NULL;

-- Step 4: Remove old column (after app updated)
ALTER TABLE users DROP COLUMN old_field;
```

## Related Patterns

- `sp_prisma_001` - Prisma model structure
- `sp_repository_001` - Repository pattern

---
name: zero-downtime-migrations
description: "Zero-downtime database migration strategies for PostgreSQL. Covers lock timeouts, concurrent index creation, expand-and-contract migrations, and batched backfills."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['database', 'postgresql', 'migrations', 'zero-downtime', 'drizzle', 'prisma']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Zero-Downtime Database Migrations

This skill outlines strategies to run Postgres database migrations on production without causing performance degradation or service downtime.

## Core Rules

### 1. Set Lock Timeout
Migrations that acquire exclusive locks (e.g., adding foreign keys, altering columns) should have a tight lock timeout to avoid queuing requests and blocking the application.
```sql
-- Set lock timeout to 2 seconds
SET lock_timeout = 2000;
```

### 2. Create Indexes CONCURRENTLY
Creating an index normally blocks writes to the table. Always use `CONCURRENTLY`.
```sql
-- Drizzle example (written in migration SQL)
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email ON users(email);
```
*Note: CREATE INDEX CONCURRENTLY cannot run inside a transaction. Disable ORM transactions for this migration.*

### 3. The Expand and Contract Pattern
Never perform a breaking change (like renaming or deleting a column) in a single deployment. Split it into three stages:
1. **Expand**: Add the new column/table. Update the application to write to both the old and new locations, but read from the old.
2. **Backfill**: Migrate historical data in batches (not a single giant query to prevent table lock and replication lag).
3. **Contract**: Update the application to read and write only to the new location. Remove the old column/table in a later deployment.

### 4. Adding NOT NULL Columns safely
Adding a `NOT NULL` column to a populated table fails unless it has a default. For complex scenarios:
1. Add the column as nullable.
2. Backfill default values to all rows in small batches.
3. Alter the column to `NOT NULL` with a lock timeout.

### 5. Example: Batched Backfill Script
```typescript
async function backfill() {
  const BATCH_SIZE = 1000;
  let cursor = 0;
  while (true) {
    const updated = await db.execute(sql`
      UPDATE users 
      SET active_new = true 
      WHERE id IN (
        SELECT id FROM users 
        WHERE id > ${cursor} AND active_new IS NULL 
        ORDER BY id 
        LIMIT ${BATCH_SIZE}
      )
      RETURNING id
    `);
    if (updated.length === 0) break;
    cursor = updated[updated.length - 1].id;
  }
}
```

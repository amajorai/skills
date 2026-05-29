---
name: db-migrate
description: Run a database migration safely in production. Covers dry-run, rollback plan, zero-downtime patterns, and verification. Use before applying any schema change to a live database with real user data.
argument-hint: <migration description or migration file>
---

# DB Migrate

You are running a production database migration. Safety first: data loss is not recoverable. Work through each phase in order.

**Migration:** {{args}}

If `{{args}}` is empty, ask the user which migration file or schema change to run before proceeding.


## Phase 1: Understand the Migration

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Read the migration file(s): what exactly does this change? Tables added/dropped? Columns added/altered/dropped? Indexes? Constraints? |
| 2 | Production data scale: how many rows in affected tables? Any tables > 1M rows? Foreign key relationships? |

Assess risk level:
- **Low**: Adding nullable columns, adding indexes (with `CONCURRENTLY`), creating new tables
- **Medium**: Adding NOT NULL columns with defaults, renaming columns with a deprecation period
- **High**: Dropping columns/tables, changing column types, removing constraints, backfilling large tables


## Phase 2: Interview

Ask the user (combine related questions):

- **Backup**: Is there a recent backup? When was the last successful restore test?
- **Downtime**: Is a maintenance window acceptable, or must this be zero-downtime?
- **Rollback plan**: If the migration fails halfway, what is the recovery procedure?
- **Traffic**: What is current traffic? Is there a low-traffic window?

Do not proceed to Phase 3 without confirming a backup exists.


## Phase 3: Dry Run

Run the migration against a staging or development database that mirrors production.

Drizzle Kit has no `--dry-run` flag. To preview, generate the SQL with `npx drizzle-kit generate` and read the generated file in `drizzle/` before applying. Then apply against staging by pointing your config at the staging database:

On Windows, run the bash snippets in this file via the Bash tool or Git Bash. PowerShell and cmd do not accept inline `VAR=value` prefixes, `$(...)` substitution, `/dev/null` redirects, or unix `head` pipes.

```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)

# Drizzle: generate SQL to inspect, then apply to staging
npx drizzle-kit generate
DATABASE_URL="$STAGING_DATABASE_URL" npx drizzle-kit migrate

# Prisma: apply pending migrations to staging
DATABASE_URL="$STAGING_DATABASE_URL" npx prisma migrate deploy

# Raw SQL: review the file first, then apply
psql "$STAGING_DATABASE_URL" < migration.sql
```

Verify:
- [ ] Migration applies without errors on staging
- [ ] Application still starts and all health checks pass after migration
- [ ] Key queries still work (spot-check 3–5 representative queries)
- [ ] Migration duration recorded (this is the expected duration in production)

If staging fails, stop. Fix the migration and repeat.


## Phase 4: Zero-Downtime Patterns (if required)

For high-traffic databases, use expand-contract:

**Adding a NOT NULL column**:
1. Add column as NULLABLE first (deploy to prod)
2. Backfill existing rows in batches: `UPDATE ... WHERE id > $last_id LIMIT 1000`
3. Add NOT NULL constraint once all rows are filled (deploy to prod)
4. Remove old column (if applicable) in a later migration

**Renaming a column**:
1. Add the new column (deploy to prod, write to both)
2. Backfill new column from old column
3. Switch reads to new column (deploy to prod)
4. Drop old column (deploy to prod after confirming no reads)

**Large table index**:
```sql
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
-- CONCURRENTLY: takes longer but doesn't lock the table
```

**Dropping a table**:
1. Remove all application code that references the table (deploy to prod)
2. Confirm no queries hit the table for 24 hours (check logs)
3. Drop the table


## Phase 5: Backup Verification

Before running in production:

```bash
# Verify backup exists and is recent
# For PostgreSQL, use the custom format (-Fc) so pg_restore can read it:
BACKUP_FILE="backup_pre_migration_$(date +%Y%m%d_%H%M%S).dump"
pg_dump -Fc "$PRODUCTION_DATABASE_URL" -f "$BACKUP_FILE"

# Verify the backup is readable and non-empty
pg_restore --list "$BACKUP_FILE" | head -20
```

(If you prefer a plain `.sql` dump, omit `-Fc`/`-f` and redirect to a `.sql` file, but then verify with `head` instead of `pg_restore`, which only reads custom/directory/tar archives.)

Do not proceed without a fresh backup taken within the last hour.


## Phase 6: Production Run

Run the migration with monitoring:

1. Open the database monitoring dashboard
2. Run the migration:
   ```bash
   npx drizzle-kit migrate   # or your ORM's migrate command
   ```
3. Watch for: lock waits, connection spikes, query time increases
4. Verify immediately after:
   - Application health endpoint returns 200
   - Key user flows work (login, core action)
   - Error rate unchanged in Sentry/logs


## Phase 7: Rollback Procedure

If the migration causes problems:

**Note:** Drizzle Kit has no built-in rollback command, and `prisma migrate deploy` only rolls forward. Neither ORM auto-reverts an applied migration. You must roll back manually with the inverse SQL or by restoring from backup.

**Manual rollback** (prepare this BEFORE running):
- Write the inverse SQL before starting
- For DROP TABLE migrations: restore from backup (there is no other way)
- For ALTER TABLE migrations: `ALTER TABLE ... DROP COLUMN` or reverse the change

Document the rollback script and verify it works on staging before going to production.


## Phase 8: Post-Migration Verification

- [ ] Application running with no elevated error rate (check 15 min post-migration)
- [ ] Affected queries performing as expected (no new slow queries)
- [ ] New columns/tables populated correctly
- [ ] No orphaned data from dropped constraints
- [ ] Backup retained for at least 7 days post-migration


## Completion Report

- Migration applied (timestamp, duration)
- Tables/columns affected
- Zero-downtime strategy used (if applicable)
- Post-migration verification: pass/fail
- Rollback script location

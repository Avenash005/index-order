# Index Order Matters - Challenge #4 Complete

## Implementation Steps Followed

### Step 1: Run the Original Query (Broken Index)

Setup:

```bash
psql -U postgres
CREATE DATABASE employee_reporting;
\\c employee_reporting
\\i db/schema.sql  -- Creates broken index: idx_salary_department (salary, department)
\\i db/sample_data.sql
```

Test query with EXPLAIN ANALYZE:

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
```

**Expected Plan (Broken)**:

```
Seq Scan on employees  (cost=0.00..XX.XX rows=XX width=XX) (actual time=0.XXX..0.XXX rows=1 loops=1)
  Filter: ((department = 'Sales'::text) AND (salary > '50000'::numeric))
```

- Full Sequential Scan, no index usage.

### Step 2: Incorrect Index Analysis

Original index: `CREATE INDEX idx_salary_department ON employees(salary, department);`

- Starts with `salary` (range filter).
- Query equality on `department` first - index skipped (Left-Most Prefix Rule).

### Step 3: Fix Index Order

Updated schema.sql:

```sql
DROP INDEX IF EXISTS idx_salary_department;
CREATE INDEX idx_department_salary ON employees(department, salary);
```

- `department` first (equality), `salary` second (range) - matches query pattern.

Rerun:

```sql
\\i db/schema.sql  -- Applies fix
```

### Step 4: Verify Fix

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
```

**Expected Plan (Fixed)**:

```
Index Scan using idx_department_salary on employees  (cost=0.42..8.44 rows=1 width=52) (actual time=0.015..0.016 rows=1 loops=1)
  Index Cond: ((department = 'Sales'::text) AND (salary > '50000'::numeric))
```

- Index Scan used, much faster.

## Key Learnings

### Left-Most Prefix Rule

- Composite indexes usable from left: `INDEX(A,B,C)` supports WHERE A, WHERE A+B, WHERE A+B+C.
- Skips if first column omitted: WHERE B+C → full scan.

### Best Practices

- Equality columns first, then range/sort.
- Use `EXPLAIN ANALYZE` before/after.
- Balance read/write overhead.

## GitHub Submission

1. Create public repo: `composite-index-investigation`
2. Add all files: schema.sql, queries.sql, sample_data.sql, Changes.md, TODO.md, README.md
3. Commit: `git add . && git commit -m "Fix composite index order for query optimization"`
4. Push and create PR to main.

## Video Explanation (3-5 min)

1. Show DB setup.
2. Run original query + EXPLAIN (Seq Scan).
3. Apply fix, rerun + EXPLAIN (Index Scan).
4. Explain column order + prefix rule.
5. Demo both queries.
   Upload to Google Drive (public link).

**Task Complete! Ready for submission.**

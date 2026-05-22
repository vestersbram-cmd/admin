---
name: sql-admin
description: PostgreSQL database operations using sql_admin package. Query data, get connections/pools, inspect schemas, and perform DDL operations.
---
# PostgreSQL Database Administration (sql_admin)

## When This Skill Activates

Use this skill when the user:
- Needs to connect to a database (PostgreSQL is the default)
- Wants to query or retrieve data from database tables
- Wants to inspect database schema (tables, columns, indexes, constraints)
- Needs to perform DDL operations (rename, truncate, drop, copy tables)
- Asks about database connection pooling
- Needs to generate CREATE TABLE DDL from existing tables
- Mentions any database, table, or SQL operation (PostgreSQL is assumed by default)

**IMPORTANT:** Always use the `sql_admin_tool` for database operations. Do NOT use command-line tools like `psql`, `mysql`, or `sqlite3`.

**CRITICAL:** Use the `sql_admin_tool` to create connection pools. The tool uses a hardcoded server name `postgres-server-mapping` and requires you to specify the `db_name` parameter:

1. Call the tool directly: `sql_admin_tool(db_name="your_database_name")`
2. The tool returns a connection pool object that you can use in Python code
3. For advanced operations (schema introspection, DDL, etc.), use `execute_code` with the sql_admin package functions after getting a pool

DO NOT use this skill for:
- SQLite operations (only when explicitly mentioned as SQLite)
- GUI-based database tools

---

## Prerequisites

### 1. Install the package

```bash
pip install sql_admin
```

### 2. Configure database connection

Create or edit `sql_settings.json` in the sql_admin package directory:

```json
{
  "my_server": {
    "host": "localhost",
    "port": 5432,
    "user": "postgres",
    "password": "your_password",
    "disabled": false
  }
}
```

**Configuration location:** The file is at the package root (where sql_admin is installed).

---

## Getting Connections

### Option 1: Single Connection (quick_connect)

Use for one-off queries or short-lived operations:

```python
from sql_admin.database.db import quick_connect

# Connect to a specific database
conn = await quick_connect(server_name="my_server", db_name="mydb")

# Use the connection
result = await conn.fetch("SELECT * FROM users LIMIT 10")

# Close when done
await conn.close()
```

**Parameters:**
- `server_name`: Name from `sql_settings.json` (required)
- `db_name`: Database name (defaults to "postgres")
- `localhost`: Override host to localhost (bool, default False)

### Option 2: Connection Pool (quick_pool)

Use for repeated operations or long-running processes:

```python
from sql_admin.database.db import quick_pool

# Create a connection pool
pool = await quick_pool(
    server_name="my_server",
    db_name="mydb",
    min_size=1,      # Minimum connections (default: 1)
    max_size=8       # Maximum connections (default: 8)
)

# Use the pool
async with pool.acquire() as conn:
    result = await conn.fetch("SELECT * FROM users")

# Pool is automatically managed
```

**Parameters:**
- `server_name`: Name from `sql_settings.json` (required)
- `db_name`: Database name (defaults to "postgres")
- `localhost`: Override host to localhost (bool, default False)
- `min_size`: Minimum pool size (default: 1)
- `max_size`: Maximum pool size (default: 8)

### Option 3: Quick Query (quick_connect + auto-close)

For simple one-off queries without manual connection management:

```python
from sql_admin.database.db import quick_query

results = await quick_query(
    query="SELECT * FROM users WHERE id = $1",
    server_name="my_server",
    db_name="mydb",
    args=(123,)
)
```

**Parameters:**
- `query`: SQL query string (required)
- `server_name`: Name from `sql_settings.json` (required)
- `db_name`: Database name (defaults to "postgres")
- `localhost`: Override host to localhost (bool, default False)
- `args`: Query parameters as tuple (optional)

---

## Querying Data

### Basic Queries with quick_query

For simple SELECT queries without connection management:

```python
from sql_admin.database.db import quick_query

# Select all rows
results = await quick_query(
    query="SELECT * FROM bank_accounts LIMIT 1",
    server_name="my_server",
    db_name="bank"
)

# Select with parameters
results = await quick_query(
    query="SELECT * FROM users WHERE id = $1",
    server_name="my_server",
    db_name="mydb",
    args=(123,)
)

# Process results
for row in results:
    print(row)  # asyncpg.Record object
    print(row["column_name"])  # Access by column name
```

### Queries with Connection Pool

For repeated queries or complex operations:

```python
from sql_admin.database.db import quick_pool

pool = await quick_pool(server_name="my_server", db_name="bank")

async with pool.acquire() as conn:
    # Single query
    result = await conn.fetchrow("SELECT * FROM bank_accounts WHERE id = $1", 1)
    print(result)
    
    # Multiple rows
    results = await conn.fetch("SELECT * FROM bank_accounts LIMIT 10")
    for row in results:
        print(row["column_name"])
    
    # Execute without returning data (INSERT, UPDATE, DELETE)
    await conn.execute("UPDATE bank_accounts SET balance = 100 WHERE id = 1")
```

### Query with Connection

For single operations where you need more control:

```python
from sql_admin.database.db import quick_connect

conn = await quick_connect(server_name="my_server", db_name="bank")

try:
    result = await conn.fetchrow("SELECT * FROM bank_accounts LIMIT 1")
    print(result)
finally:
    await conn.close()
```

---

## Schema Introspection

### List Tables

```python
from sql_admin.database.schema import list_tables_in_schema

tables = await list_tables_in_schema(server="my_server", db="mydb", schema="public")
# Returns: ["users", "orders", "products", ...]
```

### Get Table Columns

```python
from sql_admin.database.schema import get_table_columns

columns = await get_table_columns(
    server="my_server",
    db="mydb",
    schema="public",
    table="users"
)
# Returns: [
#   {"column_name": "id", "data_type": "integer", "is_nullable": "NO", "column_default": "nextval(...)", "is_pk": true},
#   {"column_name": "email", "data_type": "text", "is_nullable": "NO", "column_default": None, "is_pk": false},
#   ...
# ]
```

### Get Table Indexes

```python
from sql_admin.database.schema import get_table_indexes

indexes = await get_table_indexes(
    server="my_server",
    db="mydb",
    schema="public",
    table="users"
)
# Returns: [{"name": "users_email_idx", "definition": "CREATE UNIQUE INDEX ...", "is_unique": true, "columns": ["email"]}, ...]
```

### Get Table Constraints

```python
from sql_admin.database.schema import get_table_constraints

constraints = await get_table_constraints(
    server="my_server",
    db="mydb",
    schema="public",
    table="users"
)
# Returns: [{"name": "users_pkey", "type": "PRIMARY KEY", "definition": "PRIMARY KEY (id)"}, ...]
```

### List Databases

```python
from sql_admin.database.schema import list_databases

databases = await list_databases(server="my_server")
# Returns: ["postgres", "mydb", "testdb", ...]
```

### Generate CREATE TABLE DDL

```python
from sql_admin.database.schema import generate_create_table_ddl

ddl = await generate_create_table_ddl(
    server="my_server",
    db="mydb",
    schema="public",
    table="users"
)
# Returns: 'CREATE TABLE "public"."users" (\n  "id" integer DEFAULT nextval(...),\n  ...);'
```

---

## DDL Operations

### Rename Table

```python
from sql_admin.database.modify import rename_table

await rename_table(
    server="my_server",
    db="mydb",
    schema="public",
    old_name="users_old",
    new_name="users"
)
```

### Truncate Table

```python
from sql_admin.database.modify import truncate_table

await truncate_table(
    server="my_server",
    db="mydb",
    schema="public",
    table="users"
)
```

### Drop Table

```python
from sql_admin.database.modify import drop_table

await drop_table(
    server="my_server",
    db="mydb",
    schema="public",
    table="users_temp"
)
```

### Copy Table

```python
from sql_admin.database.modify import copy_table

# Copy with data
await copy_table(
    server="my_server",
    db="mydb",
    schema="public",
    table="users",
    new_table_name="users_backup",
    with_data=True
)

# Copy structure only
await copy_table(
    server="my_server",
    db="mydb",
    schema="public",
    table="users",
    new_table_name="users_template",
    with_data=False
)

# Copy to different database
await copy_table(
    server="my_server",
    db="mydb",
    schema="public",
    table="users",
    new_table_name="users",
    with_data=True,
    target_db="mydb_backup"
)
```

### Reorder Columns

```python
from sql_admin.database.modify import reorder_columns

await reorder_columns(
    server="my_server",
    db="mydb",
    schema="public",
    table="users",
    new_column_order=["id", "email", "name", "created_at"]
)
```

---

## Singleton Connection Manager

For persistent pools across operations:

```python
from sql_admin.database.db import DBConnectionManager

# Get or create a pool (cached)
pool = await DBConnectionManager.get_pool(server="my_server", db="mydb")

# Use the pool
async with pool.acquire() as conn:
    result = await conn.fetch("SELECT * FROM users")
```

The singleton ensures the same pool is reused for the same server/database combination.

---

## Common Patterns

### Pattern 1: Inspect Schema Before Operation

```python
from sql_admin.database.schema import get_table_columns
from sql_admin.database.db import quick_pool

# Check column exists before query
columns = await get_table_columns(server="my_server", db="mydb", schema="public", table="users")
col_names = [c["column_name"] for c in columns]

if "email" in col_names:
    pool = await quick_pool(server_name="my_server", db_name="mydb")
    async with pool.acquire() as conn:
        result = await conn.fetch("SELECT email FROM users")
```

### Pattern 2: Generate DDL for Documentation

```python
from sql_admin.database.schema import generate_create_table_ddl

# Generate DDL for all tables
tables = await list_tables_in_schema(server="my_server", db="mydb", schema="public")
for table in tables:
    ddl = await generate_create_table_ddl(server="my_server", db="mydb", schema="public", table=table)
    print(f"-- {table}\n{ddl}\n")
```

### Pattern 3: Safe Table Copy with Verification

```python
from sql_admin.database.modify import copy_table
from sql_admin.database.schema import get_table_columns

# Copy table
await copy_table(server="my_server", db="mydb", schema="public", table="users", new_table_name="users_backup", with_data=True)

# Verify copy
original_cols = await get_table_columns(server="my_server", db="mydb", schema="public", table="users")
backup_cols = await get_table_columns(server="my_server", db="mydb", schema="public", table="users_backup")

assert len(original_cols) == len(backup_cols), "Column count mismatch"
```

---

## Pitfalls

| Issue | Cause | Fix |
|-------|-------|-----|
| Connection fails | `sql_settings.json` missing or invalid | Ensure file exists at package root with valid JSON |
| Server not found | Wrong `server_name` in function call | Match server_name to key in `sql_settings.json` |
| Pool exhaustion | Too many concurrent operations | Increase `max_size` in `quick_pool()` |
| Cache stale after DDL | Metadata not invalidated | DDL operations auto-invalidate cache; manual ops need `invalidate_table_cache()` |

---

## Notes

- All operations are async/await based
- PostgreSQL is the primary target (SQLite support is limited)
- Connection pooling uses `asyncpg` for performance
- Schema introspection uses PostgreSQL's `information_schema`
- DDL operations automatically invalidate table cache
- Use `quick_connect` for one-off operations, `quick_pool` for repeated operations
Here is a **complete and practical PostgreSQL guide** on how to create a **new database user with permissions limited to a specific database**. This is suitable for production or development environments where access control is important.

---

# 📘 PostgreSQL User & Permissions Setup Guide

## 🎯 Goal:

You want to create a user who:

* Can **connect to a specific database**
* Can **read and write data** (SELECT, INSERT, UPDATE, DELETE)
* Cannot modify schemas or drop tables
* Has no access to other databases

---

## 🧱 Step-by-Step Instructions

### ✅ 1. Login as `postgres` (superuser)

In Command Prompt or terminal:

```bash
psql -U postgres
```

Or for a specific database:

```bash
psql -U postgres -d mydb
```

---

### ✅ 2. Create the user

```sql
CREATE USER evan WITH PASSWORD 'your_secure_password';
```

You can verify with:

```sql
\du
```

---

### ✅ 3. Grant access to a specific database

```sql
GRANT CONNECT ON DATABASE mydb TO evan;
```

This allows the user to connect but not use tables yet.

---

### ✅ 4. Grant access to the schema (default is `public`)

```sql
\c mydb
GRANT USAGE ON SCHEMA public TO evan;
```

---

### ✅ 5. Grant permissions on existing tables

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO evan;
```

> 📝 This grants read/write on all current tables in the `public` schema of `mydb`.

---

### ✅ 6. Grant permissions on future tables

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO evan;
```

> 🧠 This ensures future tables created in `public` will be accessible to `evan`.

---

### ✅ 7. (Optional) Grant access to sequences (if using `SERIAL` / `BIGSERIAL` IDs)

```sql
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO evan;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO evan;
```

---

### ✅ 8. Verify privileges

To see table access:

```sql
\z users
```

To list all privileges:

```sql
\dp
```

---

## 🛑 Optional: Revoke permissions if needed

Revoke specific permissions:

```sql
REVOKE DELETE ON TABLE users FROM evan;
```

Revoke all:

```sql
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM evan;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM evan;
```

---

## 🧪 Example Test as `evan`

```bash
psql -U evan -d mydb
```

Then test:

```sql
SELECT * FROM users;

INSERT INTO users(name, email) VALUES ('Test User', 'test@example.com');
```

---

## 🧾 Full Example Script (copy-paste)

```sql
-- Login as superuser (postgres)
CREATE USER evan WITH PASSWORD 'evan123';

GRANT CONNECT ON DATABASE mydb TO evan;

\c mydb

GRANT USAGE ON SCHEMA public TO evan;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO evan;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO evan;

GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO evan;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO evan;
```

---

## 🧠 Summary

| Step | Action                                       |
| ---- | -------------------------------------------- |
| 1    | Create user                                  |
| 2    | Grant connect to DB                          |
| 3    | Grant usage on schema                        |
| 4    | Grant table access                           |
| 5    | Grant future table access                    |
| 6    | Grant sequence access                        |
| 7    | Verify with `\z` or test with `psql -U user` |

---


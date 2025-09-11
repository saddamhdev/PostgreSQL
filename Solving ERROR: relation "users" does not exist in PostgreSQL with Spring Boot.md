Got it 👍 — let me prepare a **step-by-step documentation** for solving the error you faced:

---

# 📄 Documentation: Solving `ERROR: relation "users" does not exist` in PostgreSQL with Spring Boot

---

## 🔍 Problem

When running your Spring Boot project, you saw:

```
org.postgresql.util.PSQLException: ERROR: relation "users" does not exist
```

This means that Hibernate/JPA tried to query the `users` table, but PostgreSQL couldn’t find it.

---

## ✅ Root Causes

1. **Database schema mismatch**
   The table was not created in the target database (`ssc_data`).

2. **Wrong role/privileges**
   The role `ssc_data` had no ownership rights, so tables were never created.

3. **Hibernate DDL setting**
   `spring.jpa.hibernate.ddl-auto` was not configured to create/update tables.

4. **Schema mismatch**
   Tables were created in `postgres` database or owned by another role (`postgres`).

---

## 🛠️ Solution Steps

### 1. Verify connection

Connect as the Spring Boot role:

```bash
sudo -u postgres /www/server/pgsql/bin/psql -h 127.0.0.1 -U ssc_data -d ssc_data
```

Check current database:

```sql
\c ssc_data
```

---

### 2. Check tables

Run:

```sql
\dt
```

If `Did not find any relations.` → No tables exist for this database.

---

### 3. Fix privileges

Ensure the app user (`ssc_data`) owns the database:

```sql
ALTER DATABASE ssc_data OWNER TO ssc_data;
GRANT ALL PRIVILEGES ON DATABASE ssc_data TO ssc_data;
```

If you created the DB as `postgres`, transfer ownership:

```sql
REASSIGN OWNED BY postgres TO ssc_data;
```

---

### 4. Configure Spring Boot

In your `application.properties` (or YAML):

```properties
spring.datasource.url=jdbc:postgresql://159.89.172.251:5432/ssc_data
spring.datasource.username=ssc_data
spring.datasource.password=your_password_here

spring.jpa.hibernate.ddl-auto=update   # options: create, update, validate, none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

* `update` → creates missing tables without dropping data
* `create` → drops and recreates schema every run (good for testing only)

---

### 5. Restart application

Run Spring Boot again.
Hibernate will log:

```
Hibernate: create table users (...)
```

---

### 6. Validate

Check tables again:

```sql
\c ssc_data
\dt
```

You should now see `users`, `student_submissions`, etc.

---

### ⚡ Pro Tips

* **Schema drift**: If some tables were created under `postgres` instead of `ssc_data`, reassign:

```sql
REASSIGN OWNED BY postgres TO ssc_data;
```

* **Backup first**: Always `pg_dump` your database before using `create` (it wipes data).
* **Security**: Never use `trust` in `pg_hba.conf` for production. Use `md5` with strong password.

---

## 🎯 Final Expected State

* `ssc_data` owns the database and all tables.
* Spring Boot auto-creates the schema.
* Running `\dt` inside `ssc_data` shows your JPA entities (e.g., `users`).

---

👉 Do you want me to also draft a **ready rollback/repair SQL script** (transfer existing tables to `ssc_data` ownership), so even if Hibernate created them under `postgres`, you can fix ownership in one go?

```sql
REASSIGN OWNED BY postgres TO ssc_data;

-- Give ssc_data full rights on its DB
GRANT ALL PRIVILEGES ON DATABASE ssc_data TO ssc_data;

-- Make sure it can create schemas/tables
ALTER USER ssc_data CREATEDB;
ALTER USER ssc_data CREATEROLE;

-- Optional but helpful for Hibernate
ALTER DATABASE ssc_data OWNER TO ssc_data;
```

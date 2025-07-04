The error you're seeing is:

```
ERROR: must be owner of table users
```

### 🔎 Why this happens:

Hibernate is trying to **alter the `users` table** to add new columns (`details` and `tags`). But the current PostgreSQL user your Spring Boot app is using (probably `evan`) **is not the owner of the `users` table**, so PostgreSQL blocks the schema update.

---

### ✅ Solution Options:

#### 🔧 Option 1: Manually add the columns (recommended for restricted users)

Login as a privileged user (e.g., `postgres`) and run:

```sql
ALTER TABLE users ADD COLUMN details jsonb;
ALTER TABLE users ADD COLUMN tags text[];
```

> ✅ After this, Hibernate won’t try to alter the table again, and the app will work as expected.

---

#### 🔒 Option 2: Change Spring Boot to avoid DDL auto-execution

In `application.properties`:

```properties
spring.jpa.hibernate.ddl-auto=none
```

This disables Hibernate’s automatic schema generation. You manage the database schema manually.

---

#### 🛠️ Option 3: Change table ownership (not recommended unless you trust the user)

If you **must** allow `evan` to modify schema:

```sql
-- Logged in as postgres
ALTER TABLE users OWNER TO evan;
```

> ⚠️ This gives `evan` full control of the table — dangerous in production or multi-user environments.

---

### ✅ Recommendation for You:

Since you're using `evan` as a restricted user:

* Add the columns yourself via `psql` or `pgAdmin` using the `postgres` user (Option 1).
* Disable schema auto-modification by setting `spring.jpa.hibernate.ddl-auto=none`.

---


Great — let’s create **a clean new PostgreSQL database, user, password, and permissions** for your Spring Boot R2DBC project.

Follow EXACT commands below inside **psql**.

---

# ✅ **1. Create User**

```sql
CREATE USER icsquiz WITH PASSWORD 'wNAmjmnGkCBnXX7ABZE';
```

---

# ✅ **2. Create Database**

```sql
CREATE DATABASE icsquiz OWNER icsquiz;
```

---

# ✅ **3. Grant Permissions**

Connect to the new database:

```bash
\c icsquiz
```

Now grant privileges:

```sql
GRANT ALL PRIVILEGES ON DATABASE icsquiz TO icsquiz;
GRANT ALL PRIVILEGES ON SCHEMA public TO icsquiz;
```

Allow the user to create tables:

```sql
ALTER SCHEMA public OWNER TO icsquiz;
```

---

# ✅ **4. Allow Spring Boot R2DBC to create tables**

```sql
GRANT CREATE, USAGE ON SCHEMA public TO icsquiz;
```

---

# ⚠️ **5. Very Important (R2DBC needs this)**

PostgreSQL 16 sometimes blocks privilege inheritance.

Make sure user owns future tables:

```sql
ALTER DEFAULT PRIVILEGES FOR USER icsquiz IN SCHEMA public
GRANT ALL PRIVILEGES ON TABLES TO icsquiz;

ALTER DEFAULT PRIVILEGES FOR USER icsquiz IN SCHEMA public
GRANT ALL PRIVILEGES ON SEQUENCES TO icsquiz;
```

---

# 🚀 Now verify

```sql
\dt
```

Your Spring Boot should now create tables without:

❌ permission denied for schema public

---

# ⭐ Final Spring Boot application.properties

```
spring.r2dbc.url=r2dbc:postgresql://159.89.172.251:5432/icsquiz
spring.r2dbc.username=icsquiz
spring.r2dbc.password=wNAmGkCBnXXjj7ABZE
```

---

If you want, I can give you **a full VPS PostgreSQL security configuration** (pg_hba.conf + postgresql.conf) optimised for R2DBC as well.

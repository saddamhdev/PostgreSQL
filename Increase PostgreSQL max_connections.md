Below is a **complete, clean, step-by-step documentation** written professionally on **how to set PostgreSQL max_connections in your VPS** — **exactly matching your setup (aaPanel / BaoTa PostgreSQL at /www/server/pgsql).**

You can use this as team documentation or internal DevOps notes.

---

# 📘 **Documentation: How to Increase PostgreSQL `max_connections` on VPS (aaPanel / BaoTa PGSQL Installation)**

This guide explains how to safely update PostgreSQL’s `max_connections` on a VPS where PostgreSQL is installed under:

```
/www/server/pgsql/
```

This path indicates a **BaoTa Panel (aaPanel)** installation, which uses custom directories and manual process management (NOT systemctl / NOT Docker / NOT apt).

---

# 📍 **1. Verify PostgreSQL Installation Path**

Run:

```bash
ls /www/server/pgsql
```

You should see directories like:

```
bin/  data/  logs/  etc...
```

* **Binary path:** `/www/server/pgsql/bin/`
* **Data path:** `/www/server/pgsql/data/`
* **Config files:** Located inside data directory.

---

# 📍 **2. Locate PostgreSQL Configuration File**

Run:

```bash
/www/server/pgsql/bin/psql -U postgres -c "SHOW config_file;"
```

Expected output:

```
/www/server/pgsql/data/postgresql.conf
```

This is the file we must edit.

---

# 📍 **3. Edit postgresql.conf**

Open the file:

```bash
nano /www/server/pgsql/data/postgresql.conf
```

Find:

```
max_connections = 100
```

Change it to your desired value (based on RAM):

| VPS RAM | Recommended max_connections |
| ------- | --------------------------- |
| 2 GB    | 200                         |
| 4 GB    | 300                         |
| 8 GB    | 500–800                     |
| 16 GB   | 1000–1500                   |

Example:

```
max_connections = 500
```

Optional but recommended:

```
shared_buffers = 1GB
```

---

# 📍 **4. STOP PostgreSQL (BaoTa style)**

aaPanel starts PostgreSQL manually using `pg_ctl`.

Stop it safely:

```bash
su - postgres -c "/www/server/pgsql/bin/pg_ctl stop -D /www/server/pgsql/data -m fast"
```

or:

```bash
sudo -u postgres /www/server/pgsql/bin/pg_ctl stop -D /www/server/pgsql/data -m fast
```

You should see:

```
server stopped
```

Verify:

```bash
ps aux | grep postgres
```

Only grep should appear.

---

# 📍 **5. START PostgreSQL (BaoTa style)**

Start the server:

```bash
sudo -u postgres /www/server/pgsql/bin/pg_ctl start -D /www/server/pgsql/data
```

Output:

```
LOG: redirecting log output...
server started
```

---

# 📍 **6. Verify New max_connections Value**

Run:

```bash
/www/server/pgsql/bin/psql -U postgres -c "SHOW max_connections;"
```

If successful, you will see:

```
 max_connections 
-----------------
 500
```

🎉 Your PostgreSQL is now using the updated connection limit.

---

# 📍 **7. Validate Service Health**

Check running processes:

```bash
ps aux | grep postgres
```

You should see:

* main postgres master
* logger
* walwriter
* autovacuum
* logical replication
* background writer

Example:

```
postgres: checkpointer
postgres: walwriter
postgres: autovacuum launcher
```

---

# 📍 **8. Troubleshooting**

### ✔ Error: “config_file not found”

Run:

```bash
find /www/server/pgsql -name "postgresql.conf"
```

### ✔ Error: “permission denied”

Run:

```bash
chmod 700 /www/server/pgsql/data
chown -R postgres:postgres /www/server/pgsql/data
```

### ✔ Changes not applied after restart

You restarted BT Panel instead of PostgreSQL.

Use:

```bash
pg_ctl stop/start
```

as described above.

---

# 🎯 **FULL PROCESS SUMMARY (copy-paste quick start)**

```
# 1. Edit config
nano /www/server/pgsql/data/postgresql.conf
# change max_connections

# 2. Stop PostgreSQL
sudo -u postgres /www/server/pgsql/bin/pg_ctl stop -D /www/server/pgsql/data -m fast

# 3. Start PostgreSQL
sudo -u postgres /www/server/pgsql/bin/pg_ctl start -D /www/server/pgsql/data

# 4. Verify
/www/server/pgsql/bin/psql -U postgres -c "SHOW max_connections;"
```

---

# 🚀 **Want me to generate a fully optimized `postgresql.conf` for high concurrency (icsQuiz 50k users)?**

Just say:

➡ **"Optimize postgresql.conf for high load"**

I will generate a tuned config for your exact VPS specs.

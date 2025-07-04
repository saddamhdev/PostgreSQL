
 <project>

     C:\Users\01957>psql -U evan -d mydb
    Password for user evan:
    
    psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  password authentication failed for user "evan"
    
    C:\Users\01957>psql -U evan -d mydb
    Password for user evan:
    
    psql (17.5)
    WARNING: Console code page (437) differs from Windows code page (1252)
             8-bit characters might not work correctly. See psql reference
             page "Notes for Windows users" for details.
    Type "help" for help.
    
    mydb=> INSERT INTO users(name, email) VALUES ('Test User', 'test@example.com');
    ERROR:  permission denied for sequence users_id_seq
    mydb=>
 </project>


Thanks — you're successfully connected as `evan`, but still getting:

```
ERROR: permission denied for sequence users_id_seq
```

That confirms `evan` has permission to insert into the `users` table but **not to use the sequence** that auto-generates the `id`.

---

## ✅ Final Fix: Grant sequence permission

Log back in as **postgres**:

```bash
psql -U postgres -d mydb
```

Then run:

```sql
GRANT USAGE, SELECT ON SEQUENCE users_id_seq TO evan;
```

✅ This will let `evan` use the `id` auto-increment mechanism.

---

## 🔁 Optional: For all sequences in `public` schema

If you want to future-proof access for `evan`, run:

```sql
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO evan;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT ON SEQUENCES TO evan;
```

---

## ✅ After That

Log back in as `evan` and try again:

```sql
INSERT INTO users(name, email) VALUES ('Test User', 'test@example.com');
```

🎉 This should now insert successfully.

---

Let me know if you'd like a full `.sql` script that sets up:

* a user
* access to a specific DB
* table + sequence permissions
* future-proof default privileges

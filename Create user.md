Thanks for sharing the output!

Here's what's happening:

---

### 🧾 Summary of What You Have

#### ✅ Databases:

You currently have these databases:

* `mydb`
* `mydb2`
* `mydb3`
* `postgres` (default)
* `template0`, `template1` (system templates)

#### ✅ Roles:

You only have **one user/role** in the system:

* `postgres` — superuser

There is **no user named `snvn`**. That's why your connection failed:

> `FATAL: password authentication failed for user "snvn"`

---

### ✅ Fix: Create the `snvn` User

If you want to log in or connect in **pgAdmin** or your app as `snvn`, you must **create that user** first.

Run this inside `psql`:

```sql
CREATE USER snvn WITH PASSWORD 'yourpassword';
```

If you want `snvn` to have full admin access (like `postgres`):

```sql
ALTER USER snvn WITH SUPERUSER;
```

---

### 🔁 Then in pgAdmin:

1. Right-click your server → **Disconnect** (if connected)
2. Right-click → **Properties**
3. In the **Connection** tab:

   * Username: `snvn`
   * Password: use what you just set above
4. ✅ Click Save & Reconnect

---

Would you like to:

* Limit `snvn` to specific databases only?
* Let `snvn` create databases but not be full superuser?
* Create a user with read-only access?



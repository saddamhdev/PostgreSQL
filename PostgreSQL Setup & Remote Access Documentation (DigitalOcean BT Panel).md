Got it ✅
Here’s a clean documentation for your setup so you (or other devs) can repeat this without headache.

---

# 📘 PostgreSQL Setup & Remote Access Documentation (DigitalOcean BT Panel)

## 1. Verify PostgreSQL Installation

Check which PostgreSQL instance is running (since BT Panel installs its own):

```bash
sudo -u postgres psql -c "SHOW config_file;"
```

Output:

```
/www/server/pgsql/data/postgresql.conf
```

So your **active config** is in:

* `postgresql.conf` → `/www/server/pgsql/data/postgresql.conf`
* `pg_hba.conf` → `/www/server/pgsql/data/pg_hba.conf`

---

## 2. Allow Remote Connections

### Edit `postgresql.conf`

```bash
sudo nano /www/server/pgsql/data/postgresql.conf
```

Find:

```
#listen_addresses = 'localhost'
```

Change to:

```
listen_addresses = '*'
```

### Edit `pg_hba.conf`

```bash
sudo nano /www/server/pgsql/data/pg_hba.conf
```

At the bottom, add either:

* **Secure (only Jenkins server can connect):**

```
host    ssc_data    ssc_data    <JENKINS_IP>/32    md5
```

* **Testing (allow all, not recommended for production):**

```
host    ssc_data    ssc_data    0.0.0.0/0    md5
```

---

## 3. Restart PostgreSQL (Correct User)

PostgreSQL must run as `postgres`, not root.

```bash
sudo -u postgres /www/server/pgsql/bin/pg_ctl restart -D /www/server/pgsql/data
```

To check logs:

```bash
tail -f /www/server/pgsql/data/server.log
```

---

## 4. Firewall Rules

Ensure only Jenkins/your dev machine can reach port `5432`:

```bash
# Allow Jenkins server
sudo ufw allow from <JENKINS_IP> to any port 5432

# Remove wide-open rule if it exists
sudo ufw delete allow 5432/tcp
```

---

## 5. Verify Listening

```bash
sudo ss -ltnp | grep 5432
```

Expected:

```
0.0.0.0:5432
[::]:5432
```

---

## 6. Test Connection

From **local server**:

```bash
psql -U ssc_data -d ssc_data -h localhost -W
```

From **remote (Windows/Jenkins)**:

```powershell
psql -U ssc_data -d ssc_data -h 159.89.172.251 -W
```

---

## 7. Spring Boot Configuration

In `application-prod.properties`:

```properties
spring.datasource.url=jdbc:postgresql://159.89.172.251:5432/ssc_data
spring.datasource.username=ssc_data
spring.datasource.password=YOUR_STRONG_PASSWORD
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

---

## 8. Common Errors

* **`connection refused`** → PostgreSQL not listening externally. Fix `listen_addresses` and restart.
* **`password authentication failed`** → Wrong password or missing entry in `pg_hba.conf`.
* **`could not open shared memory segment`** → Started as root. Must always run as `postgres`.

---

## 9. Optional: Create Systemd Service

So you don’t have to use `pg_ctl` manually.

Create `/etc/systemd/system/postgresql-bt.service`:

```ini
[Unit]
Description=Custom PostgreSQL (BT Panel)
After=network.target

[Service]
Type=forking
User=postgres
ExecStart=/www/server/pgsql/bin/pg_ctl start -D /www/server/pgsql/data -l /www/server/pgsql/data/server.log
ExecStop=/www/server/pgsql/bin/pg_ctl stop -D /www/server/pgsql/data
ExecReload=/www/server/pgsql/bin/pg_ctl reload -D /www/server/pgsql/data
PIDFile=/www/server/pgsql/data/postmaster.pid

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable postgresql-bt
sudo systemctl start postgresql-bt
```

Now you can manage it like:

```bash
sudo systemctl restart postgresql-bt
```

---

✅ With this, Jenkins → Spring Boot → PostgreSQL pipeline will work properly and securely.

---

Do you want me to also **write a short Jenkins deployment note** (how to configure the pipeline to use the new PostgreSQL settings)?

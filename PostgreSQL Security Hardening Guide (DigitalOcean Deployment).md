Perfect 👍 — here’s a **PostgreSQL Security Documentation** tailored for your setup. You can keep this as a reference for your server or share with your team.

---

# 🔐 PostgreSQL Security Hardening Guide (DigitalOcean Deployment)

## 1. Secure PostgreSQL User Accounts

* Use **strong passwords** (≥16 characters, mix upper/lowercase, numbers, symbols).

* Example for setting a strong password:

  ```sql
  ALTER USER ssc_data WITH PASSWORD 'Very$trongP@ssw0rd123!';
  ```

* Avoid using the `postgres` superuser account for applications.
  Instead, create a **least-privilege user** (`ssc_data`) with access only to its own database.

---

## 2. Restrict Access via `pg_hba.conf`

Edit your **pg\_hba.conf** (you already found it at `/www/server/pgsql/data/pg_hba.conf`):

```conf
# Allow Windows PC (public IP)
host    ssc_data    ssc_data    59.153.103.109/32    md5

# Allow Jenkins server (replace with its real public IP)
host    ssc_data    ssc_data    <JENKINS_IP>/32      md5

# Local connections (safe, keep them)
local   all         all                             trust
host    all         all         127.0.0.1/32        trust
host    all         all         ::1/128             trust
```

➡️ This means:

* ✅ Only **your PC** and **your Jenkins server** can connect remotely.
* ❌ Everyone else on the internet is blocked.

Restart PostgreSQL to apply changes:

```bash
sudo -u postgres /www/server/pgsql/bin/pg_ctl restart -D /www/server/pgsql/data
```

---

## 3. Firewall Protection (`ufw`)

Always protect port **5432** with firewall rules.

```bash
# Allow only your Windows PC
sudo ufw allow from 59.153.103.109 to any port 5432

# Allow only your Jenkins server
sudo ufw allow from <JENKINS_IP> to any port 5432

# Deny everyone else
sudo ufw deny 5432
```

Check rules:

```bash
sudo ufw status numbered
```

➡️ This ensures PostgreSQL is **invisible** to the rest of the internet.

---

## 4. Run PostgreSQL on a Non-Default Port (Optional)

Changing from `5432` → another high port (e.g., `6543`) reduces automated scans.

Edit `/www/server/pgsql/data/postgresql.conf`:

```conf
port = 6543
```

Then update firewall + Spring Boot JDBC URL:

```properties
spring.datasource.url=jdbc:postgresql://159.89.172.251:6543/ssc_data
```

---

## 5. Use SSL/TLS (Optional, Recommended for Production)

Enable SSL in `postgresql.conf`:

```conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
```

➡️ This ensures all DB traffic between Jenkins/Spring Boot ↔ PostgreSQL is **encrypted**.

---

## 6. Monitoring & Maintenance

* Check login attempts:

  ```bash
  sudo tail -f /www/server/pgsql/data/log/postgresql.log
  ```
* Regularly apply security updates:

  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
* Rotate strong passwords periodically.

---

## ✅ Summary

* PostgreSQL is now **locked down** to specific IPs.
* Firewall denies brute-force attacks from random bots.
* Application users (`ssc_data`) have **strong passwords**.
* Optionally add **SSL** & **custom port** for extra protection.

---

👉 Would you like me to also draft a **step-by-step Jenkins + Spring Boot + PostgreSQL secure deployment checklist** (CI/CD + DB security together)? That way you’ll have one full document for your project.

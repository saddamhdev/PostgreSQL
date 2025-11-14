Perfect 👍 Let’s do a **secure PostgreSQL installation on your DigitalOcean Ubuntu 22.04 droplet**, step by step.

I’ll assume you want:

* PostgreSQL installed inside the droplet
* A dedicated DB + user for your Spring Boot app
* **Strong password**
* **Firewall restricted** (only allow your app to connect, not the whole internet)

---

# 🚀 PostgreSQL Installation Instructions (DigitalOcean Droplet)

### 1️⃣ Update server

```bash
sudo apt update && sudo apt upgrade -y
```

---

### 2️⃣ Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

Enable + start:

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
```

---

### 3️⃣ Create DB + user

Switch to postgres system user:

```bash
sudo -i -u postgres
```

Enter PostgreSQL shell:

```bash
psql
```

Inside `psql`, create DB and user with a strong password:

```sql
CREATE DATABASE trainingapp;
CREATE USER springuser WITH ENCRYPTED PASSWORD 's6T!v9dN$Qw7xLmP2eZ#';
GRANT ALL PRIVILEGES ON DATABASE trainingapp TO springuser;
\q
```

Exit postgres user:

```bash
exit
```

---

### 4️⃣ Configure PostgreSQL for connections

Edit config:

```bash
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Find:

```
#listen_addresses = 'localhost'
```

Change to:

```
listen_addresses = '*'
```

---

### 5️⃣ Restrict access in `pg_hba.conf`

```bash
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

Add line **for your app server IP only** (replace `X.X.X.X` with the server running Spring Boot or Jenkins):

```
host    trainingapp    springuser    X.X.X.X/32    md5
```

If your app runs **on the same droplet**, you don’t need this (local connection works).

---

### 6️⃣ Restart PostgreSQL

```bash
sudo service pgsql restart

```

---

### 7️⃣ Configure Firewall

Enable ufw if not active:

```bash
sudo ufw enable
```

Always allow SSH (so you don’t lock yourself out):

```bash
sudo ufw allow 22/tcp
```

Allow PostgreSQL only for your app server’s IP (replace `X.X.X.X`):

```bash
sudo ufw allow from X.X.X.X to any port 5432 proto tcp
```

Check:

```bash
sudo ufw status
```

If you also use **DigitalOcean Cloud Firewall**, go to dashboard → Networking → Firewalls → add rule:

* TCP
* Port 5432
* Source = your app server IP

---

### 8️⃣ Test connection

From your app server or PC (if allowed):

```bash
psql -h 159.89.172.251 -U springuser -d trainingapp -p 5432
```

It should prompt for password (`s6T!v9dN$Qw7xLmP2eZ#`).

---

### 9️⃣ Spring Boot configuration

In `application.properties` (if app runs on same droplet):

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/trainingapp
spring.datasource.username=springuser
spring.datasource.password=s6T!v9dN$Qw7xLmP2eZ#
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

If app runs remotely:

```properties
spring.datasource.url=jdbc:postgresql://159.89.172.251:5432/trainingapp
spring.datasource.username=springuser
spring.datasource.password=s6T!v9dN$Qw7xLmP2eZ#
```

---

✅ Done! PostgreSQL is installed, secure, and ready.

---

👉 Do you want me to also show you how to **make Jenkins inject DB credentials (username/password) securely** instead of writing them in `application.properties`?

# Frappe v16 + PostgreSQL 15 Setup on Ubuntu 24.04 WSL2

This guide sets up:

- Ubuntu 24.04 LTS on WSL2
- Python 3.14
- PostgreSQL 15.17
- NodeJS 24
- Yarn 1.x
- Redis
- Bench
- Frappe Framework v16
- PostgreSQL instead of MariaDB

This setup follows the latest Frappe v16 recommendations while fixing the real-world issues that happen during installation.

---

# IMPORTANT NOTES

## 1. Use Linux Filesystem ONLY

DO NOT create the project inside:

```bash
/mnt/c/
```

Use:

```bash
/home/<your-user>/
```

Example:

```bash
/home/abhi/frappe-dev
```

Otherwise:
- file watching becomes slow
- yarn install becomes painful
- node_modules performance tanks
- bench hot reload becomes unstable

---

## 2. Enable Enough RAM for WSL

Create this file in Windows:

```text
C:\Users\<YOUR_USERNAME>\.wslconfig
```

Add:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Then restart WSL:

```powershell
wsl --shutdown
```

---

# STEP 1 — Enable systemd in WSL

Open Ubuntu terminal.

Edit:

```bash
sudo nano /etc/wsl.conf
```

Add:

```ini
[boot]
systemd=true
```

Save:
- CTRL + O
- Enter
- CTRL + X

Shutdown WSL from PowerShell:

```powershell
wsl --shutdown
```

Reopen Ubuntu.

Verify:

```bash
systemctl status
```

If systemd works, continue.

---

# STEP 2 — Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

---

# STEP 3 — Install Core Dependencies

```bash
sudo apt install -y \
git curl build-essential gcc g++ make \
pkg-config software-properties-common \
redis-server xvfb libfontconfig wkhtmltopdf \
libssl-dev libffi-dev \
python3-dev python3-pip python3-venv \
pipx
```

---

# STEP 4 — Install Python 3.14

Add deadsnakes PPA:

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
```

Install Python 3.14:

```bash
sudo apt install -y \
python3.14 \
python3.14-dev \
python3.14-venv
```

Verify:

```bash
python3.14 --version
```

Expected:

```bash
Python 3.14.x
```

---

# STEP 5 — Upgrade pip

```bash
python3.14 -m pip install --upgrade pip
```

Verify:

```bash
python3.14 -m pip --version
```

---

# STEP 6 — Install PostgreSQL 15.17

## Add PostgreSQL Repository

```bash
sudo apt install -y gnupg2
```

```bash
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
gpg --dearmor | sudo tee /usr/share/keyrings/postgresql.gpg > /dev/null
```

```bash
echo "deb [signed-by=/usr/share/keyrings/postgresql.gpg] \
http://apt.postgresql.org/pub/repos/apt noble-pgdg main" | \
sudo tee /etc/apt/sources.list.d/pgdg.list
```

Update:

```bash
sudo apt update
```

Install PostgreSQL:

```bash
sudo apt install -y \
postgresql-15 \
postgresql-client-15 \
postgresql-contrib-15 \
libpq-dev
```

Verify:

```bash
psql --version
```

Expected:

```bash
psql (PostgreSQL) 15.17
```

---

# STEP 7 — Install MariaDB Development Libraries

IMPORTANT:

Even when using PostgreSQL, Frappe still installs `mysqlclient`.

So you MUST install MariaDB development headers.

Install:

```bash
sudo apt install -y \
libmariadb-dev \
libmariadb-dev-compat \
mariadb-client
```

NOTE:
You do NOT need:
- mariadb-server
- mysql-server

Only development libraries are needed.

---

# STEP 8 — Start Services

Enable and start PostgreSQL:

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Enable and start Redis:

```bash
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

Verify:

```bash
sudo systemctl status postgresql
sudo systemctl status redis-server
```

---

# STEP 9 — Create PostgreSQL User

Open PostgreSQL shell:

```bash
sudo -u postgres psql
```

Run:

```sql
CREATE ROLE frappe WITH LOGIN PASSWORD 'frappe';
ALTER ROLE frappe CREATEDB;
ALTER ROLE frappe SUPERUSER;
\q
```

---

# STEP 10 — Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Reload shell:

```bash
source ~/.bashrc
```

---

# STEP 11 — Install NodeJS 24

Install Node 24:

```bash
nvm install 24
nvm use 24
nvm alias default 24
```

Verify:

```bash
node -v
npm -v
```

Expected:
- Node v24.x

---

# STEP 12 — Install Yarn

Frappe requires Yarn Classic (1.x).

Install:

```bash
npm install -g yarn
```

Verify:

```bash
yarn -v
```

Expected:
- 1.22+

---

# STEP 13 — Install uv

Modern Bench internally uses `uv`.

Install:

```bash
pipx install uv
```

Reload shell:

```bash
source ~/.bashrc
```

Verify:

```bash
uv --version
```

---

# STEP 14 — Install Bench

Install Bench:

```bash
pipx install frappe-bench
```

Enable pipx paths:

```bash
pipx ensurepath
```

Reload shell:

```bash
source ~/.bashrc
```

Verify:

```bash
bench --version
```

---

# STEP 15 — Create Workspace

```bash
mkdir ~/frappe-dev
cd ~/frappe-dev
```

---

# STEP 16 — Initialize Bench

IMPORTANT:
This step may take a long time.

Run:

```bash
bench init frappe-bench \
--frappe-branch version-16 \
--python python3.14
```

This installs:
- Python environment
- Frappe framework
- Node packages
- Redis configs
- Bench configs

---

# STEP 17 — Open Bench Directory

```bash
cd frappe-bench
```

---

# STEP 18 — Create Site Using PostgreSQL

IMPORTANT:
Always specify PostgreSQL explicitly.

Run:

```bash
bench new-site dev.local \
--db-type postgres
```

Use:
- DB Host → localhost
- DB Port → 5432
- DB User → frappe
- DB Password → frappe

Then set:
- Administrator password

---

# STEP 19 — Start Frappe

Run:

```bash
bench start
```

Open browser:

```text
http://localhost:8000
```

Login:
- User → Administrator
- Password → the password you created

---

# OPTIONAL — Install ERPNext

Inside bench directory:

```bash
bench get-app erpnext --branch version-16
```

Install app:

```bash
bench --site dev.local install-app erpnext
```

---

# USEFUL COMMANDS

## Start Bench

```bash
bench start
```

---

## Stop Bench

CTRL + C

---

## Open PostgreSQL

```bash
sudo -u postgres psql
```

---

## Restart Redis

```bash
sudo systemctl restart redis-server
```

---

## Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

---

# COMMON ERRORS

---

## ERROR: uv not found

Fix:

```bash
pipx install uv
```

---

## ERROR: mysqlclient build failed

Fix:

```bash
sudo apt install -y \
libmariadb-dev \
libmariadb-dev-compat \
mariadb-client
```

---

## ERROR: Port 8000 already in use

Check:

```bash
lsof -i :8000
```

Kill conflicting process.

---

## ERROR: Redis connection refused

Restart Redis:

```bash
sudo systemctl restart redis-server
```

---

## ERROR: PostgreSQL authentication failed

Verify user exists:

```bash
sudo -u postgres psql
```

Then:

```sql
\du
```

---

# FINAL RECOMMENDED STACK

| Component | Version |
|---|---|
| Ubuntu | 24.04 |
| Python | 3.14 |
| PostgreSQL | 15.17 |
| NodeJS | 24 |
| Yarn | 1.22+ |
| Redis | 6+ |
| Bench | latest |
| Frappe | version-16 |

---

# EXTRA NOTES / COMMON ISSUES

---

## Site Opens but Shows "127.0.0.1 does not exist"

Cause:
- Bench does not know which site should respond to localhost requests.

Fix:

```bash
bench use <site-name>
```

Example:

```bash
bench use abhi.com
```

Then restart:

```bash
bench start
```

---

## Site Loads but CSS/JS Assets Are Missing

Symptoms:
- Plain HTML page
- No styling
- Broken UI
- Console shows missing CSS/JS files

Common Causes:
- Assets were not built properly
- Wrong hostname mapping
- Site name mismatch
- Browser cached broken assets

Fix:

Stop bench:

```bash
CTRL + C
```

Rebuild assets:

```bash
bench build
```

Clear cache:

```bash
bench clear-cache
bench clear-website-cache
```

Ensure correct site is active:

```bash
bench use <site-name>
```

Start again:

```bash
bench start
```

Open:

```text
http://localhost:8000/app
```

NOT just `/`.

---



## PostgreSQL Authentication Failure

Error:

```text
password authentication failed for user "postgres"
```

Cause:
- Ubuntu uses peer auth by default
- Bench connects using TCP/password auth

Fix:

```bash
sudo -u postgres psql
```

Then:

```sql
ALTER USER postgres PASSWORD 'postgres';
\q
```

---

## Bench Init Fails with "uv not found"

Cause:
- New Bench versions require `uv`

Fix:

```bash
pipx install uv
```

---

## mysqlclient Build Failure During Bench Init

Error:

```text
Failed to build mysqlclient
```

Cause:
- Frappe still installs mysqlclient internally
- MariaDB development headers are missing

Fix:

```bash
sudo apt install -y \
libmariadb-dev \
libmariadb-dev-compat \
mariadb-client
```

NOTE:
Do NOT install:
- mariadb-server
- mysql-server

Only dev libraries are required.

---

## PostgreSQL Support Is Experimental

Current Frappe v16 PostgreSQL support is still experimental.

Possible issues:
- migrations may fail occasionally
- some third-party apps assume MariaDB
- certain SQL queries may break
- ERPNext modules may have edge-case issues

For development:
- PostgreSQL is fine

For production ERP workloads:
- MariaDB is still safer currently

---

## WSL Performance Problems

Symptoms:
- slow asset builds
- slow hot reload
- high CPU usage
- node_modules lag

Most Common Cause:
Project stored inside:

```text
/mnt/c/
```

Correct Location:

```text
/home/<user>/
```

Example:

```text
/home/abhi/frappe-dev
```

---

## Browser Cache Issues

Sometimes old assets remain cached.

Fix:
- hard refresh browser
- open in incognito
- clear browser cache

Hard refresh shortcut:

```text
CTRL + SHIFT + R
```

---

## Scheduler Disabled Warning

Message:

```text
Scheduler is disabled
```

This is normal after site creation.

Enable scheduler later with:

```bash
bench enable-scheduler
```

Or for specific site:

```bash
bench --site <site-name> enable-scheduler
```

---

## Default Frappe Desk URL

Frappe Desk UI is located at:

```text
http://localhost:8000/app
```

Opening just `/` may show:
- website route
- 404 page
- empty site

depending on configuration.

---

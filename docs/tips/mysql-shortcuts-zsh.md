# 🚀 MySQL CLI Shortcuts & Aliases for macOS (Zsh)

A practical guide for SQL course students in the [RedRover School](https://redrover.school/?lang=en).

If you installed MySQL on macOS following the [MySQL on Mac installation guide](https://docs.google.com/document/d/1J3Jl6Y6-7pf40rZXycrU1N85WjXpFx3dbAs0G_x_weA/edit?tab=t.0), managing the server via terminal typically requires long paths like `/usr/local/mysql/support-files/mysql.server ...`.

This guide walks you through setting up convenient terminal aliases to start, stop, check server status, and safely wipe/recreate databases in one short command.

---

## 🛠 1. Add MySQL to PATH

By default, MySQL binaries (`mysql`, `mysqldump`, etc.) are located in `/usr/local/mysql/bin`. To execute them from any directory without getting `command not found: mysql`, add this directory to your `PATH` environment variable.

```bash
export PATH="/usr/local/mysql/bin:$PATH"
```

---

## ⚡️ 2. Available Shortcuts & Functions

Once configured, you will have access to the following commands:

1. **Server Management:**
   - `mysql-start` — Start the local MySQL server.
   - `mysql-stop` — Stop the MySQL server.
   - `mysql-status` — Check whether the MySQL server is running.
   - `mysql-restart` — Restart the MySQL server.

2. **Database Wipe / Reset:**
   - `mysql-wipe <database_name>` — Drops and recreates a clean database with an interactive `(y/N)` confirmation safeguard.

---

## 📝 3. Step-by-Step Installation

### Step 1: Open `.zshrc` in an Editor
Open your shell configuration file in terminal using `nano`:

```bash
nano ~/.zshrc
```

*(Or open it in VS Code: `code ~/.zshrc`)*

---

### Step 2: Paste the Configuration Block
Scroll to the end of the file and paste the following snippet:

```bash
# ==========================================
# MySQL Configuration & Aliases
# ==========================================

# 1. Add MySQL binaries to PATH
export PATH="/usr/local/mysql/bin:$PATH"

# 2. Server management aliases
alias mysql-status="sudo /usr/local/mysql/support-files/mysql.server status"
alias mysql-start="sudo /usr/local/mysql/support-files/mysql.server start"
alias mysql-stop="sudo /usr/local/mysql/support-files/mysql.server stop"
alias mysql-restart="sudo /usr/local/mysql/support-files/mysql.server restart"

# 3. Safe database wipe function (drops & recreates empty DB)
mysql-wipe() {
  if [ -z "$1" ]; then
    echo "⚠️  Error: database name required. Example: mysql-wipe my_database"
    return 1
  fi

  # Interactive confirmation
  echo -n "Are you sure you want to completely wipe the database '$1'? (y/N): "
  read -r confirmation

  if [[ "$confirmation" =~ ^[Yy]$ ]]; then
    mysql -u root -p -e "DROP DATABASE IF EXISTS \`$1\`; CREATE DATABASE \`$1\`;"
    if [ $? -eq 0 ]; then
      echo "✅ Database '$1' successfully wiped and recreated."
    fi
  else
    echo "❌ Operation cancelled."
  fi
}
```

> **Note on root password:** If your local `root` user does not have a password configured, remove the `-p` flag from the `mysql` invocation in the `mysql-wipe` function.

---

### Step 3: Save and Exit
If using `nano`:
1. Press `Ctrl + O`, then hit `Enter` (to write changes).
2. Press `Ctrl + X` (to exit `nano`).

---

### Step 4: Reload Shell Configuration
Apply the changes immediately to your current terminal session:

```bash
source ~/.zshrc
```

---

## 🧪 4. Usage Examples

### Check Server Status
```bash
mysql-status
# Output: SUCCESS! MySQL running (PID: 12345)
```

### Start Server
```bash
mysql-start
# Prompts for your macOS administrator password (sudo) and starts MySQL
```

### Stop Server
```bash
mysql-stop
```

### Wipe & Reset a Database (e.g. `classicmodels`)
```bash
mysql-wipe classicmodels
```

**Example interactive session:**
```text
Are you sure you want to completely wipe the database 'classicmodels'? (y/N): y
Enter password: <enter your MySQL root password>
✅ Database 'classicmodels' successfully wiped and recreated.
```

If you press `n`, `Enter`, or any other key:
```text
Are you sure you want to completely wipe the database 'classicmodels'? (y/N): n
❌ Operation cancelled.
```

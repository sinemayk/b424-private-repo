# Hands-on Linux-04: Linux File Permissions & Package Management

The goal of this hands-on training is to help students practice **Linux file permissions** and **package management** in a real AWS/DevOps-style environment.

The labs use **Amazon Linux 2023** and **Ubuntu** side by side so students can see how the same administration task is performed on different Linux distributions.

---

## Learning Outcomes

By the end of this hands-on training, students will be able to:

* Read Linux file permissions.
* Change permissions with `chmod`.
* Make Python and shell scripts executable.
* Check detailed file information with `stat`.
* Understand how `umask` affects default permissions.
* Explain how package managers and repositories work.
* Use `dnf` on Amazon Linux 2023.
* Use `apt` on Ubuntu.
* Search, install, inspect, update, and remove packages.
* Compare equivalent package-management tasks across different Linux distributions.

---

# Section 1 - Prepare the Lab Environment

Create a workspace:

```bash
mkdir -p ~/linux-day4/{permissions,scripts,packages}
cd ~/linux-day4
```

Check the structure:

```bash
ls -R
```

Expected structure:

```text
linux-day4/
├── packages
├── permissions
└── scripts
```

---

# Section 2 - Read Linux File Permissions

Move into the permissions directory:

```bash
cd ~/linux-day4/permissions
```

Create some files:

```bash
touch app.conf app.log notes.txt
```

Check permissions:

```bash
ls -l
```

Example output:

```text
-rw-r--r-- 1 ec2-user ec2-user 0 Sep 24 10:00 app.conf
```

Break it down:

```text
-  rw-  r--  r--
|   |    |    |
|   |    |    └── Others
|   |    └─────── Group
|   └──────────── Owner
└──────────────── File type
```

Permission values:

| Permission | Meaning | Numeric Value |
|---|---|---:|
| `r` | Read | 4 |
| `w` | Write | 2 |
| `x` | Execute | 1 |

---

# Section 3 - Practice `chmod`

## Example 1 - Configuration File

Set `app.conf` to:

* Owner: read/write
* Group: read
* Others: read

```bash
chmod 644 app.conf
ls -l app.conf
```

Expected permission:

```text
-rw-r--r--
```

---

## Example 2 - Private File

Create:

```bash
touch secrets.txt
```

Allow only the owner to read and write:

```bash
chmod 600 secrets.txt
ls -l secrets.txt
```

Expected:

```text
-rw-------
```

This type of permission is useful for files containing sensitive configuration values.

---

## Example 3 - Shared Read-Only File

Create:

```bash
touch release-notes.txt
```

Set:

```bash
chmod 444 release-notes.txt
ls -l release-notes.txt
```

Expected:

```text
-r--r--r--
```

No user has write permission.

---

## Example 4 - Executable Script

Create:

```bash
touch deploy.sh
```

Set:

```bash
chmod 755 deploy.sh
ls -l deploy.sh
```

Expected:

```text
-rwxr-xr-x
```

Meaning:

```text
Owner  -> read + write + execute
Group  -> read + execute
Others -> read + execute
```

---

## Example 5 - Owner and Group Only

Set:

```bash
chmod 750 deploy.sh
ls -l deploy.sh
```

Expected:

```text
-rwxr-x---
```

Now users outside the owner and group have no access.

---

# Section 4 - Symbolic `chmod` Examples

Add execute permission for the owner:

```bash
chmod u+x deploy.sh
```

Remove write permission from the group:

```bash
chmod g-w app.conf
```

Remove all permissions from others:

```bash
chmod o-rwx secrets.txt
```

Add read permission for everyone:

```bash
chmod a+r release-notes.txt
```

Check after each command:

```bash
ls -l
```

---

# Section 5 - Make Python Scripts Executable

This is an important DevOps use of Linux permissions.

Move into the scripts directory:

```bash
cd ~/linux-day4/scripts
```

---

## Python Example 1 - Hello DevOps

Create the script:

```bash
cat > hello.py <<'EOF'
#!/usr/bin/env python3

print("Hello from Linux!")
print("Python script executed successfully.")
EOF
```

Check the file:

```bash
cat hello.py
```

Try to run it directly:

```bash
./hello.py
```

You may get:

```text
Permission denied
```

Check permissions:

```bash
ls -l hello.py
```

Make it executable:

```bash
chmod +x hello.py
```

Check again:

```bash
ls -l hello.py
```

Run it:

```bash
./hello.py
```

Expected output:

```text
Hello from Linux!
Python script executed successfully.
```

---

## Python Example 2 - System Information Script

Create:

```bash
cat > system_info.py <<'EOF'
#!/usr/bin/env python3

import os
import platform

print("Current user:", os.getenv("USER"))
print("Operating system:", platform.system())
print("Kernel:", platform.release())
print("Current directory:", os.getcwd())
EOF
```

Make it executable:

```bash
chmod 755 system_info.py
```

Run:

```bash
./system_info.py
```

Also compare with:

```bash
python3 system_info.py
```

### Important Difference

This works even without execute permission:

```bash
python3 system_info.py
```

because `python3` is executing the file.

This requires execute permission:

```bash
./system_info.py
```

because Linux is executing the file directly.

---
## Python Example 3 - Write to a Log File

Create:

```bash
cat > app_logger.py <<'EOF'
#!/usr/bin/env python3

from datetime import datetime

with open("application.log", "a") as log:
    log.write(f"{datetime.now()} - Application check completed\n")

print("Log entry added.")
EOF
```

Make it executable:

```bash
chmod 700 app_logger.py
```

Run it several times:

```bash
./app_logger.py
./app_logger.py
./app_logger.py
```

Check the log:

```bash
cat application.log
```

Check permissions:

```bash
ls -l app_logger.py application.log
```

---

# Section 6 - Why the Shebang Matters

Look at the first line of the Python scripts:

```python
#!/usr/bin/env python3
```

This tells Linux which interpreter should execute the script.

Check where Python is located:

```bash
command -v python3
```

Example:

```text
/usr/bin/python3
```

Without a valid shebang, directly executing the script may fail even when execute permission exists.

---

# Section 7 - `stat` Command

Check detailed information:

```bash
stat ~/linux-day4/scripts/hello.py
```

Look for:

* File size
* Permissions
* Owner
* Group
* Access time
* Modify time
* Change time

Compare:

```bash
ls -l ~/linux-day4/scripts/hello.py
```

`ls -l` gives a quick summary.

`stat` gives more detailed metadata.

---

# Section 8 - Permission Troubleshooting Example

Create:

```bash
cd ~/linux-day4/scripts

cat > broken.py <<'EOF'
#!/usr/bin/env python3
print("The script works!")
EOF
```

Remove execute permission:

```bash
chmod 644 broken.py
```

Try:

```bash
./broken.py
```

Check:

```bash
ls -l broken.py
```

Fix:

```bash
chmod +x broken.py
```

Run again:

```bash
./broken.py
```

This follows a common DevOps troubleshooting process:

```text
Run
 ↓
Error
 ↓
Check permissions
 ↓
Fix permissions
 ↓
Run again
```

---

# Section 9 - AWS SSH Key Permission Example

A private EC2 key should not be accessible to other users.

Example:

```bash
chmod 400 mykey.pem
```

Check:

```bash
ls -l mykey.pem
```

Expected pattern:

```text
-r--------
```

Example SSH command:

```bash
ssh -i mykey.pem ec2-user@<public-ip>
```

---

# Section 10 - Default Permissions with `umask`

Check the current value:

```bash
umask
```

A common value is:

```text
0022
```

Create:

```bash
cd ~/linux-day4/permissions

touch default-file.txt
mkdir default-dir
```

Check:

```bash
ls -ld default-file.txt default-dir
```

A common result with `umask 022` is:

```text
File      -> 644
Directory -> 755
```

Conceptually:

```text
Files:       666 - 022 -> 644
Directories: 777 - 022 -> 755
```

> `umask` removes permission bits from the default permission set.

---

# Section 11 - Try a Different `umask`

Temporarily change it:

```bash
umask 027
```

Create:

```bash
touch secure-file.txt
mkdir secure-dir
```

Check:

```bash
ls -ld secure-file.txt secure-dir
```

A common result:

```text
secure-file.txt -> 640
secure-dir      -> 750
```

Restore a common default:

```bash
umask 022
```

---

# Section 12 - Package Managers and Repositories

A package manager downloads and manages software from repositories.

Basic workflow:

```text
User
  ↓
Package Manager
  ↓
Repository
  ↓
Package + Dependencies
```

For this course:

| Distribution | Main Package Manager |
|---|---|
| Amazon Linux 2023 | `dnf` |
| Ubuntu | `apt` |

Repositories provide:

* Software packages
* Package metadata
* Dependencies
* Updates
* Security fixes

---

# Section 13 - Amazon Linux 2023 vs Ubuntu Side by Side

## Identify the System

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `cat /etc/os-release` | `cat /etc/os-release` |

Use:

```bash
cat /etc/os-release
```

before running distribution-specific package commands.

---

## Check Package Manager

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `dnf --version` | `apt --version` |

---

## Refresh Package Metadata

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf makecache` | `sudo apt update` |

---

## Check Available Updates

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf check-update` | `apt list --upgradable` |

---

## Upgrade Installed Packages

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf upgrade -y` | `sudo apt upgrade -y` |

---

# Section 14 - Example 1: Install `tree`

`tree` displays directories in a tree-style structure.

## Amazon Linux 2023

```bash
sudo dnf install tree -y
```

## Ubuntu

```bash
sudo apt install tree -y
```

Verify on both:

```bash
tree --version
```

Try:

```bash
tree ~/linux-day4
```

Remove it:

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf remove tree -y` | `sudo apt remove tree -y` |

---

# Section 15 - Example 2: Install a Web Server

The package names are different.

| Task | Amazon Linux 2023 | Ubuntu |
|---|---|---|
| Package | `httpd` | `apache2` |
| Install | `sudo dnf install httpd -y` | `sudo apt install apache2 -y` |
| Package info | `dnf info httpd` | `apt show apache2` |
| Remove | `sudo dnf remove httpd -y` | `sudo apt remove apache2 -y` |

### Amazon Linux 2023

```bash
sudo dnf search httpd
sudo dnf info httpd
sudo dnf install httpd -y
sudo dnf list installed httpd
```

### Ubuntu

```bash
apt search apache2
apt show apache2
sudo apt install apache2 -y
apt list --installed apache2
```

---

# Section 16 - Example 3: Git Package

## Amazon Linux 2023

```bash
dnf info git
sudo dnf install git -y
git --version
sudo dnf list installed git
```

## Ubuntu

```bash
apt show git
sudo apt install git -y
git --version
apt list --installed git
```

Remove:

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf remove git -y` | `sudo apt remove git -y` |

---

# Section 17 - Example 4: Python Package Tools

Check Python:

```bash
python3 --version
```

Check pip:

```bash
pip3 --version
```

If `pip3` is not installed:

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `sudo dnf install python3-pip -y` | `sudo apt install python3-pip -y` |

Verify:

```bash
pip3 --version
```

This connects package management with the executable Python scripts from the permission lab.

---

# Section 18 - Search for Packages

## Search for Git

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `dnf search git` | `apt search git` |

## Search for Python Packages

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `dnf search python3` | `apt search python3` |

## Search for Web Server Packages

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `dnf search httpd` | `apt search apache2` |

---

# Section 19 - Show Package Information

| Amazon Linux 2023 | Ubuntu |
|---|---|
| `dnf info git` | `apt show git` |
| `dnf info python3` | `apt show python3` |
| `dnf info httpd` | `apt show apache2` |

Package information commonly includes:

* Package name
* Version
* Architecture
* Repository
* Description

---

# Section 20 - List Installed Packages

## Amazon Linux 2023

```bash
dnf list installed
```

Use `less`:

```bash
dnf list installed | less
```

Search for Python:

```bash
dnf list installed | grep python
```

## Ubuntu

```bash
apt list --installed
```

Use `less`:

```bash
apt list --installed | less
```

Search for Python:

```bash
apt list --installed | grep python
```

---

# Section 21 - Remove Unused Packages

## Amazon Linux 2023

```bash
sudo dnf autoremove -y
```

## Ubuntu

```bash
sudo apt autoremove -y
```

Use this carefully on important systems.

Review what the package manager plans to remove before confirming changes in production environments.

---

# Section 22 - Package Management Quick Comparison

| Task | Amazon Linux 2023 | Ubuntu |
|---|---|---|
| Package manager | `dnf` | `apt` |
| Refresh metadata | `dnf makecache` | `apt update` |
| Search | `dnf search PACKAGE` | `apt search PACKAGE` |
| Package info | `dnf info PACKAGE` | `apt show PACKAGE` |
| Install | `dnf install PACKAGE` | `apt install PACKAGE` |
| Remove | `dnf remove PACKAGE` | `apt remove PACKAGE` |
| Upgrade packages | `dnf upgrade` | `apt upgrade` |
| Installed packages | `dnf list installed` | `apt list --installed` |
| Remove unused packages | `dnf autoremove` | `apt autoremove` |

---

# Section 23 - Mini-Lab: Build and Run a Python Health Check

Create:

```bash
cd ~/linux-day4/scripts

cat > health_check.py <<'EOF'
#!/usr/bin/env python3

import shutil
import platform

print("=== Server Health Check ===")
print("OS:", platform.system())
print("Kernel:", platform.release())

tools = ["python3", "git", "curl"]

for tool in tools:
    location = shutil.which(tool)

    if location:
        print(f"[OK] {tool}: {location}")
    else:
        print(f"[MISSING] {tool}")
EOF
```

Check permissions:

```bash
ls -l health_check.py
```

Make executable:

```bash
chmod 750 health_check.py
```

Run:

```bash
./health_check.py
```

Inspect with:

```bash
stat health_check.py
```

---

# Section 24 - Mini-Lab: Different Permission Levels

Create three scripts:

```bash
touch public.py team.py private.py
```

Apply:

```bash
chmod 755 public.py
chmod 750 team.py
chmod 700 private.py
```

Check:

```bash
ls -l *.py
```

Compare:

```text
755 -> owner rwx, group r-x, others r-x
750 -> owner rwx, group r-x, others ---
700 -> owner rwx, group ---, others ---
```

Discuss:

* Which is suitable for a public utility script?
* Which is suitable for a team deployment script?
* Which is suitable for an owner-only administration script?

---

# Section 25 - Practice Tasks

## Task 1 - Permissions

1. Create `~/linux-day4/practice`.
2. Create `config.txt`.
3. Set its permissions to `640`.
4. Create `deploy.py`.
5. Add a Python shebang.
6. Make it executable with `750`.
7. Run it with `./deploy.py`.
8. Check it with `stat`.

---

## Task 2 - Permission Troubleshooting

1. Create `test.py`.
2. Add a simple `print()` statement.
3. Set permission to `644`.
4. Try `./test.py`.
5. Identify the problem.
6. Fix the permission.
7. Run the script again.

---

## Task 3 - `umask`

1. Check the current `umask`.
2. Set `umask 027`.
3. Create one file.
4. Create one directory.
5. Check their permissions.
6. Explain the results.
7. Restore `umask 022`.

---

## Task 4 - Amazon Linux 2023

1. Confirm the OS.
2. Check the `dnf` version.
3. Search for `tree`.
4. Display package information.
5. Install `tree`.
6. Verify the installation.
7. Use `tree` on `~/linux-day4`.
8. Remove `tree`.

---

## Task 5 - Ubuntu

1. Confirm the OS.
2. Check the `apt` version.
3. Search for `tree`.
4. Display package information.
5. Install `tree`.
6. Verify the installation.
7. Use `tree` on `~/linux-day4`.
8. Remove `tree`.

---

## Task 6 - Compare Package Names

Complete this table:

| Purpose | Amazon Linux 2023 | Ubuntu |
|---|---|---|
| Apache web server | ? | ? |
| Python package installer | ? | ? |
| Git | ? | ? |

Then install one package on each operating system.

---

# Final Challenge

Create an executable Python script named:

```text
environment_check.py
```

The script should print:

* Current user
* Current directory
* Operating system
* Kernel version
* Whether `git` exists
* Whether `python3` exists
* Whether `curl` exists

Requirements:

```text
1. Use #!/usr/bin/env python3
2. Set permission to 750
3. Run with ./environment_check.py
4. Check permissions with ls -l
5. Inspect metadata with stat
```

---

# Quick Review

## Permissions

```bash
ls -l
chmod 755 file
chmod 750 file
chmod 700 file
chmod 644 file
chmod 600 file
chmod u+x file
stat file
umask
```

## Python Script

```bash
chmod +x script.py
./script.py
python3 script.py
```

## Amazon Linux 2023

```bash
dnf --version
sudo dnf makecache
dnf search package-name
dnf info package-name
sudo dnf install package-name -y
sudo dnf remove package-name -y
dnf list installed
sudo dnf check-update
sudo dnf upgrade -y
sudo dnf autoremove -y
```

## Ubuntu

```bash
apt --version
sudo apt update
apt search package-name
apt show package-name
sudo apt install package-name -y
sudo apt remove package-name -y
apt list --installed
apt list --upgradable
sudo apt upgrade -y
sudo apt autoremove -y
```

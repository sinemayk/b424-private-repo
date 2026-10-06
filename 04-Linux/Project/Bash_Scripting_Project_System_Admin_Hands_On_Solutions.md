# Bash Scripting Project — System Administration
## Hands-On Solution

This solution follows the **System Administration  project**, from Part 1 through the final `linux-admin.sh` toolkit.

For every script, you can create it with:

```bash
nano script-name.sh
```

Then make it executable:

```bash
chmod +x script-name.sh
```

---

# Part 1 — System Information Script

Create:

```bash
nano system-info.sh
```

Add:

```bash
#!/bin/bash

echo "===== SYSTEM INFORMATION ====="
echo

echo "Current User:"
whoami
echo

echo "Hostname:"
hostname
echo

echo "Date:"
date
echo

echo "Kernel:"
uname -r
echo

echo "User ID and Groups:"
id
echo

echo "Uptime:"
uptime
```

Make executable:

```bash
chmod +x system-info.sh
```

Run:

```bash
./system-info.sh
```

Example:

```text
===== SYSTEM INFORMATION =====

Current User:
ec2-user

Hostname:
ip-172-31-20-10

Date:
Mon Sep 28 18:30:21 UTC 2026

Kernel:
6.1.0-101-amazon

User ID and Groups:
uid=1000(ec2-user) gid=1000(ec2-user) groups=1000(ec2-user),10(wheel)

Uptime:
18:30:21 up 2 hours, 1 user, load average: 0.00, 0.01, 0.00
```

---

# Part 2 — Store Information in Variables

Modify:

```bash
nano system-info.sh
```

Use:

```bash
#!/bin/bash

current_user=$(whoami)
computer_name=$(hostname)
current_date=$(date)
kernel_version=$(uname -r)
user_info=$(id)
system_uptime=$(uptime)

echo "===== SYSTEM INFORMATION ====="
echo

echo "Current User: $current_user"
echo "Hostname: $computer_name"
echo "Date: $current_date"
echo "Kernel: $kernel_version"
echo "User ID and Groups: $user_info"
echo "Uptime: $system_uptime"
```

Notice:

```bash
current_user=$(whoami)
```

means:

```text
run whoami
↓
take its output
↓
save that output in current_user
```

Then:

```bash
echo "$current_user"
```

displays it.

### Question

**What is the advantage of storing command output inside a variable?**

It lets us reuse the command result later without repeatedly running the command.

For example:

```bash
hostname=$(hostname)
```

Now `$hostname` can be used many times.

---

# Part 3 — File and Directory Inspection Script

Create:

```bash
nano file-audit.sh
```

Add:

```bash
#!/bin/bash

echo "===== FILE SYSTEM AUDIT ====="
echo

echo "--- PUBLIC ---"
ls -ld public
ls -l public/announcement.txt
echo

echo "--- PRIVATE ---"
ls -ld private
ls -l private/secrets.txt
echo

echo "--- DEVELOPERS ---"
ls -ld developers
ls -l developers/project.txt
ls -l developers/deploy.sh
echo

echo "--- SHARED ---"
ls -ld shared
echo
```

Run:

```bash
chmod +x file-audit.sh
./file-audit.sh
```

Example:

```text
--- DEVELOPERS ---
drwxrws--- 2 ec2-user developers 45 Sep 28 12:00 developers
-rw-rw---- 1 ec2-user developers 30 Sep 28 12:00 developers/project.txt
-rwxr-x--- 1 ec2-user developers 50 Sep 28 12:00 developers/deploy.sh
```

### Why `ls -ld`?

For a directory:

```bash
ls -ld developers
```

shows information about the **directory itself**.

Without `-d`:

```bash
ls -l developers
```

shows what is **inside** the directory.

### Question

Why is one script more useful than entering every command manually?

Because the administrator can repeat the same audit consistently with:

```bash
./file-audit.sh
```

instead of typing many commands every time.

---

# Part 4 — Use a `for` Loop

Create:

```bash
nano directory-check.sh
```

Add:

```bash
#!/bin/bash

echo "===== DIRECTORY CHECK ====="
echo

for dir in public private developers shared
do
    echo "Checking: $dir"
    ls -ld "$dir"
    echo
done
```

Run:

```bash
chmod +x directory-check.sh
./directory-check.sh
```

Output:

```text
===== DIRECTORY CHECK =====

Checking: public
drwxr-xr-x ...

Checking: private
drwx------ ...

Checking: developers
drwxrws--- ...

Checking: shared
drwxrwxrwt ...
```

Important line:

```bash
for dir in public private developers shared
```

The variable `$dir` becomes:

```text
public
private
developers
shared
```

one at a time.

### Question

**What problem does a loop solve?**

A loop prevents us from writing the same commands repeatedly.

Instead of:

```bash
ls -ld public
ls -ld private
ls -ld developers
ls -ld shared
```

we use:

```bash
for dir in public private developers shared
do
    ls -ld "$dir"
done
```

---

# Part 5 — Inspect Important Files With a Loop

Extend `directory-check.sh`.

Use:

```bash
#!/bin/bash

echo "===== DIRECTORY CHECK ====="
echo

echo "--- DIRECTORIES ---"

for dir in public private developers shared
do
    echo "Checking directory: $dir"
    ls -ld "$dir"
    echo
done

echo "--- IMPORTANT FILES ---"

for file in public/announcement.txt private/secrets.txt developers/project.txt developers/deploy.sh
do
    echo "Checking file: $file"
    ls -l "$file"
    echo
done
```

Run:

```bash
./directory-check.sh
```

Now one script checks both directories and files.

---

# Part 6 — Disk and Memory Check

Create:

```bash
nano resource-check.sh
```

Add:

```bash
#!/bin/bash

echo "===== RESOURCE CHECK ====="
echo

echo "--- UPTIME ---"
uptime
echo

echo "--- MEMORY ---"
free -h
echo

echo "--- DISK USAGE ---"
df -h
echo
```

Make executable and run:

```bash
chmod +x resource-check.sh
./resource-check.sh
```

### Memory

```bash
free -h
```

Example:

```text
               total        used        free      shared  buff/cache   available
Mem:           949Mi       280Mi       340Mi        10Mi       329Mi       530Mi
Swap:             0B          0B          0B
```

`-h` means **human-readable**.

### Disk

```bash
df -h
```

Example:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       8G   3.2G  4.8G  41% /
```

### Questions

**Which command displays memory usage?**

```bash
free -h
```

**Which command displays filesystem usage?**

```bash
df -h
```

**Why should a system administrator monitor free disk space?**

Because if a filesystem becomes full, applications may fail to create files, logs may stop working, databases may fail, and the operating system may become unstable.

---

# Part 7 — Process Check

Create:

```bash
nano process-check.sh
```

Add:

```bash
#!/bin/bash

echo "===== PROCESS INFORMATION ====="
echo

echo "Current User:"
whoami
echo

echo "--- RUNNING PROCESSES ---"
ps aux
```

Run:

```bash
chmod +x process-check.sh
./process-check.sh
```

## Challenge

Start:

```bash
sleep 300 &
```

You may see:

```text
[1] 2431
```

`2431` is the process ID.

Search for it:

```bash
ps aux | grep "sleep 300"
```

A cleaner search is:

```bash
ps aux | grep "[s]leep 300"
```

Or:

```bash
ps -ef | grep "[s]leep 300"
```

### What is a PID?

PID means **Process ID**. Every running process receives a unique numeric identifier.

### What does `&` do?

```bash
sleep 300 &
```

runs the command in the **background**, leaving the terminal available for other commands.

### Why inspect running processes?

Administrators may need to identify high CPU usage, high memory usage, stuck programs, unexpected services, or failed applications.

---

# Part 8 — Network Information

Create:

```bash
nano network-info.sh
```

Add:

```bash
#!/bin/bash

echo "===== NETWORK INFORMATION ====="
echo

echo "Hostname:"
hostname
echo

echo "--- INTERFACES ---"
ip link show
echo

echo "--- IP ADDRESSES ---"
ip addr show
echo

echo "--- ROUTING ---"
ip route show
echo
```

Make executable:

```bash
chmod +x network-info.sh
```

Run:

```bash
./network-info.sh
```

## Hostname

```bash
hostname
```

## Interfaces

```bash
ip link show
```

Possible interfaces include `lo`, `eth0`, or `ens5`.

## IP addresses

```bash
ip addr show
```

Example:

```text
inet 172.31.15.44/20
```

## Routing

```bash
ip route show
```

Example:

```text
default via 172.31.0.1 dev ens5
172.31.0.0/20 dev ens5 proto kernel scope link
```

### Questions

**What is an IP address?**

An address used to identify a device/interface on an IP network.

**What is a default route?**

The route used when no more specific route matches the destination.

**Why does an administrator need network configuration?**

It helps troubleshoot internet access, application connectivity, routing, gateway, and interface configuration issues.

---

# Part 9 — User Input

Create:

```bash
nano inspect-path.sh
```

Add:

```bash
#!/bin/bash

echo "===== PATH INSPECTION ====="
echo

echo "Enter a file or directory to inspect:"
read target

echo
echo "Inspecting: $target"
echo

ls -ld "$target"
```

Run:

```bash
chmod +x inspect-path.sh
./inspect-path.sh
```

Test with:

```text
public
private
developers
shared
developers/project.txt
```

Important:

```bash
read target
```

stores keyboard input in the variable `$target`.

---

# Part 10 — Script Arguments

Create:

```bash
nano quick-check.sh
```

Add:

```bash
#!/bin/bash

target=$1

echo "===== QUICK CHECK ====="
echo
echo "Target: $target"
echo

ls -ld "$target"
```

Run:

```bash
chmod +x quick-check.sh
./quick-check.sh developers
```

Or:

```bash
./quick-check.sh private/secrets.txt
```

### What is `$1`?

For:

```bash
./quick-check.sh developers
```

`$1` is `developers`.

### What would `$2` represent?

For:

```bash
./example.sh apple orange
```

```text
$1 = apple
$2 = orange
```

### Why are arguments useful?

They allow administrators to automate commands without stopping for interactive input.

---

# Part 11 — `if / elif / else`

Create:

```bash
nano admin-check.sh
```

Add:

```bash
#!/bin/bash

echo "Which area do you want to inspect?"
echo "public"
echo "private"
echo "developers"
echo "shared"

read area

echo

if [ "$area" = "public" ]
then
    echo "Public area selected."
    echo "Checking public files..."
    ls -ld public

elif [ "$area" = "private" ]
then
    echo "Private area selected."
    echo "Checking restricted files..."
    ls -ld private

elif [ "$area" = "developers" ]
then
    echo "Development workspace selected."
    echo "Checking collaborative files..."
    ls -ld developers

elif [ "$area" = "shared" ]
then
    echo "Shared directory selected."
    echo "Checking shared workspace..."
    ls -ld shared

else
    echo "Unknown area selected."
fi
```

Run:

```bash
chmod +x admin-check.sh
./admin-check.sh
```

---

# Part 12 — Administration Menu

Create:

```bash
nano admin-tool.sh
```

Use:

```bash
#!/bin/bash

echo "===== LINUX ADMINISTRATION TOOL ====="
echo
echo "1. Show system information"
echo "2. Show disk usage"
echo "3. Show memory usage"
echo "4. Show network information"
echo "5. Show running processes"
echo "6. Inspect project directories"
echo "7. Show current user and groups"
echo "8. Exit"
echo

echo "Choose an option:"
read option

case "$option" in

    1)
        echo "===== SYSTEM INFORMATION ====="
        echo "User: $(whoami)"
        echo "Hostname: $(hostname)"
        echo "Date: $(date)"
        echo "Kernel: $(uname -r)"
        uptime
        ;;

    2)
        echo "===== DISK USAGE ====="
        df -h
        ;;

    3)
        echo "===== MEMORY USAGE ====="
        free -h
        ;;

    4)
        echo "===== NETWORK INFORMATION ====="
        ip addr show
        echo
        ip route show
        ;;

    5)
        echo "===== RUNNING PROCESSES ====="
        ps aux
        ;;

    6)
        echo "===== PROJECT DIRECTORIES ====="

        for dir in public private developers shared
        do
            echo
            echo "Checking: $dir"
            ls -ld "$dir"
        done
        ;;

    7)
        echo "===== USER INFORMATION ====="
        whoami
        id
        ;;

    8)
        echo "Exiting administration tool."
        exit 0
        ;;

    *)
        echo "Invalid option."
        ;;

esac
```

Make executable:

```bash
chmod +x admin-tool.sh
```

Run:

```bash
./admin-tool.sh
```

---

# Part 13 — Create an Administration Report

Create:

```bash
nano system-report.sh
```

Use:

```bash
#!/bin/bash

echo "================================" > system-report.txt
echo "       SYSTEM REPORT" >> system-report.txt
echo "================================" >> system-report.txt
echo >> system-report.txt

echo "Date: $(date)" >> system-report.txt
echo "Hostname: $(hostname)" >> system-report.txt
echo "Current User: $(whoami)" >> system-report.txt
echo >> system-report.txt

echo "--- SYSTEM ---" >> system-report.txt
echo "Kernel: $(uname -r)" >> system-report.txt
echo "Uptime:" >> system-report.txt
uptime >> system-report.txt
echo >> system-report.txt

echo "--- MEMORY ---" >> system-report.txt
free -h >> system-report.txt
echo >> system-report.txt

echo "--- DISK USAGE ---" >> system-report.txt
df -h >> system-report.txt
echo >> system-report.txt

echo "--- NETWORK ---" >> system-report.txt
ip addr show >> system-report.txt
echo >> system-report.txt
ip route show >> system-report.txt
echo >> system-report.txt

echo "--- USERS ---" >> system-report.txt
whoami >> system-report.txt
id >> system-report.txt
echo >> system-report.txt

echo "--- PROCESSES ---" >> system-report.txt
ps aux >> system-report.txt
echo >> system-report.txt

echo "--- DIRECTORY INFORMATION ---" >> system-report.txt

for dir in public private developers shared
do
    echo "Directory: $dir" >> system-report.txt
    ls -ld "$dir" >> system-report.txt
    echo >> system-report.txt
done

echo "================================" >> system-report.txt
echo "       END OF REPORT" >> system-report.txt
echo "================================" >> system-report.txt

echo "Report created: system-report.txt"
```

Run:

```bash
chmod +x system-report.sh
./system-report.sh
```

View it:

```bash
cat system-report.txt
```

or:

```bash
less system-report.txt
```

## `>` versus `>>`

```bash
echo "Hello" > file.txt
```

creates or **overwrites** the file.

```bash
echo "World" >> file.txt
```

**appends** to the file.

### Why save reports?

A stored report provides historical evidence, troubleshooting information, audit data, and system state snapshots.

### Why include date and hostname?

They identify **which machine** produced the report and **when** it was generated.

---

# Part 14 — Simulated Administration Problem

First inspect:

```bash
ls -l developers/project.txt
```

Expected original state:

```text
-rw-rw---- ...
```

That corresponds to permission `660`.

Now simulate the mistake:

```bash
chmod 777 developers/project.txt
```

Inspect:

```bash
ls -l developers/project.txt
```

You should now see:

```text
-rwxrwxrwx ...
```

Run:

```bash
./file-audit.sh
```

Then fix it:

```bash
chmod 660 developers/project.txt
```

Inspect again:

```bash
ls -l developers/project.txt
```

Now:

```text
-rw-rw----
```

Run the audit again:

```bash
./file-audit.sh
```

### What changed?

Before:

```text
660
-rw-rw----
```

During misconfiguration:

```text
777
-rwxrwxrwx
```

After repair:

```text
660
-rw-rw----
```

### Why is `777` dangerous?

It gives read, write, and execute permissions to owner, group, and others. On a multi-user system, that is unnecessarily permissive and can allow unauthorized modification or execution.

---

# Final Project — Linux Administration Toolkit

Create:

```bash
nano linux-admin.sh
```

Use this solution:

```bash
#!/bin/bash

current_user=$(whoami)
computer_name=$(hostname)
current_date=$(date)

target=$1

echo "===================================="
echo "       LINUX ADMIN TOOLKIT"
echo "===================================="
echo
echo "Administrator: $current_user"
echo "Hostname: $computer_name"
echo "Date: $current_date"
echo


# -----------------------------------
# OPTIONAL COMMAND-LINE TARGET
# -----------------------------------

if [ "$target" = "private" ]
then
    echo "Private area supplied as argument."
    echo "Checking restricted area:"
    ls -ld private
    echo

elif [ "$target" = "developers" ]
then
    echo "Developers area supplied as argument."
    echo "Checking development workspace:"
    ls -ld developers
    echo

elif [ "$target" = "shared" ]
then
    echo "Shared area supplied as argument."
    echo "Checking shared workspace:"
    ls -ld shared
    echo

elif [ "$target" = "public" ]
then
    echo "Public area supplied as argument."
    echo "Checking public workspace:"
    ls -ld public
    echo

else
    echo "No recognized project directory supplied as argument."
    echo
fi


# -----------------------------------
# ADMINISTRATION MENU
# -----------------------------------

while true
do

    echo
    echo "===================================="
    echo "       LINUX ADMIN TOOLKIT"
    echo "===================================="
    echo
    echo "1. System information"
    echo "2. Resource usage"
    echo "3. Network information"
    echo "4. Process information"
    echo "5. File and directory information"
    echo "6. User information"
    echo "7. Generate system report"
    echo "8. Exit"
    echo
    echo "Choose an option:"
    read option

    case "$option" in

        1)
            echo
            echo "===== SYSTEM INFORMATION ====="
            echo "Current User: $current_user"
            echo "Hostname: $computer_name"
            echo "Date: $(date)"
            echo "Kernel: $(uname -r)"
            echo

            echo "User ID and Groups:"
            id
            echo

            echo "Uptime:"
            uptime
            ;;


        2)
            echo
            echo "===== RESOURCE USAGE ====="

            echo
            echo "--- UPTIME ---"
            uptime

            echo
            echo "--- MEMORY ---"
            free -h

            echo
            echo "--- DISK USAGE ---"
            df -h
            ;;


        3)
            echo
            echo "===== NETWORK INFORMATION ====="

            echo
            echo "Hostname:"
            hostname

            echo
            echo "--- INTERFACES ---"
            ip link show

            echo
            echo "--- IP ADDRESSES ---"
            ip addr show

            echo
            echo "--- ROUTING ---"
            ip route show
            ;;


        4)
            echo
            echo "===== PROCESS INFORMATION ====="

            echo
            echo "Current User:"
            whoami

            echo
            echo "--- RUNNING PROCESSES ---"
            ps aux
            ;;


        5)
            echo
            echo "===== FILE AND DIRECTORY INFORMATION ====="

            echo
            echo "--- DIRECTORIES ---"

            for dir in public private developers shared
            do
                echo
                echo "Checking: $dir"
                ls -ld "$dir"
            done

            echo
            echo "--- IMPORTANT FILES ---"

            for file in public/announcement.txt private/secrets.txt developers/project.txt developers/deploy.sh
            do
                echo
                echo "Checking: $file"
                ls -l "$file"
            done
            ;;


        6)
            echo
            echo "===== USER INFORMATION ====="
            echo

            echo "Current User:"
            whoami

            echo
            echo "User ID and Groups:"
            id
            ;;


        7)
            echo "================================" > system-report.txt
            echo "       SYSTEM REPORT" >> system-report.txt
            echo "================================" >> system-report.txt
            echo >> system-report.txt

            echo "Date: $(date)" >> system-report.txt
            echo "Hostname: $(hostname)" >> system-report.txt
            echo "Current User: $(whoami)" >> system-report.txt
            echo >> system-report.txt

            echo "--- SYSTEM ---" >> system-report.txt
            echo "Kernel: $(uname -r)" >> system-report.txt
            echo "Uptime:" >> system-report.txt
            uptime >> system-report.txt
            echo >> system-report.txt

            echo "--- MEMORY ---" >> system-report.txt
            free -h >> system-report.txt
            echo >> system-report.txt

            echo "--- DISK USAGE ---" >> system-report.txt
            df -h >> system-report.txt
            echo >> system-report.txt

            echo "--- NETWORK ---" >> system-report.txt
            ip addr show >> system-report.txt
            echo >> system-report.txt
            ip route show >> system-report.txt
            echo >> system-report.txt

            echo "--- USERS ---" >> system-report.txt
            whoami >> system-report.txt
            id >> system-report.txt
            echo >> system-report.txt

            echo "--- PROCESSES ---" >> system-report.txt
            ps aux >> system-report.txt
            echo >> system-report.txt

            echo "--- DIRECTORY INFORMATION ---" >> system-report.txt

            for dir in public private developers shared
            do
                echo "Directory: $dir" >> system-report.txt
                ls -ld "$dir" >> system-report.txt
                echo >> system-report.txt
            done

            echo "================================" >> system-report.txt
            echo "       END OF REPORT" >> system-report.txt
            echo "================================" >> system-report.txt

            echo
            echo "System report created successfully."
            echo "File: system-report.txt"
            ;;


        8)
            echo
            echo "Exiting Linux Administration Toolkit."
            echo "Goodbye!"
            break
            ;;


        *)
            echo
            echo "Invalid option."
            echo "Please select a number from 1 to 8."
            ;;

    esac

done
```

Make executable:

```bash
chmod +x linux-admin.sh
```

Run normally:

```bash
./linux-admin.sh
```

Or test the `$1` requirement:

```bash
./linux-admin.sh developers
```

You can also test:

```bash
./linux-admin.sh private
./linux-admin.sh shared
./linux-admin.sh public
```

---

# Verify Final Project Requirements

| Requirement | Example in script |
|---|---|
| Shebang | `#!/bin/bash` |
| Variable | `current_user=$(whoami)` |
| Command substitution | `$(hostname)` |
| `for` loop | directory/file inspection |
| `if / elif / else` | command-line target section |
| `read` | `read option` |
| `$1` | `target=$1` |
| `case` | menu |
| Admin commands | `ps`, `df`, `free`, `ip`, etc. |
| Redirection | `> system-report.txt`, `>>` |

The prohibited features remain unused:

```text
Arrays
Functions
Bash file tests such as -f, -d, -r, -w, -x
Debugging tools
```

---

# Final Questions — Suggested Answers

**1. Why is Bash useful for Linux system administrators?**

Bash can combine Linux commands into reusable scripts and automate repetitive administration tasks.

**2. What is the benefit of automating repeated commands?**

Automation saves time, reduces repetitive typing, and makes operations more consistent.

**3. What is a variable?**

A variable stores a value that can be reused later.

```bash
username=$(whoami)
```

**4. What does command substitution do?**

It runs a command and stores or inserts its output.

```bash
hostname=$(hostname)
```

**5. What does `$1` represent?**

The first command-line argument supplied to a script.

**6. Why are loops useful?**

Loops allow the same operation to be applied to multiple items without repeating code.

**7. When would you use `if / else`?**

When the script needs to make a decision based on a condition.

**8. When would you use `case`?**

When there are several known choices, such as an administration menu.

**9. Why should administrators monitor disk and memory usage?**

To detect resource shortages before they cause application or system problems.

**10. Why should administrators inspect running processes?**

To identify active applications, resource usage, failed or stuck processes, or unexpected activity.

**11. Why is network information important during troubleshooting?**

It helps identify problems with IP addresses, interfaces, routes, gateways, and connectivity.

**12. Why should administrative scripts create logs or reports?**

Reports provide evidence and historical information for troubleshooting and auditing.

**13. What could happen if an administrator writes an incorrect automated script?**

It could modify or delete files, change permissions, stop services, or affect many systems very quickly.

**14. Why should scripts be tested before being used on important systems?**

Testing helps detect errors before the script is used on important or production systems.

---

# Project Goal

The goal is to take Linux administration commands that are normally executed manually and turn them into **repeatable administrative tools**.

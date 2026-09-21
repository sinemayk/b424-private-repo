# Hands-on Linux-01: Linux Essentials for AWS-DevOps

## Goal

The goal of this hands-on training is to help students practice the Linux fundamentals introduced in Day 1 and connect them directly to AWS/DevOps work.

---

## Learning Outcomes

By the end of this hands-on training, students will be able to:

- Identify the current Linux distribution and kernel version.
- Identify the current user, hostname, and working directory.
- Understand the difference between the Linux kernel and a Linux distribution.
- Navigate the Linux filesystem using absolute and relative paths.
- Use basic shell commands such as `pwd`, `ls`, `cd`, `mkdir`, and `touch`.
- Work with hidden files and directories.
- Use common `ls` options such as `-l`, `-a`, `-h`, and combinations such as `-lah`.
- Understand the purpose of `.` , `..` , `~` , and `/`.
- Use simple shell globbing with `*`, `?`, and `[]`.
- Distinguish globbing from brace expansion.
- Perform a small AWS/EC2-style Linux environment inspection.

---

## Environment

You can complete this lab on any Linux environment, including:

- Amazon Linux 2023 EC2 instance
- Ubuntu EC2 instance
- AWS CloudShell
- WSL
- Local Linux virtual machine

> For AWS-DevOps students, an EC2 instance or AWS CloudShell is recommended.

---

# Part 1 - Inspect the Linux System

## Step 1 - Check the Current User

Run:

```bash
whoami
```

This command displays the username of the current user.

Example:

```text
ec2-user
```

or:

```text
ubuntu
```

---

## Step 2 - Check the Hostname

Run:

```bash
hostname
```

The hostname identifies the current system.

On an EC2 instance, the hostname may look similar to:

```text
ip-172-31-10-25
```

---

## Step 3 - Check the Linux Distribution

Run:

```bash
cat /etc/os-release
```

This file contains information about the Linux distribution.

Example on Amazon Linux 2023:

```text
NAME="Amazon Linux"
VERSION="2023"
ID="amzn"
```

Example on Ubuntu:

```text
NAME="Ubuntu"
VERSION="24.04 LTS"
ID=ubuntu
```

### Question

Is the system running:

- Amazon Linux?
- Ubuntu?
- Another Linux distribution?

Write your answer below:

```text
Distribution:
Version:
```

---

## Step 4 - Check the Linux Kernel

Run:

```bash
uname -r
```

Example:

```text
6.x.x-xxx.amzn2023.x86_64
```

The distribution and the Linux kernel are not the same thing.

A Linux distribution combines components such as:

```text
Linux Kernel
+
System Utilities
+
Libraries
+
Package Manager
+
Applications
```

---

## Step 5 - Display More System Information

Run:

```bash
uname -a
```

This provides additional information about the kernel and system architecture.

---

# Part 2 - Understand the Shell Prompt

Your Linux prompt may look similar to:

```text
ec2-user@ip-172-31-10-25:~$
```

or:

```text
ubuntu@ip-172-31-10-25:~$
```

Typical components are:

```text
username@hostname:current_directory$
```

The final symbol usually indicates the user type:

```text
$   normal user
#   root/privileged user
```

Check your current user again:

```bash
whoami
```

---

# Part 3 - Working Directory and Filesystem Basics

Linux uses a hierarchical filesystem.

Important symbols:

| Symbol | Meaning |
|---|---|
| `/` | Root of the filesystem |
| `~` | Current user's home directory |
| `.` | Current directory |
| `..` | Parent directory |

---

## Step 1 - Show the Current Directory

Run:

```bash
pwd
```

`pwd` means:

```text
Print Working Directory
```

---

## Step 2 - Move to the Root Directory

Run:

```bash
cd /
```

Then verify:

```bash
pwd
```

Expected result:

```text
/
```

List the directories under `/`:

```bash
ls
```

You may see directories such as:

```text
bin
boot
dev
etc
home
opt
tmp
usr
var
```

---

## Step 3 - Return to the Home Directory

Run:

```bash
cd ~
```

or simply:

```bash
cd
```

Verify:

```bash
pwd
```

---

## Step 4 - Move to the Parent Directory

Create a practice directory first:

```bash
mkdir linux-day1
cd linux-day1
```

Now check:

```bash
pwd
```

Move one level up:

```bash
cd ..
```

Check again:

```bash
pwd
```

---

## Step 5 - Use `cd -`

Move to `/tmp`:

```bash
cd /tmp
```

Now return to your previous directory:

```bash
cd -
```

Run again:

```bash
cd -
```

`cd -` switches between the current directory and the previous directory.

This is very useful during administration work.

---

# Part 4 - Command Anatomy

A Linux command commonly follows this structure:

```text
command [options] [arguments]
```

Example:

```bash
ls -l /var/log
```

Breakdown:

```text
ls          command
-l          option
/var/log    argument
```

Try:

```bash
ls -l /etc
```

Then:

```bash
ls -lah /var/log
```

---

# Part 5 - Listing Files and Directories

Return to your home directory:

```bash
cd ~
```

Create a practice directory:

```bash
mkdir -p linux-day1/listing
cd linux-day1/listing
```

Create some files:

```bash
touch file1.txt
touch file2.txt
touch notes.log
```

Now run:

```bash
ls
```

Try long format:

```bash
ls -l
```

Try human-readable file sizes:

```bash
ls -lh
```

---

# Part 6 - Hidden Files

In Linux, files and directories beginning with `.` are normally hidden.

Create hidden files:

```bash
touch .secret
touch .env
mkdir .config-demo
```

Run:

```bash
ls
```

Notice that hidden items are not displayed.

Now run:

```bash
ls -a
```

Then:

```bash
ls -la
```

Finally:

```bash
ls -lah
```

### Question

What is the difference between these commands?

```bash
ls
ls -l
ls -a
ls -lah
```

---

# Part 7 - Creating Directories

Go to the Day 1 directory:

```bash
cd ~/linux-day1
```

Create a directory:

```bash
mkdir project
```

Create nested directories:

```bash
mkdir -p project/config/nginx
```

Verify:

```bash
ls -R project
```

If `tree` is available, you can also try:

```bash
tree project
```

---

# Part 8 - Understanding `touch`

The `touch` command is commonly used to create an empty file, but its main purpose is to update file timestamps.

Create a file:

```bash
cd ~/linux-day1/project
touch app.log
```

Check its details:

```bash
ls -l app.log
```

Wait a few seconds and run:

```bash
touch app.log
```

Check again:

```bash
ls -l app.log
```

The file already existed, so `touch` updated its timestamp instead of creating a second file.

Create multiple empty files:

```bash
touch dev.txt test.txt prod.txt
```

Verify:

```bash
ls -l
```

---

# Part 9 - Absolute and Relative Paths

Assume your current directory is:

```text
/home/ec2-user/linux-day1/project
```

or:

```text
/home/ubuntu/linux-day1/project
```

## Absolute Path

An absolute path starts from `/`.

Example:

```bash
cd /var/log
```

Check:

```bash
pwd
```

---

## Relative Path

A relative path depends on your current location.

Return to the project directory:

```bash
cd ~/linux-day1/project
```

Create:

```bash
mkdir -p config/app
```

Move using a relative path:

```bash
cd config/app
```

Check:

```bash
pwd
```

Go two levels up:

```bash
cd ../..
```

---

# Part 10 - Simple Globbing

Create a new directory:

```bash
mkdir -p ~/linux-day1/globbing
cd ~/linux-day1/globbing
```

Create files:

```bash
touch file1.txt
touch file2.txt
touch file3.txt
touch fileA.txt
touch fileB.txt
touch app.log
touch api.log
touch database.log
touch notes.md
```

---

## `*` - Match Zero or More Characters

List all `.log` files:

```bash
ls *.log
```

List files beginning with `file`:

```bash
ls file*
```

---

## `?` - Match Exactly One Character

Run:

```bash
ls file?.txt
```

This matches names such as:

```text
file1.txt
fileA.txt
```

---

## `[]` - Match One Character from a Set or Range

Run:

```bash
ls file[1-3].txt
```

Try:

```bash
ls file[A-B].txt
```

---

# Part 11 - Brace Expansion

Brace expansion is different from globbing.

Globbing matches existing filenames.

Brace expansion generates text before the command is executed.

Create files:

```bash
touch server{1..5}.log
```

Verify:

```bash
ls server*
```

Create environment directories:

```bash
mkdir -p {dev,test,prod}
```

Verify:

```bash
ls
```

Create month/year combinations:

```bash
touch {jan,feb,mar}-{2025..2026}.txt
```

Verify:

```bash
ls *.txt
```

---

# Part 12 - `type` and Command Discovery

The shell does not treat every command in exactly the same way.

Run:

```bash
type cd
```

Then:

```bash
type echo
```

Then:

```bash
type ls
```

Depending on your environment, the output may show that a command is:

- a shell builtin
- an alias
- an external executable

Try:

```bash
which ls
```

Compare the output of:

```bash
type ls
which ls
```

---

# Part 13 - AWS/EC2 Mini-Lab

Pretend that you have just connected to an unfamiliar EC2 Linux server.

Your task is to quickly understand the environment.

Run the following commands one by one:

```bash
whoami
hostname
pwd
cat /etc/os-release
uname -r
ls
ls -la
```

Now create a small application workspace:

```bash
mkdir -p ~/aws-devops-app/config
cd ~/aws-devops-app
touch app.log
touch README.md
touch config/app.conf
```

Verify:

```bash
pwd
ls -lah
ls -lah config
```

Create environment-specific directories:

```bash
mkdir -p environments/{dev,test,prod}
```

Verify:

```bash
ls environments
```

Create log files:

```bash
touch app-{1..5}.log
```

List only the log files:

```bash
ls *.log
```

---

# Part 14 - Day 1 Challenge

Complete the following tasks without copying the commands from previous sections.

## Task 1

Create this directory structure inside your home directory:

```text
devops-lab/
├── config/
├── logs/
├── scripts/
└── environments/
    ├── dev/
    ├── test/
    └── prod/
```

---

## Task 2

Inside `logs`, create:

```text
app1.log
app2.log
app3.log
error.log
access.log
```

Use brace expansion where appropriate.

---

## Task 3

Inside `config`, create:

```text
app.conf
db.conf
.env
```

---

## Task 4

Display:

1. Your current working directory.
2. All files including hidden files.
3. Only `.log` files.
4. Only files starting with `app`.
5. The Linux distribution.
6. The Linux kernel version.
7. The current user.
8. The hostname.

---

## Task 5

From the `devops-lab/logs` directory:

- Move to the parent directory using a relative path.
- Move to `/var/log` using an absolute path.
- Return to the previous directory using `cd -`.
- Return to your home directory using `~`.

---

# Part 15 - Knowledge Check

Answer the following questions.

### 1. What is the difference between Linux and a Linux distribution?

---

### 2. What does `/` represent?

---

### 3. What does `~` represent?

---

### 4. What does `..` represent?

---

### 5. What is the difference between an absolute path and a relative path?

---

### 6. What is the difference between these commands?

```bash
ls
ls -l
ls -a
ls -lah
```

---

### 7. What does `touch` do if the file already exists?

---

### 8. What does this pattern mean?

```bash
*.log
```

---

### 9. What does this pattern mean?

```bash
file?.txt
```

---

### 10. Is this globbing or brace expansion?

```bash
touch server{1..5}.log
```

---

### 11. Which command shows the Linux distribution?

---

### 12. Which command shows the Linux kernel version?

---

# Part 16 - Cleanup

If you want to remove the practice files after finishing the lab:

```bash
cd ~
rm -rf linux-day1
rm -rf aws-devops-app
rm -rf devops-lab
```

> Be very careful with `rm -rf`. Always verify the path before running it.

---

# Summary

In this hands-on session, you practiced:

```text
whoami
hostname
pwd
cat /etc/os-release
uname -r
uname -a
ls
ls -l
ls -a
ls -lah
cd
cd ..
cd /
cd ~
cd -
mkdir
mkdir -p
touch
type
which
```

You also practiced:

```text
Absolute paths
Relative paths
Hidden files
Shell globbing
Brace expansion
Basic EC2/Linux environment inspection
```

These fundamentals will be used repeatedly throughout the rest of the Linux and AWS-DevOps course.

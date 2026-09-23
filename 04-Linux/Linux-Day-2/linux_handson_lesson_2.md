# Hands-on Linux-02: Command Line Basics, SSH, Downloads, Archives, and Help

## Goal

The goal of this hands-on training is to help students practice the Day 2 Linux commands used frequently in AWS and DevOps environments.

---

## Learning Outcomes

By the end of this hands-on session, students will be able to:

- Remove files and directories safely.
- Copy, move, and rename files and directories.
- Use `echo` and basic output redirection.
- Display and combine file contents with `cat`.
- Test network reachability with `ping`.
- Connect to Amazon Linux and Ubuntu EC2 instances with SSH.
- Download files with `wget` and `curl`.
- Create and extract archives with `tar`.
- Extract ZIP archives with `unzip`.
- Download and install AWS CLI v2 as a practical example.
- Find Linux command documentation with `--help`, `man`, `info`, `type`, `which`, and `command -v`.

---

# Part 1 - Prepare the Lab Workspace

Go to your home directory:

```bash
cd ~
```

Create a Day 2 lab directory:

```bash
mkdir -p linux-day2
cd linux-day2
```

Verify your location:

```bash
pwd
```

---

# Part 2 - Removing Empty Directories with `rmdir`

Create an empty directory:

```bash
mkdir empty-dir
```

Verify:

```bash
ls
```

Remove it:

```bash
rmdir empty-dir
```

Verify again:

```bash
ls
```

`rmdir` removes **empty directories only**.

Try:

```bash
mkdir test-dir
touch test-dir/file.txt
rmdir test-dir
```

The command should fail because the directory is not empty.

---

# Part 3 - Removing Files with `rm`

Create a file:

```bash
touch old.log
```

Verify:

```bash
ls -l
```

Remove it:

```bash
rm old.log
```

Verify:

```bash
ls -l
```

> Linux command-line deletion normally does not send files to a recycle bin.

---

## Interactive Removal

Create another file:

```bash
touch important.txt
```

Run:

```bash
rm -i important.txt
```

You should be asked to confirm the deletion.

---

## Recursive Removal

Create a directory with files:

```bash
mkdir -p old-app/logs
touch old-app/app.conf
touch old-app/logs/app.log
```

Try:

```bash
rm old-app
```

This should fail because `old-app` is a directory.

Remove the directory recursively:

```bash
rm -r old-app
```

Verify:

```bash
ls
```

---

## Important Safety Note

You may also see:

```bash
rm -rf directory-name
```

Options:

```text
-r   recursive
-f   force
```

Use `rm -rf` carefully.

Before destructive commands, develop this habit:

```bash
pwd
ls
```

Confirm that you are in the correct location before deleting anything.

---

# Part 4 - Copying Files with `cp`

Create a file:

```bash
echo "APP_ENV=dev" > app.conf
```

Display it:

```bash
cat app.conf
```

Copy it:

```bash
cp app.conf app.conf.bak
```

Verify:

```bash
ls -l
```

Display both:

```bash
cat app.conf
cat app.conf.bak
```

---

## Copy a File to Another Directory

Create a backup directory:

```bash
mkdir backup
```

Copy the file:

```bash
cp app.conf backup/
```

Verify:

```bash
ls -l backup
```

---

## Copy a Directory Recursively

Create:

```bash
mkdir -p config/nginx
touch config/nginx/nginx.conf
```

Copy the entire directory:

```bash
cp -r config config-backup
```

Verify:

```bash
ls -R config-backup
```

---

# Part 5 - Moving and Renaming with `mv`

Create:

```bash
touch application.log
mkdir logs
```

Move the file:

```bash
mv application.log logs/
```

Verify:

```bash
ls logs
```

---

## Rename a File

Rename it:

```bash
mv logs/application.log logs/app.log
```

Verify:

```bash
ls -l logs
```

The `mv` command is used for both:

```text
moving
renaming
```

---

## Interactive Move

Create:

```bash
echo "old" > config.txt
echo "new" > new-config.txt
```

Try:

```bash
mv -i new-config.txt config.txt
```

If the destination exists, `mv -i` asks before overwriting it.

---

# Part 6 - `echo` and Output Redirection

Print text to the terminal:

```bash
echo "Hello DevOps"
```

Write text into a file:

```bash
echo "PORT=8080" > app.env
```

Display it:

```bash
cat app.env
```

Append another line:

```bash
echo "DEBUG=false" >> app.env
```

Display it again:

```bash
cat app.env
```

Remember:

```text
>    overwrite/create
>>   append
```

The redirection operators are handled by the shell; they are not options of `echo`.

---

# Part 7 - Working with `cat`

Create two files:

```bash
echo "line from file 1" > file1.txt
echo "line from file 2" > file2.txt
```

Display the first file:

```bash
cat file1.txt
```

Display both:

```bash
cat file1.txt file2.txt
```

Combine them into a new file:

```bash
cat file1.txt file2.txt > combined.txt
```

Verify:

```bash
cat combined.txt
```

`cat` is convenient for quick inspection of small text and configuration files.

---

# Part 8 - File Management Mini-Lab

Create this directory structure:

```text
app/
├── config/
├── logs/
└── backup/
```

Commands:

```bash
mkdir -p app/{config,logs,backup}
```

Create files:

```bash
touch app/config/app.conf
touch app/logs/app.log
```

Add configuration:

```bash
echo "APP_ENV=dev" > app/config/app.conf
```

Create a backup:

```bash
cp app/config/app.conf app/backup/app.conf.bak
```

Rename the log file:

```bash
mv app/logs/app.log app/logs/application.log
```

Verify everything:

```bash
ls -R app
```

---

# Part 9 - Testing Reachability with `ping`

Test an IP address:

```bash
ping -c 4 8.8.8.8
```

Test a hostname:

```bash
ping -c 4 example.com
```

`-c 4` tells `ping` to send four requests and stop.

---

## Troubleshooting Thought Exercise

Suppose:

```bash
ping -c 4 8.8.8.8
```

works, but:

```bash
ping -c 4 example.com
```

fails.

What could be wrong?

A likely area to investigate is:

```text
DNS resolution
```

Also remember:

> A failed ping does not automatically mean the server is down.

ICMP may be blocked by a firewall, security policy, or network configuration.

---

# Part 10 - SSH to an EC2 Instance

## Step 1 - Protect the Private Key

Assume your key file is:

```text
mykey.pem
```

Run:

```bash
chmod 400 mykey.pem
```

---

## Step 2 - Amazon Linux 2023

For Amazon Linux:

```bash
ssh -i mykey.pem ec2-user@<public-ip-or-dns>
```

Example format:

```bash
ssh -i mykey.pem ec2-user@203.0.113.10
```

---

## Step 3 - Ubuntu

For Ubuntu:

```bash
ssh -i mykey.pem ubuntu@<public-ip-or-dns>
```

The default username depends on the AMI.

Common examples:

```text
Amazon Linux 2023   ec2-user
Ubuntu              ubuntu
```

---

# Part 11 - Basic SSH Troubleshooting

If SSH fails, check the following.

## 1. Correct username

Amazon Linux:

```text
ec2-user
```

Ubuntu:

```text
ubuntu
```

---

## 2. Correct private key

Make sure you are using the key pair associated with the EC2 instance.

---

## 3. Key-file permissions

Check:

```bash
ls -l mykey.pem
```

Then:

```bash
chmod 400 mykey.pem
```

---

## 4. Security Group

For normal direct SSH access, the EC2 security group must allow inbound TCP port:

```text
22
```

Prefer allowing SSH from a narrow trusted IP range rather than the entire internet.

---

## 5. Correct destination

Verify that you are using the correct:

```text
Public IPv4 address
or
Public DNS name
```

---

# Part 12 - Downloading with `wget`

Create a downloads directory:

```bash
mkdir -p ~/linux-day2/downloads
cd ~/linux-day2/downloads
```

Download a page:

```bash
wget https://example.com/
wget https://raw.githubusercontent.com/techpro-aws-devops/Todo-List-with-Jenkins-Pipeline/main/Jenkinsfile
```

Check:

```bash
ls -l
```

---

## Choose the Local Filename

```bash
wget -O example.html https://example.com/
```

Verify:

```bash
ls -lh example.html

```

Display part of the file:

```bash
cat example.html
```

---

# Part 13 - Using `curl`

Display a web page:

```bash
curl https://example.com
```

---

## Show HTTP Headers

```bash
curl -I https://example.com
```

---

## Follow Redirects

```bash
curl -L https://example.com
```

---

## Save with a Custom Filename

```bash
curl -o page.html https://example.com
```

Verify:

```bash
ls -lh page.html
```

---

## Useful Automation Options

Example pattern:

```bash
curl -fL -o artifact.zip <URL>
```

Important options:

```text
-I    headers only
-L    follow redirects
-o    choose output filename
-O    use remote filename
-f    fail on HTTP errors
-sS   quiet output but still show errors
```

---

# Part 14 - `wget` vs `curl`

Both can download files.

Use `wget` when you want a simple non-interactive download workflow.

Use `curl` when you need more control over:

- HTTP headers
- redirects
- APIs
- request methods
- uploads
- status/error handling

As a DevOps engineer, you should be comfortable with both.

---

# Part 15 - Creating Archives with `tar`

Go back to the lab:

```bash
cd ~/linux-day2
```

Create sample files:

```bash
mkdir -p project/config
echo "APP_ENV=dev" > project/config/app.conf
echo "hello" > project/README.txt
```

---

## Create a `.tar` Archive

```bash
tar -cvf project.tar project/
```

Verify:

```bash
ls -lh project.tar
```

---

## Extract a `.tar` Archive

Create a destination directory:

```bash
mkdir extracted
```

Extract:

```bash
tar -xvf project.tar -C extracted
```

Verify:

```bash
ls -R extracted
```

---

# Part 16 - Create a Compressed `.tar.gz` Archive

Create:

```bash
tar -czvf project.tar.gz project/
```

Verify:

```bash
ls -lh project.tar.gz
```

Extract:

```bash
mkdir extracted-gzip
tar -xzvf project.tar.gz -C extracted-gzip
```

Verify:

```bash
ls -R extracted-gzip
```

Common options:

```text
-c   create
-x   extract
-v   verbose
-f   archive filename
-z   gzip compression
```

---

# Part 17 - ZIP Archives with `unzip`

Check whether `unzip` is installed:

```bash
unzip -v
```

If the command is missing, install it using the package manager appropriate for your distribution.

Amazon Linux 2023:

```bash
sudo dnf install unzip -y
```

Ubuntu:

```bash
sudo apt update
sudo apt install unzip -y
```

---

# Part 18 - Practical AWS CLI v2 Download

This lab uses the official AWS CLI v2 Linux ZIP installer.

Create a directory:

```bash
mkdir -p ~/linux-day2/awscli-install
cd ~/linux-day2/awscli-install
```

Check the machine architecture:

```bash
uname -m
```

For x86_64 Linux, download:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

Verify:

```bash
ls -lh awscliv2.zip
```

Extract:

```bash
unzip awscliv2.zip
```

Inspect:

```bash
ls
```

Install:

```bash
sudo ./aws/install
```

Verify:

```bash
aws --version
```

> If AWS CLI is already installed, the installer may require update-specific options. Follow the current AWS CLI documentation for update scenarios.

---

# Part 19 - Getting Help with `--help`

Try:

```bash
ls --help
```

Then:

```bash
cp --help
```

And:

```bash
curl --help
```

Use `--help` when you need a quick reminder about syntax and options.

---

# Part 20 - Manual Pages with `man`

Try:

```bash
man ls
```

Then:

```bash
man ssh
```

Useful keys inside `man`:

```text
/word    search
n        next match
q        quit
```

Example:

Inside:

```bash
man ssh
```

search for:

```text
/identity
```

Then press:

```text
n
```

to find the next match.

---

# Part 21 - `info`

Try:

```bash
info coreutils
```

or:

```bash
info ls
```

`info` provides GNU documentation when available.

If the command or documentation is not installed on your system, move to the next part.

---

# Part 22 - `type`, `which`, and `command -v`

Check `cd`:

```bash
type cd
```

Because `cd` is normally a shell builtin, you may see:

```text
cd is a shell builtin
```

Check `ls`:

```bash
type ls
```

Check SSH:

```bash
which ssh
```

Try:

```bash
command -v curl
```

Compare:

```bash
type curl
which curl
command -v curl
```

These commands help you understand how the shell finds and resolves commands.

---

# Part 23 - Day 2 Challenge

Complete the following without copying commands from the earlier sections.

## Task 1 - Build an Application Workspace

Create:

```text
devops-app/
├── config/
├── logs/
├── backup/
└── artifacts/
```

---

## Task 2 - Configuration

Create:

```text
devops-app/config/app.conf
```

with:

```text
APP_ENV=test
PORT=8080
```

---

## Task 3 - Backup

Copy:

```text
app.conf
```

to:

```text
backup/app.conf.bak
```

---

## Task 4 - Logs

Create:

```text
app.log
```

Then rename it to:

```text
application.log
```

---

## Task 5 - Network Check

Run a four-packet ping test against:

```text
example.com
```

---

## Task 6 - SSH Command

Write the correct SSH command for:

1. Amazon Linux 2023
2. Ubuntu

Do not connect unless you have an EC2 instance available.

---

## Task 7 - Download

Use `curl` to download:

```text
https://example.com
```

and save it as:

```text
example.html
```

---

## Task 8 - Archive

Create:

```text
devops-app.tar.gz
```

from the entire `devops-app` directory.

---

## Task 9 - Help Without Google

Find:

- The help for `tar`
- The manual page for `ssh`
- Where the `curl` executable is located

---

# Part 24 - Knowledge Check

Answer these questions.

### 1. What is the difference between `rmdir` and `rm -r`?

---

### 2. What is the difference between `cp` and `mv`?

---

### 3. What does `>` do?

---

### 4. What does `>>` do?

---

### 5. Does a failed ping always mean the remote server is down?

---

### 6. What is the default SSH username for Amazon Linux?

---

### 7. What is the default SSH username for Ubuntu?

---

### 8. What port is normally used by SSH?

---

### 9. What does `curl -I` do?

---

### 10. What does `curl -L` do?

---

### 11. What is the difference between `.tar` and `.tar.gz`?

---

### 12. Which command tells you whether something is a shell builtin, alias, function, or executable?

---

# Part 25 - Cleanup

Return home:

```bash
cd ~
```

Review before deleting:

```bash
pwd
ls
```

Remove the Day 2 lab only if you no longer need it:

```bash
rm -r ~/linux-day2
```

If you created the challenge directory separately:

```bash
rm -r ~/devops-app
```

---

# Summary

Commands practiced in Day 2:

```text
rmdir
rm
rm -i
rm -r
cp
cp -r
mv
mv -i
echo
cat
ping
ssh
wget
curl
tar
unzip
man
info
type
which
command -v
```

Important DevOps concepts practiced:

```text
Safe file deletion
File backups
Moving and renaming
Shell redirection
Reachability testing
EC2 SSH usernames
Private key permissions
Security Group awareness
Artifact downloads
Archives and compression
AWS CLI installation
Command documentation
```

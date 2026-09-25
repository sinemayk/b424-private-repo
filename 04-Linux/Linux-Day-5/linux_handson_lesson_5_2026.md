# Hands-on Linux-05: Linux Environment Variables

The goal of this hands-on session is to teach students how to work with **shell variables, environment variables, PATH, scripting, and quoting** in a Linux AWS-DevOps environment.

---

# Learning Outcomes

By the end of this hands-on lab, students will be able to:

* Explain the difference between shell variables and environment variables.
* Create, export, and remove variables.
* Access environment variables using Linux commands.
* Understand how variables are passed to child processes.
* Modify the PATH variable.
* Create custom Linux commands.
* Use environment variables inside scripts.
* Understand single and double quotes.
* Read environment variables from Python applications.

---

# Part 1 - Shell Variables and Environment Variables

## 1. Create a Shell Variable

Create a variable:

```bash
COURSE="Linux"
```

Display the value:

```bash
echo $COURSE
```

Example output:

```text
Linux
```

A shell variable exists only in the current shell session.

---

## 2. Check Variable Availability

Open another terminal.

Run:

```bash
echo $COURSE
```

The variable is empty because it was not exported.

---

## 3. Create an Environment Variable

Export the variable:

```bash
export COURSE="AWS-DevOps"
```

Check:

```bash
echo $COURSE
```

Open another shell:

```bash
bash
```

Check again:

```bash
echo $COURSE
```

The value is available because it is an environment variable.

Exit the child shell:

```bash
exit
```

---

# Part 2 - Viewing Environment Variables

## View All Environment Variables

Using:

```bash
env
```

or:

```bash
printenv
```

---

## View a Specific Variable

Examples:

```bash
printenv HOME
```

```bash
echo $USER
```

```bash
echo $PATH
```

---

## View Shell Variables and Functions

Use:

```bash
set
```

This displays:

* Shell variables
* Environment variables
* Shell functions

---

# Part 3 - Creating and Removing Variables

Create:

```bash
export TECHPRO="AWS-DevOps"
```

Check:

```bash
printenv TECHPRO
```

Remove:

```bash
unset TECHPRO
```

Verify:

```bash
printenv TECHPRO
```

---

# Part 4 - Common Linux Environment Variables

Check these variables:

```bash
echo $HOME
echo $USER
echo $SHELL
echo $PATH
echo $LANG
echo $PWD
```

Explanation:

| Variable | Purpose |
|---|---|
| HOME | User home directory |
| USER | Current username |
| SHELL | Current shell |
| PATH | Command search locations |
| LANG | Language settings |
| PWD | Current directory |

---

# Part 5 - PATH Variable

## View PATH

```bash
echo $PATH
```

Example:

```text
/usr/local/bin:/usr/bin:/bin
```

Linux searches these directories when executing commands.

---

## Find Command Location

Example:

```bash
which ls
```

Output:

```text
/usr/bin/ls
```

Check command type:

```bash
type cd
```

---

# Part 6 - Create Your Own Linux Command

Create a scripts directory:

```bash
mkdir -p ~/my_scripts
cd ~/my_scripts
```

Create a script:

```bash
nano hello-linux.sh
```

Add:

```bash
#!/bin/bash

echo "Hello from my Linux script!"
echo "Current user: $USER"
echo "Current directory: $(pwd)"
```

Save.

Make executable:

```bash
chmod +x hello-linux.sh
```

Run:

```bash
./hello-linux.sh
```

---

## Add Script Directory to PATH

Currently:

```bash
hello-linux.sh
```

will fail.

Add directory:

```bash
export PATH=$PATH:$HOME/my_scripts
```

Now run:

```bash
hello-linux.sh
```

The command works from any directory.

---

# Part 7 - Permanent PATH Configuration

Temporary:

```bash
export PATH=$PATH:$HOME/my_scripts
```

Permanent:

Edit:

```bash
nano ~/.bashrc
```

Add:

```bash
export PATH=$PATH:$HOME/my_scripts
```

Reload:

```bash
source ~/.bashrc
```

Test:

```bash
hello-linux.sh
```

---

# Part 8 - Environment Variables Inside Scripts

Create:

```bash
nano variables.sh
```

Add:

```bash
#!/bin/bash

echo "User: $USER"
echo "Home: $HOME"
echo "Course: $COURSE"
```

Make executable:

```bash
chmod +x variables.sh
```

Set variable:

```bash
export COURSE="AWS-DevOps"
```

Run:

```bash
./variables.sh
```

---

# Part 9 - Python and Environment Variables

Create:

```bash
nano env_test.py
```

Add:

```python
#!/usr/bin/env python3

import os

print("User:", os.getenv("USER"))
print("Home:", os.getenv("HOME"))
print("Course:", os.getenv("COURSE"))
```

Make executable:

```bash
chmod +x env_test.py
```

Run:

```bash
./env_test.py
```

---

# Part 10 - Quoting Variables

## Without Quotes

```bash
NAME=Linux
echo Hello $NAME
```

Output:

```text
Hello Linux
```

---

## Double Quotes

Variables are expanded:

```bash
NAME="Linux"

echo "Hello $NAME"
```

Output:

```text
Hello Linux
```

---

## Single Quotes

Variables are not expanded:

```bash
echo 'Hello $NAME'
```

Output:

```text
Hello $NAME
```

---

# Part 11 - Spaces in Variables

Wrong:

```bash
MESSAGE=Hello Linux
```

Linux interprets this incorrectly.

Correct:

```bash
MESSAGE="Hello Linux"
```

Check:

```bash
echo "$MESSAGE"
```

---

# Part 12 - Command Substitution

Store command output:

Recommended method:

```bash
DATE=$(date)
```

Display:

```bash
echo $DATE
```

Example:

```bash
CURRENT_DIR=$(pwd)

echo "Current directory is $CURRENT_DIR"
```

---

# Part 13 - AWS DevOps Examples

## AWS Region Variable

```bash
export AWS_REGION=us-east-1
```

Check:

```bash
echo $AWS_REGION
```

Used by:

* AWS CLI
* Terraform
* SDKs

---

## Application Configuration

Example:

```bash
export DATABASE_URL=mysql://localhost
export DB_USER=admin
```

Applications can read these values instead of storing passwords in source code.

---

# Practice Tasks

## Task 1 - Variables

1. Create a variable called:

```text
STUDENT
```

with your name.

2. Print it.

3. Export it.

4. Open a new shell.

5. Check if it exists.

---

## Task 2 - PATH

1. Create:

```text
~/tools
```

2. Create a script:

```text
welcome.sh
```

3. Make it executable.

4. Add the directory to PATH.

5. Run:

```bash
welcome.sh
```

without using:

```bash
./
```

---

## Task 3 - Python Script

Create:

```text
system_info.py
```

The script should display:

* Username
* Home directory
* Current directory
* Linux shell

Use:

```python
os.getenv()
```

---

## Task 4 - Quoting

Predict the output:

```bash
NAME="Linux"

echo "Operating System: $NAME"

echo 'Operating System: $NAME'
```

Explain the difference.

---

# Final Review Commands

```bash
env
printenv
set
export
unset
echo
which
type
source ~/.bashrc
chmod +x script.sh
```

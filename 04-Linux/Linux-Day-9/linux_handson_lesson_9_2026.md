# Hands-on Linux-09: Shell Scripting Conditional Statements

## Purpose

This practical training teaches students how to use Bash conditional
statements to make decisions in shell scripts.

The exercises build on the Day 8 topics:

-   Bash scripts
-   Variables
-   `read`
-   Command substitution
-   Positional parameters
-   Exit status

## Learning Outcomes

By the end of this hands-on training, students will be able to:

-   Explain how Bash conditional statements work.
-   Explain how `if` uses a command or test exit status.
-   Use `if`, `if-else`, and `if-elif-else`.
-   Use nested `if` statements.
-   Compare numbers and strings.
-   Test files and directories.
-   Combine conditions with `&&`, `||`, and `!`.
-   Test whether a command or service is available/running.
-   Validate positional parameters.
-   Use `exit` and `$?`.
-   Build simple AWS/DevOps-oriented health and deployment checks.

------------------------------------------------------------------------

# Section 1 - If Statements

## 1. How Bash `if` Works

A Bash `if` statement runs commands when a condition/test succeeds.

The important idea is:

``` text
exit status 0        → condition is true / successful
non-zero exit status → condition is false / unsuccessful
```

This connects directly to the Day 8 topic `$?`.

Basic structure:

``` bash
if [[ condition ]]; then
    commands
fi
```

For this course, use `[[ ... ]]` in new Bash scripts.

> `[ ... ]` is also valid Bash syntax because it is the traditional
> `test` command form. You will see it in existing scripts, but we will
> primarily use `[[ ... ]]`.

## 2. Create a Working Directory

``` bash
mkdir -p "$HOME/conditional-statements"
cd "$HOME/conditional-statements"
pwd
```

## 3. Create a Basic `if` Script

Create `if-statement.sh`:

``` bash
#!/usr/bin/env bash

read -r -p "Input a number: " number

if [[ $number -gt 50 ]]; then
    echo "The number is greater than 50."
fi
```

Make it executable and run it:

``` bash
chmod +x if-statement.sh
./if-statement.sh
```

Test with `75` and `25`.

## 4. Numeric Comparison Operators

  Operator   Meaning
  ---------- -----------------------
  `-eq`      Equal
  `-ne`      Not equal
  `-lt`      Less than
  `-le`      Less than or equal
  `-gt`      Greater than
  `-ge`      Greater than or equal

Examples:

``` bash
if [[ $number -eq 50 ]]; then echo "Exactly 50."; fi
if [[ $number -ne 50 ]]; then echo "Not 50."; fi
if [[ $number -lt 50 ]]; then echo "Less than 50."; fi
if [[ $number -le 50 ]]; then echo "50 or less."; fi
if [[ $number -gt 50 ]]; then echo "Greater than 50."; fi
if [[ $number -ge 50 ]]; then echo "50 or greater."; fi
```

Keep this distinction clear:

``` text
Numbers → -eq -ne -lt -le -gt -ge
Strings → == !=
```

------------------------------------------------------------------------

# Section 2 - String Conditions

## 5. String Comparison Operators

  Operator   Meaning
  ---------- ---------------------
  `==`       Equal
  `!=`       Not equal
  `-z`       String is empty
  `-n`       String is not empty

Example:

``` bash
ENVIRONMENT="production"

if [[ "$ENVIRONMENT" == "production" ]]; then
    echo "Production environment."
fi

if [[ "$ENVIRONMENT" != "development" ]]; then
    echo "Environment is not development."
fi

if [[ -z "$ENVIRONMENT" ]]; then
    echo "Environment is empty."
fi

if [[ -n "$ENVIRONMENT" ]]; then
    echo "Environment has a value."
fi
```

When expanding a string variable, normally use double quotes:

``` bash
"$ENVIRONMENT"
```

------------------------------------------------------------------------

# Section 3 - File Test Operators

## 6. Test Linux Files and Directories

  Test   Meaning
  ------ --------------------------------------
  `-e`   Path exists
  `-f`   Regular file
  `-d`   Directory
  `-r`   Readable
  `-w`   Writable
  `-x`   Executable
  `-s`   Exists and size is greater than zero

Examples:

``` bash
FILE="deployment.log"

if [[ -e "$FILE" ]]; then echo "Path exists."; fi
if [[ -f "$FILE" ]]; then echo "Regular file."; fi
if [[ -r "$FILE" ]]; then echo "Readable."; fi
if [[ -w "$FILE" ]]; then echo "Writable."; fi
if [[ -s "$FILE" ]]; then echo "Non-empty."; fi
```

Directory:

``` bash
DIRECTORY="$HOME/conditional-statements"

if [[ -d "$DIRECTORY" ]]; then
    echo "Directory exists."
fi
```

Executable:

``` bash
SCRIPT="if-statement.sh"

if [[ -x "$SCRIPT" ]]; then
    echo "Script is executable."
fi
```

## 7. Basic File Test Script

Create `file-check-basic.sh`:

``` bash
#!/usr/bin/env bash

FILE="deployment.log"

if [[ -f "$FILE" ]]; then
    echo "$FILE is a regular file."
fi

if [[ -r "$FILE" ]]; then
    echo "$FILE is readable."
fi

if [[ -w "$FILE" ]]; then
    echo "$FILE is writable."
fi
```

Run:

``` bash
chmod +x file-check-basic.sh
touch deployment.log
./file-check-basic.sh
```

------------------------------------------------------------------------

# Section 4 - Logical Operators

## 8. Combine Conditions

  Operator   Meaning
  ---------- ---------
  `&&`       AND
  `||`       OR
  `!`        NOT

AND:

``` bash
CPU=75
MEMORY=70

if [[ $CPU -gt 80 && $MEMORY -gt 80 ]]; then
    echo "CPU and memory usage are high."
fi
```

OR:

``` bash
ENVIRONMENT="production"

if [[ "$ENVIRONMENT" == "production" || "$ENVIRONMENT" == "staging" ]]; then
    echo "Deployment environment accepted."
fi
```

NOT:

``` bash
FILE="deployment.log"

if [[ ! -f "$FILE" ]]; then
    echo "$FILE does not exist."
fi
```

------------------------------------------------------------------------

# Section 5 - Command Status Conditions

## 9. Test Whether a Command Exists

``` bash
if command -v curl > /dev/null 2>&1; then
    echo "curl is installed."
else
    echo "curl is not installed."
fi
```

`command -v` returns success when Bash can find the command.

The redirection:

``` bash
> /dev/null 2>&1
```

hides the command's normal output and error output.

## 10. Check a Service

On systems using systemd:

``` bash
if systemctl is-active --quiet sshd; then
    echo "SSH service is running."
else
    echo "SSH service is not running."
fi
```

On some Ubuntu systems the service is named `ssh` rather than `sshd`.

Check the service name if necessary:

``` bash
systemctl list-units --type=service | grep -E 'ssh|sshd'
```

The important concept is that the command returns a status that `if` can
evaluate.

------------------------------------------------------------------------

# Section 6 - If-Else Statements

## 11. Basic `if-else`

``` bash
if [[ condition ]]; then
    commands
else
    other_commands
fi
```

Example:

``` bash
read -r -p "Input a number: " number

if [[ $number -ge 10 ]]; then
    echo "The number is greater than or equal to 10."
else
    echo "The number is smaller than 10."
fi
```

## 12. Practical File Example

Create `ifelse-file.sh`:

``` bash
#!/usr/bin/env bash

read -r -p "Enter a filename: " filename

if [[ -f "$filename" ]]; then
    echo "File already exists."
else
    touch "$filename"
    echo "File created."
fi
```

Run:

``` bash
chmod +x ifelse-file.sh
./ifelse-file.sh
```

Test with an existing file and a new filename.

------------------------------------------------------------------------

# Section 7 - If-Elif-Else Statements

## 13. Multiple Conditions

``` bash
if [[ condition1 ]]; then
    commands
elif [[ condition2 ]]; then
    commands
else
    commands
fi
```

## 14. Number Example

Create `elif-statement.sh`:

``` bash
#!/usr/bin/env bash

read -r -p "Input a number: " number

if [[ $number -eq 10 ]]; then
    echo "The number is equal to 10."
elif [[ $number -gt 10 ]]; then
    echo "The number is greater than 10."
else
    echo "The number is smaller than 10."
fi
```

Run:

``` bash
chmod +x elif-statement.sh
./elif-statement.sh
```

Test `10`, `20`, and `5`.

## 15. AWS/DevOps Environment Example

Create `environment-check.sh`:

``` bash
#!/usr/bin/env bash

ENVIRONMENT="${1:-}"

if [[ "$ENVIRONMENT" == "development" ]]; then
    echo "Development environment."
elif [[ "$ENVIRONMENT" == "staging" ]]; then
    echo "Staging environment."
elif [[ "$ENVIRONMENT" == "production" ]]; then
    echo "Production environment."
else
    echo "Unknown environment."
fi
```

Run:

``` bash
chmod +x environment-check.sh

./environment-check.sh development
./environment-check.sh staging
./environment-check.sh production
./environment-check.sh test
```

------------------------------------------------------------------------

# Section 8 - Nested If Statements

## 16. Nested `if`

Create `nested-if-statement.sh`:

``` bash
#!/usr/bin/env bash

SCRIPT="deploy.sh"

if [[ -e "$SCRIPT" ]]; then

    if [[ -x "$SCRIPT" ]]; then
        echo "Script exists and is executable."
    else
        echo "Script exists but is not executable."
    fi

else
    echo "Script does not exist."
fi
```

Run:

``` bash
chmod +x nested-if-statement.sh
touch deploy.sh
./nested-if-statement.sh
```

Then:

``` bash
chmod -x deploy.sh
./nested-if-statement.sh
```

Finally:

``` bash
rm deploy.sh
./nested-if-statement.sh
```

------------------------------------------------------------------------

# Section 9 - `read` + Conditions

## 17. CPU Usage Check

Create `cpu-check.sh`:

``` bash
#!/usr/bin/env bash

read -r -p "Enter CPU usage (%): " CPU

if [[ $CPU -gt 80 ]]; then
    echo "WARNING: High CPU usage."
else
    echo "CPU usage is normal."
fi
```

Run:

``` bash
chmod +x cpu-check.sh
./cpu-check.sh
```

Test `50`, `80`, and `90`.

### Challenge

Modify the script:

``` text
0-70   → Normal
71-80  → Warning
81+    → Critical
```

Use `if`, `elif`, and `else`.

------------------------------------------------------------------------

# Section 10 - Positional Parameters + Conditions

## 18. Validate Script Arguments

Create `deploy-check.sh`:

``` bash
#!/usr/bin/env bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <environment>"
    exit 1
fi

echo "Environment argument accepted: $1"
```

Run:

``` bash
chmod +x deploy-check.sh

./deploy-check.sh
./deploy-check.sh production
./deploy-check.sh production staging
```

The script should reject calls that do not provide exactly one argument.

------------------------------------------------------------------------

# Section 11 - `exit` and Exit Status

## 19. Successful Completion

Create `success.sh`:

``` bash
#!/usr/bin/env bash

echo "Check completed successfully."
exit 0
```

Run:

``` bash
chmod +x success.sh
./success.sh
echo "$?"
```

Expected:

``` text
0
```

## 20. Failed Completion

Create `failure.sh`:

``` bash
#!/usr/bin/env bash

echo "Required configuration is missing."
exit 1
```

Run:

``` bash
chmod +x failure.sh
./failure.sh
echo "$?"
```

Remember:

``` text
0        → normally means success
non-zero → indicates failure/error
```

Do not assume every failed command returns `1`.

------------------------------------------------------------------------

# Section 12 - Which Test Should I Use?

``` text
What are you checking?
        |
        +-- Number?
        |     -eq -ne -lt -le -gt -ge
        |
        +-- String?
        |     == != -z -n
        |
        +-- File/directory?
        |     -e -f -d -r -w -x -s
        |
        +-- Command?
              if command; then ...
```

Examples:

``` bash
[[ $COUNT -gt 5 ]]
```

``` bash
[[ "$ENVIRONMENT" == "production" ]]
```

``` bash
[[ -f "$CONFIG_FILE" ]]
```

``` bash
if command -v curl > /dev/null 2>&1; then
    echo "curl available"
fi
```

------------------------------------------------------------------------

# Exercise 1 --- File Check

Create:

``` text
file-check.sh
```

Ask the user for a path.

The script must determine:

1.  Whether the path exists.
2.  Whether it is a regular file.
3.  Whether it is a directory.
4.  Whether it is readable.
5.  Whether it is writable.
6.  Whether it is executable.

Use:

``` text
read -r -p
if
elif
else
[[ ]]
-e
-f
-d
-r
-w
-x
```

Test with:

``` text
/etc/hosts
/etc
/tmp
a-file-that-does-not-exist
```

------------------------------------------------------------------------

# Exercise 2 --- Server Health Check

Create:

``` text
health-check.sh
```

Check:

1.  Whether `/` exists.
2.  Whether `curl` is installed.
3.  Whether the SSH service is active.
4.  Whether `/var/log` exists.
5.  Whether `/var/log` is readable.

Expected output style:

``` text
Server Health Check
-------------------
Root filesystem: OK
curl: OK
SSH service: OK
/var/log: OK
/var/log readable: OK
```

Add a final result:

``` text
Health Check: PASSED
```

only when all checks pass.

Otherwise:

``` text
Health Check: FAILED
```

> On Ubuntu, the SSH service may be called `ssh` rather than `sshd`.

------------------------------------------------------------------------

# Exercise 3 --- Environment Validator

Create:

``` text
environment-validator.sh
```

Run:

``` bash
./environment-validator.sh production
```

Accept only:

``` text
development
staging
production
```

Requirements:

1.  Require exactly one argument.
2.  Display a usage message when the argument count is wrong.
3.  Use `if`, `elif`, and `else`.
4.  Exit `0` for an accepted environment.
5.  Exit non-zero for an invalid environment.

Test:

``` bash
./environment-validator.sh
./environment-validator.sh production
./environment-validator.sh staging
./environment-validator.sh development
./environment-validator.sh test
./environment-validator.sh production staging
```

------------------------------------------------------------------------

# Exercise 4 --- Deployment Safety Check

Create:

``` text
deployment-safety-check.sh
```

Before a deployment is allowed, check:

1.  A deployment script exists.
2.  The deployment script is executable.
3.  A configuration file exists.
4.  The selected environment is valid.
5.  `curl` is installed.

Use:

``` text
$1
$#
if
elif
else
-e
-f
-x
command -v
exit
```

Run:

``` bash
./deployment-safety-check.sh production
```

If all checks pass:

``` text
Deployment Safety Check
-----------------------
Environment: production
Deployment script: OK
Deployment script executable: OK
Configuration: OK
curl: OK

All deployment checks passed.
Ready to deploy.
```

If any check fails:

``` text
Deployment checks failed.
Deployment aborted.
```

Return a non-zero exit status when deployment should not proceed.

------------------------------------------------------------------------

# Final Review Questions

1.  What does a Bash `if` statement evaluate?
2.  What does exit status `0` normally indicate?
3.  What is the difference between `[[ $number -gt 10 ]]` and
    `[[ "$environment" == "production" ]]`?
4.  What do `-e`, `-f`, `-d`, `-r`, `-w`, `-x`, and `-s` mean?
5.  What is the difference between `&&`, `||`, and `!`?
6.  What does `if command -v curl > /dev/null 2>&1; then` do?
7.  What does `if systemctl is-active --quiet sshd; then` do?
8.  What is the purpose of `if [[ $# -ne 1 ]]; then ... exit 1; fi`?
9.  What is the difference between `[ ... ]` and `[[ ... ]]`?
10. Why is `[[ ... ]]` the primary syntax used in this course?

------------------------------------------------------------------------

# Instructor Solutions

> Complete the exercises before opening this section.

## Solution 1 --- File Check

``` bash
#!/usr/bin/env bash

read -r -p "Enter a path: " path

if [[ ! -e "$path" ]]; then
    echo "Path does not exist."
    exit 1
fi

if [[ -f "$path" ]]; then
    echo "Regular file: yes"
elif [[ -d "$path" ]]; then
    echo "Directory: yes"
else
    echo "Path exists but is neither a regular file nor a directory."
fi

if [[ -r "$path" ]]; then
    echo "Readable: yes"
else
    echo "Readable: no"
fi

if [[ -w "$path" ]]; then
    echo "Writable: yes"
else
    echo "Writable: no"
fi

if [[ -x "$path" ]]; then
    echo "Executable: yes"
else
    echo "Executable: no"
fi
```

## Solution 2 --- Server Health Check

``` bash
#!/usr/bin/env bash

status=0

echo "Server Health Check"
echo "-------------------"

if [[ -d "/" ]]; then
    echo "Root filesystem: OK"
else
    echo "Root filesystem: FAILED"
    status=1
fi

if command -v curl > /dev/null 2>&1; then
    echo "curl: OK"
else
    echo "curl: FAILED"
    status=1
fi

if systemctl is-active --quiet sshd; then
    echo "SSH service: OK"
else
    echo "SSH service: FAILED"
    status=1
fi

if [[ -d "/var/log" ]]; then
    echo "/var/log: OK"
else
    echo "/var/log: FAILED"
    status=1
fi

if [[ -r "/var/log" ]]; then
    echo "/var/log readable: OK"
else
    echo "/var/log readable: FAILED"
    status=1
fi

if [[ $status -eq 0 ]]; then
    echo "Health Check: PASSED"
else
    echo "Health Check: FAILED"
fi

exit "$status"
```

> On Ubuntu, use the actual SSH service name returned by
> `systemctl list-units --type=service | grep -E 'ssh|sshd'`.

## Solution 3 --- Environment Validator

``` bash
#!/usr/bin/env bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <development|staging|production>"
    exit 1
fi

ENVIRONMENT="$1"

if [[ "$ENVIRONMENT" == "development" ]]; then
    echo "Development environment accepted."
elif [[ "$ENVIRONMENT" == "staging" ]]; then
    echo "Staging environment accepted."
elif [[ "$ENVIRONMENT" == "production" ]]; then
    echo "Production environment accepted."
else
    echo "Invalid environment: $ENVIRONMENT"
    exit 1
fi

exit 0
```

## Solution 4 --- Deployment Safety Check

``` bash
#!/usr/bin/env bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <development|staging|production>"
    exit 1
fi

ENVIRONMENT="$1"
DEPLOY_SCRIPT="deploy.sh"
CONFIG_FILE="app.conf"
status=0

echo "Deployment Safety Check"
echo "-----------------------"
echo "Environment: $ENVIRONMENT"

if [[ "$ENVIRONMENT" == "development" ||
      "$ENVIRONMENT" == "staging" ||
      "$ENVIRONMENT" == "production" ]]; then
    echo "Environment: OK"
else
    echo "Environment: FAILED"
    status=1
fi

if [[ -f "$DEPLOY_SCRIPT" ]]; then
    echo "Deployment script: OK"
else
    echo "Deployment script: FAILED"
    status=1
fi

if [[ -x "$DEPLOY_SCRIPT" ]]; then
    echo "Deployment script executable: OK"
else
    echo "Deployment script executable: FAILED"
    status=1
fi

if [[ -f "$CONFIG_FILE" ]]; then
    echo "Configuration: OK"
else
    echo "Configuration: FAILED"
    status=1
fi

if command -v curl > /dev/null 2>&1; then
    echo "curl: OK"
else
    echo "curl: FAILED"
    status=1
fi

echo

if [[ $status -eq 0 ]]; then
    echo "All deployment checks passed."
    echo "Ready to deploy."
else
    echo "Deployment checks failed."
    echo "Deployment aborted."
fi

exit "$status"
```

------------------------------------------------------------------------

# Completion Checklist

-   [ ] Created a working directory
-   [ ] Created and executed an `if` script
-   [ ] Practiced numeric comparisons
-   [ ] Practiced string comparisons
-   [ ] Practiced file tests
-   [ ] Practiced `&&`, `||`, and `!`
-   [ ] Tested command availability with `command -v`
-   [ ] Tested service status with `systemctl`
-   [ ] Practiced `if-else`
-   [ ] Practiced `if-elif-else`
-   [ ] Practiced nested `if`
-   [ ] Combined `read` with conditions
-   [ ] Used `$1` and `$#`
-   [ ] Used `exit`
-   [ ] Checked `$?`
-   [ ] Completed Exercise 1
-   [ ] Completed Exercise 2
-   [ ] Completed Exercise 3
-   [ ] Completed Exercise 4
-   [ ] Answered the review questions
-   [ ] Completed the deployment safety challenge

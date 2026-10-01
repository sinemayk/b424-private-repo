# Hands-on Linux-08: Shell Scripting Basics

## Purpose

The aim of this practical training is to teach students how to write
basic Bash scripts and use them for simple AWS-DevOps tasks.

## Learning Outcomes

By the end of this hands-on training, students will be able to:

-   Explain the difference between a shell and Bash.
-   Create, execute, and debug Bash scripts.
-   Explain the purpose of the shebang.
-   Use shell variables and environment variables.
-   Accept interactive input with `read`.
-   Use single and double quotes correctly.
-   Use command substitution with `$(...)`.
-   Use positional and special parameters.
-   Perform basic arithmetic with Bash.
-   Use Bash scripts for simple system and DevOps tasks.

## Requirements

Use an **Amazon Linux 2023** or **Ubuntu** EC2 instance.

Check your environment:

``` bash
whoami
hostname
pwd
echo "$SHELL"
```

------------------------------------------------------------------------

# Section 1 - Shell Scripting Basics

## 1. Create a Working Directory

Create a directory for today's exercises:

``` bash
mkdir -p "$HOME/shell-scripting"
cd "$HOME/shell-scripting"
```

Verify:

``` bash
pwd
```

------------------------------------------------------------------------

## 2. Create Your First Bash Script

Create a file named:

``` text
basic.sh
```

Add:

``` bash
#!/usr/bin/env bash

echo "Hello World"
```

### About the first line

``` bash
#!/usr/bin/env bash
```

This is called the **shebang**.

It tells the system which interpreter should be used to execute the
script.

The `#!` characters identify the interpreter directive.

------------------------------------------------------------------------

## 3. Make the Script Executable

Check the current permissions:

``` bash
ls -l basic.sh
```

Make it executable:

``` bash
chmod +x basic.sh
```

Verify:

``` bash
ls -l basic.sh
```

Run it:

``` bash
./basic.sh
```

### Why do we use `./`?

The current directory is normally not included in `PATH`.

Therefore:

``` bash
./basic.sh
```

means:

> Execute `basic.sh` from the current directory.

------------------------------------------------------------------------

## 4. Add More Commands

Update `basic.sh`:

``` bash
#!/usr/bin/env bash

echo "Hello from Bash"
date
pwd
hostname
whoami
```

Run:

``` bash
./basic.sh
```

Observe how Bash executes the commands in sequence.

------------------------------------------------------------------------

## 5. Comments

In Bash, text beginning with `#` is treated as a comment.

Example:

``` bash
#!/usr/bin/env bash

# Display the current user
whoami

# Display the hostname
hostname

pwd
```

An inline comment is also possible:

``` bash
pwd  # Display the current working directory
```

The shebang is a special case because:

``` bash
#!/usr/bin/env bash
```

is interpreted by the operating system as the interpreter directive.

------------------------------------------------------------------------

## 6. Bash Prompt and PS1

The Bash prompt is controlled by the `PS1` shell variable.

Display the current value:

``` bash
echo "$PS1"
```

A common prompt uses:

``` text
\u@\h:\w\$
```

Meaning:

  Escape   Meaning
  -------- ----------------------------------------
  `\u`     Username
  `\h`     Hostname
  `\w`     Current working directory
  `\$`     `$` for a normal user and `#` for root

Try a temporary prompt:

``` bash
PS1='\u@\h:\w\$ '
```

Open another terminal or restore the previous prompt if necessary.

### Make the change persistent

Add the following to `~/.bashrc`:

``` bash
PS1='\u@\h:\w\$ '
```

Then reload:

``` bash
source ~/.bashrc
```

> **Note:** Do not add `.` or the current directory to `PATH` just to
> make scripts executable by name.

------------------------------------------------------------------------

## 7. Run Scripts from Your Personal `bin` Directory

Create a personal command directory:

``` bash
mkdir -p "$HOME/bin"
```

Add it to `PATH` for the current shell:

``` bash
export PATH="$HOME/bin:$PATH"
```

Verify:

``` bash
echo "$PATH"
```

Create a small command:

``` bash
cat > "$HOME/bin/serverinfo" <<'EOF'
#!/usr/bin/env bash

echo "User: $USER"
echo "Hostname: $HOSTNAME"
echo "Kernel: $(uname -r)"
EOF
```

Make it executable:

``` bash
chmod +x "$HOME/bin/serverinfo"
```

Run it without `./`:

``` bash
serverinfo
```

To make the PATH change persistent, add this line to `~/.bashrc`:

``` bash
export PATH="$HOME/bin:$PATH"
```

Then:

``` bash
source ~/.bashrc
```

------------------------------------------------------------------------

# Exercise 1 --- First AWS/DevOps Script

Create:

``` text
system-info.sh
```

The script must display:

``` text
User:
Hostname:
Current Directory:
Shell:
Kernel:
```

Use commands you already learned in Days 1--7.

Example commands you may need:

``` bash
whoami
hostname
pwd
echo "$SHELL"
uname -r
```

Make the script executable:

``` bash
chmod +x system-info.sh
```

Run:

``` bash
./system-info.sh
```

### Challenge

Make the output easier to read by adding labels:

``` text
User: ec2-user
Hostname: ip-172-31-xx-xx
Current Directory: /home/ec2-user/shell-scripting
Shell: /bin/bash
Kernel: 6.x.x
```

------------------------------------------------------------------------

# Section 2 - Shell Variables

## 8. Create a Shell Variable

Create a variable:

``` bash
NAME="Joe"
```

Display it:

``` bash
echo "$NAME"
```

Remember:

``` bash
NAME="Joe"
```

is correct.

This is incorrect:

``` bash
NAME = "Joe"
```

There must be **no spaces** around `=`.

------------------------------------------------------------------------

## 9. Variable Naming Rules

Valid examples:

``` bash
KEY="value"
_VAR=5
techpro_education="test"
KEY_1="value1"
```

Invalid examples:

``` bash
3_KEY="value"
-VAR=5
techpro-education="test"
KEY_1?="value1"
```

A Bash variable name can contain:

-   letters
-   digits
-   underscores

It cannot start with a digit.

------------------------------------------------------------------------

## 10. Create a Variable Script

Create:

``` text
variable.sh
```

Add:

``` bash
#!/usr/bin/env bash

NAME="Joe"

echo "Name: $NAME"
```

Make it executable:

``` bash
chmod +x variable.sh
```

Run:

``` bash
./variable.sh
```

------------------------------------------------------------------------

## 11. Remove a Variable

Create:

``` bash
PROJECT="aws-devops"
```

Display it:

``` bash
echo "$PROJECT"
```

Remove it:

``` bash
unset PROJECT
```

Check:

``` bash
echo "$PROJECT"
```

------------------------------------------------------------------------

## 12. Environment Variables

A normal shell variable belongs to the current shell.

Example:

``` bash
APP_ENV="production"
```

Export it:

``` bash
export APP_ENV
```

Or create and export it in one command:

``` bash
export APP_ENV="production"
```

Check:

``` bash
echo "$APP_ENV"
```

Environment variables are inherited by child processes.

You can see exported environment variables with:

``` bash
env
```

or:

``` bash
printenv
```

------------------------------------------------------------------------

## 13. Shell Variable vs Environment Variable

Try:

``` bash
APP_NAME="revision-app"
bash
echo "$APP_NAME"
exit
```

Now try:

``` bash
export APP_NAME="revision-app"
bash
echo "$APP_NAME"
exit
```

Observe the difference.

### Question

Why could the child shell access the variable in the second example?

------------------------------------------------------------------------

# Section 3 - Quoting

## 14. Single Quotes

Create:

``` bash
NAME="TechPro"
```

Run:

``` bash
echo '$NAME'
```

The output is:

``` text
$NAME
```

Single quotes preserve the text literally.

------------------------------------------------------------------------

## 15. Double Quotes

Run:

``` bash
echo "$NAME"
```

The output is:

``` text
TechPro
```

Double quotes allow variable expansion.

### DevOps example

``` bash
ENVIRONMENT="production"

echo "Deploying to $ENVIRONMENT"
```

------------------------------------------------------------------------

## 16. Why Quote Variables?

Create:

``` bash
APP_NAME="AWS DevOps Application"
```

Compare:

``` bash
echo $APP_NAME
```

and:

``` bash
echo "$APP_NAME"
```

For commands that treat arguments separately, quoting helps preserve a
value containing spaces as one argument.

A useful rule for beginners:

> When expanding a variable, use double quotes unless you have a
> specific reason not to.

------------------------------------------------------------------------

# Section 4 - Command Substitution

## 17. Capture Command Output

Command substitution allows the output of a command to become a value.

Use:

``` bash
working_directory=$(pwd)
```

Display it:

``` bash
echo "$working_directory"
```

Another example:

``` bash
HOST=$(hostname)
```

Then:

``` bash
echo "Server: $HOST"
```

------------------------------------------------------------------------

## 18. Create a Command-Substitution Script

Create:

``` text
command-substitution.sh
```

Add:

``` bash
#!/usr/bin/env bash

HOST=$(hostname)
CURRENT_DIR=$(pwd)
DATE_NOW=$(date)

echo "Hostname: $HOST"
echo "Directory: $CURRENT_DIR"
echo "Date: $DATE_NOW"
```

Make it executable:

``` bash
chmod +x command-substitution.sh
```

Run:

``` bash
./command-substitution.sh
```

### Preferred syntax

Use:

``` bash
$(command)
```

for new scripts.

Older scripts may contain backticks:

``` bash
`command`
```

For this course, use `$(...)` as the preferred syntax.

------------------------------------------------------------------------

# Section 5 - Console Input with read

## 19. Basic `read`

Update `variable.sh`:

``` bash
#!/usr/bin/env bash

echo "Enter your name:"
read -r NAME

echo "Welcome $NAME"
```

Run:

``` bash
./variable.sh
```

------------------------------------------------------------------------

## 20. Use `read -p`

A cleaner approach is:

``` bash
#!/usr/bin/env bash

read -r -p "Enter your name: " NAME

echo "Welcome $NAME"
```

Run:

``` bash
./variable.sh
```

The `-p` option displays the prompt before reading input.

------------------------------------------------------------------------

## 21. Read Sensitive Input

For information that should not appear on the screen:

``` bash
read -r -s -p "Enter your password: " PASSWORD
echo
echo "Password received."
```

The `-s` option prevents the typed characters from being displayed.

> Do not print real passwords to the terminal or store them in plain
> text.

------------------------------------------------------------------------

# Section 6 - Positional Parameters & Special Parameters

## 22. Create a Parameter Script

Create:

``` text
argument.sh
```

Add:

``` bash
#!/usr/bin/env bash

echo "Script name: $0"
echo "First parameter: $1"
echo "Second parameter: $2"
echo "Third parameter: $3"
echo "Parameter count: $#"
```

Make it executable:

``` bash
chmod +x argument.sh
```

Run:

``` bash
./argument.sh AWS DevOps Linux
```

Observe the values of:

``` text
$0
$1
$2
$3
$#
```

------------------------------------------------------------------------

## 23. `$@` --- All Positional Arguments

Update the script:

``` bash
#!/usr/bin/env bash

echo "Script name: $0"
echo "Parameter count: $#"

printf 'Argument: %s\n' "$@"
```

Run:

``` bash
./argument.sh "AWS DevOps" Linux Kubernetes
```

Notice that:

``` bash
"$@"
```

preserves argument boundaries.

This is preferred over:

``` bash
$@
```

when passing arguments to commands.

------------------------------------------------------------------------

## 24. Special Parameters

Try:

``` bash
echo "Current shell PID: $$"
```

Run a successful command:

``` bash
ls
echo "$?"
```

Run a command that does not exist:

``` bash
command-that-does-not-exist
echo "$?"
```

Remember:

``` text
$?  → exit status of the previous command
$$  → PID of the current shell/script
```

A successful command normally returns:

``` text
0
```

A non-zero value indicates that the command did not succeed.

> Do not assume that every failure returns `1`.

------------------------------------------------------------------------

## 25. Arguments Beyond `$9`

For the tenth positional parameter, use braces:

``` bash
${10}
```

not:

``` bash
$10
```

Create a test script if needed and run it with at least ten arguments.

------------------------------------------------------------------------

# Exercise 2 --- Deployment Calculator

Create:

``` text
deployment-cost.sh
```

Run it like this:

``` bash
./deployment-cost.sh 5 20
```

Interpret:

``` text
$1 = number of servers
$2 = cost per server
```

Calculate:

``` text
total = servers × cost
```

Expected output:

``` text
Servers: 5
Cost per server: 20
Total: 100
```

### Requirements

Use:

-   `$1`
-   `$2`
-   Bash arithmetic
-   variables
-   `echo`

Do not hard-code the final total.

### Test with different values

``` bash
./deployment-cost.sh 3 50
./deployment-cost.sh 10 25
./deployment-cost.sh 8 100
```

------------------------------------------------------------------------

# Section 7 - Basic Arithmetic Operations

## 28. Arithmetic with `expr`

`expr` can perform arithmetic:

``` bash
expr 3 + 5
expr 6 - 2
expr 7 \* 3
expr 9 / 3
expr 7 % 2
```

Notice that `*` needs escaping when used with `expr`.

### Teaching note

`expr` is still available, but for new Bash scripts we will prefer
Bash's native arithmetic syntax.

------------------------------------------------------------------------

## 29. Arithmetic with `let`

`let` is a Bash built-in.

Example:

``` bash
let "sum = 3 + 5"
echo "$sum"
```

Another form:

``` bash
let sub=8-4
echo "$sub"
```

Increment:

``` bash
let x++
```

Decrement:

``` bash
let y--
```

### Recommendation

`let` is useful to recognize in existing scripts, but prefer:

``` bash
$((...))
```

and:

``` bash
((...))
```

for new scripts.

------------------------------------------------------------------------

## 30. Arithmetic Expansion --- `$(( ))`

The preferred way to obtain an arithmetic result is:

``` bash
sum=$((3 + 5))
echo "$sum"
```

Without spaces:

``` bash
sum=$((3+5))
echo "$sum"
```

Both work.

------------------------------------------------------------------------

## 31. Arithmetic Operators

Practice:

``` bash
echo $((10 + 5))
echo $((10 - 5))
echo $((10 * 5))
echo $((10 / 5))
echo $((10 % 3))
```

The `%` operator returns the remainder.

For example:

``` bash
echo $((10 % 3))
```

returns:

``` text
1
```

### Integer Division

Bash arithmetic uses integers:

``` bash
echo $((10 / 3))
```

returns:

``` text
3
```

It does not return `3.33`.

------------------------------------------------------------------------

## 32. Increment and Decrement

Using arithmetic commands:

``` bash
number=10

((number++))

echo "$number"
```

Decrement:

``` bash
((number--))

echo "$number"
```

You can also use:

``` bash
((++number))
((--number))
```

The position of `++` or `--` matters when the value is used in the same
expression.

------------------------------------------------------------------------

## 33. `$(( ))` vs `(( ))`

Use:

``` bash
result=$((10 + 5))
```

when you need the arithmetic result.

Use:

``` bash
((number++))
```

when you want Bash to evaluate an arithmetic expression as a command.

Both are native Bash features.

------------------------------------------------------------------------

# Section 8 - Calculator Script

## 34. Create `calculator.sh`

Create:

``` text
calculator.sh
```

Use:

``` bash
#!/usr/bin/env bash

read -r -p "Input first number: " first_number
read -r -p "Input second number: " second_number

sum=$((first_number + second_number))
sub=$((first_number - second_number))
mul=$((first_number * second_number))
div=$((first_number / second_number))

echo "SUM=$sum"
echo "SUB=$sub"
echo "MUL=$mul"
echo "DIV=$div"
```

Make it executable:

``` bash
chmod +x calculator.sh
```

Run:

``` bash
./calculator.sh
```

Test with several pairs of numbers.

### Challenge

Add:

``` text
remainder
```

using `%`.

------------------------------------------------------------------------

# Exercise 3 --- System Report

Create:

``` text
system-report.sh
```

The script must:

1.  Ask for the student's name.
2.  Capture the hostname using `$(hostname)`.
3.  Ask for the number of servers.
4.  Calculate double the number of servers.
5.  Display a formatted report.

Example:

``` text
Student: Ali
Hostname: ip-172-31-xx-xx
Servers: 5
Scaled Servers: 10
```

### Requirements

Use:

``` text
read -r -p
variables
$(hostname)
$((...))
```

Do not hard-code the hostname or calculated server count.

------------------------------------------------------------------------

# Section 9 - Bash Debugging

## 35. Run a Script with Debugging

Create:

``` text
debug-demo.sh
```

Add:

``` bash
#!/usr/bin/env bash

NAME="AWS-DevOps"
HOST=$(hostname)

echo "Course: $NAME"
echo "Host: $HOST"
```

Run normally:

``` bash
./debug-demo.sh
```

Then run:

``` bash
bash -x debug-demo.sh
```

Observe how Bash displays the commands while executing them.

------------------------------------------------------------------------

## 36. Debug from Inside the Script

You can temporarily enable tracing:

``` bash
set -x

echo "Starting deployment check"
HOST=$(hostname)
echo "Host: $HOST"

set +x

echo "Debugging section finished"
```

Use:

``` bash
set -x
```

to start tracing and:

``` bash
set +x
```

to stop tracing.

This is useful when troubleshooting automation and deployment scripts.

------------------------------------------------------------------------

# Final Practical Challenge --- Mini DevOps Script

## Task 37 --- Server Deployment Information

Create:

``` text
deploy-check.sh
```

The script must:

1.  Display the current user.
2.  Capture the hostname.
3.  Capture the current directory.
4.  Ask the user for the number of servers.
5.  Calculate twice that number.
6.  Display a deployment summary.
7.  Display the exit status of the last successful command.

Example output:

``` text
Deployment Check
----------------
User: ec2-user
Hostname: ip-172-31-xx-xx
Directory: /home/ec2-user/shell-scripting
Requested servers: 5
Planned instances: 10
Deployment check completed.
```

### Requirements

Use at least:

-   Shebang
-   Comments
-   Variables
-   `read -r -p`
-   Command substitution
-   Arithmetic expansion
-   `echo`
-   Quoted variables
-   `$?`

------------------------------------------------------------------------

# Review Questions

Answer these questions after completing the hands-on.

### Question 1

What is the difference between a shell and Bash?

### Question 2

What does this line do?

``` bash
#!/usr/bin/env bash
```

### Question 3

Why do we normally use:

``` bash
./script.sh
```

instead of:

``` bash
script.sh
```

### Question 4

What is the purpose of:

``` bash
chmod +x script.sh
```

### Question 5

What is the difference between:

``` bash
echo '$NAME'
```

and:

``` bash
echo "$NAME"
```

### Question 6

What does this do?

``` bash
HOST=$(hostname)
```

### Question 7

What is the difference between:

``` text
$0
$1
$2
$#
$@
$?
$$
```

### Question 8

Why is this preferred:

``` bash
"$@"
```

over:

``` bash
$@
```

when passing arguments?

### Question 9

What is the difference between:

``` bash
result=$((10 + 5))
```

and:

``` bash
((number++))
```

### Question 10

What does exit status `0` normally indicate?

### Question 11

Does every failed command return exit status `1`?

### Question 12

Why is `$(command)` preferred over backticks in new Bash scripts?

------------------------------------------------------------------------

# Submission Requirements

Submit **one Markdown or text file** containing:

1.  Your name
2.  EC2/Linux distribution used
3.  Commands/scripts created
4.  Important outputs
5.  Answers to the review questions
6.  Any errors encountered
7.  How you solved those errors

Screenshots are optional unless requested by the instructor.

------------------------------------------------------------------------

# Completion Checklist

-   [ ] Created and executed a Bash script
-   [ ] Used a shebang
-   [ ] Used comments
-   [ ] Customized `PS1`
-   [ ] Added a personal `bin` directory to `PATH`
-   [ ] Created shell variables
-   [ ] Used `export` and `unset`
-   [ ] Practiced single and double quotes
-   [ ] Used `read -r -p`
-   [ ] Used command substitution
-   [ ] Used positional parameters
-   [ ] Used `$@`, `$#`, `$?`, and `$$`
-   [ ] Practiced Bash arrays
-   [ ] Practiced arithmetic
-   [ ] Used `$(( ))`
-   [ ] Used `(( ))`
-   [ ] Practiced `expr` and `let` as alternative/legacy syntax
-   [ ] Used `bash -x`
-   [ ] Completed Exercise 1
-   [ ] Completed Exercise 2
-   [ ] Completed Exercise 3
-   [ ] Completed the final DevOps script
-   [ ] Answered the review questions

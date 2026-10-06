# Hands-on Linux-10: Shell Scripting Loops

## Purpose

The purpose of this hands-on training is to teach students how to use
Bash loops to repeat tasks and build simple Linux/AWS-DevOps automation.

This lesson builds directly on Day 9:

-   Day 9: conditions make decisions.
-   Day 10: loops repeat actions.

A useful way to remember the difference:

``` text
if    → make a decision
loop  → repeat an action
```

## Learning Outcomes

By the end of this hands-on training, students will be able to:

-   Explain why loops are used in shell scripts.
-   Use `while` loops.
-   Use `until` loops.
-   Use `for` loops.
-   Loop through lists, files, and directories.
-   Use Bash arrays with loops.
-   Read a file line by line with `while read`.
-   Use `break` and `continue`.
-   Identify and stop an infinite loop.
-   Build simple Linux/AWS-DevOps monitoring and checking scripts.
-   Combine Day 9 conditions with Day 10 loops.

> **Course note:** This lesson focuses on `while`, `until`, and `for`.


------------------------------------------------------------------------

# Section 1 - Why Do We Need Loops?

Sometimes running a command once is not enough.

For example, imagine that we need to check 10 servers.

Without a loop, we might repeat the same command 10 times.

With a loop, we can automate the repetition.

A loop normally has:

``` text
Start
  ↓
Check condition / get next item
  ↓
Run commands
  ↓
Repeat
  ↓
Stop
```

------------------------------------------------------------------------

## 1. Day 9 → Day 10 Connection

On Day 9 we learned:

``` bash
if [[ $number -gt 50 ]]; then
    echo "Greater than 50"
fi
```

This makes a decision once.

Today we can repeat an action:

``` bash
number=1

while [[ $number -le 5 ]]; do
    echo "Number: $number"
    ((number++))
done
```

This checks the condition repeatedly.

------------------------------------------------------------------------

# Section 2 - While Loops

## 2. Basic `while` Syntax

The basic structure is:

``` bash
while [[ condition ]]; do
    commands
done
```

Meaning:

``` text
while the condition is true
    execute the commands
repeat
```

The important parts are:

``` text
while      → starts the loop
[[ ... ]]  → condition
do         → starts the loop body
done       → ends the loop
```

------------------------------------------------------------------------

## 3. Create the Working Directory

Create a directory:

``` bash
mkdir -p "$HOME/loops"
cd "$HOME/loops"
```

Verify:

``` bash
pwd
```

------------------------------------------------------------------------

## 4. Print Numbers with `while`

Create:

``` text
while-loop.sh
```

Add:

``` bash
#!/usr/bin/env bash

number=1

while [[ $number -le 10 ]]; do
    echo "$number"
    ((number++))
done

echo "Now, number is $number"
```

Make it executable:

``` bash
chmod +x while-loop.sh
```

Run:

``` bash
./while-loop.sh
```

Expected output:

``` text
1
2
3
4
5
6
7
8
9
10
Now, number is 11
```

### Why does the final value become 11?

The loop prints `10`, then:

``` bash
((number++))
```

changes the value to `11`.

The condition is then checked again:

``` bash
[[ $number -le 10 ]]
```

which is false.

The loop stops.

------------------------------------------------------------------------

## 5. Infinite Loops

Be careful when using `while`.

This script is dangerous:

``` bash
#!/usr/bin/env bash

number=1

while [[ $number -le 5 ]]; do
    echo "$number"
done
```

### What is wrong?

`number` never changes.

Therefore:

``` bash
[[ $number -le 5 ]]
```

always remains true.

The loop never ends.

This is called an **infinite loop**.

### How can you stop it?

Press:

``` text
Ctrl + C
```

This sends an interrupt signal to the running process.

### Correct version

``` bash
number=1

while [[ $number -le 5 ]]; do
    echo "$number"
    ((number++))
done
```

> **Important:** Every loop needs a clear path toward its stopping
> condition.

------------------------------------------------------------------------

# Section 3 - Until Loops

## 6. Basic `until` Syntax

An `until` loop is similar to `while`, but the logic is reversed.

``` bash
until [[ condition ]]; do
    commands
done
```

An `until` loop continues while the condition is **false**.

It stops when the condition becomes **true**.

------------------------------------------------------------------------

## 7. `while` vs `until`

Compare:

``` bash
number=1

while [[ $number -le 5 ]]; do
    echo "$number"
    ((number++))
done
```

and:

``` bash
number=1

until [[ $number -gt 5 ]]; do
    echo "$number"
    ((number++))
done
```

Both produce:

``` text
1
2
3
4
5
```

Remember:

  Loop      Continues when       Stops when
  --------- -------------------- -------------------------
  `while`   condition is true    condition becomes false
  `until`   condition is false   condition becomes true

------------------------------------------------------------------------

## 8. Create an `until` Script

Create:

``` text
until-loop.sh
```

Add:

``` bash
#!/usr/bin/env bash

number=1

until [[ $number -gt 10 ]]; do
    echo "$number"
    ((number++))
done

echo "Now, number is $number"
```

Make it executable:

``` bash
chmod +x until-loop.sh
```

Run:

``` bash
./until-loop.sh
```

------------------------------------------------------------------------

# Section 4 - For Loops

## 9. Basic `for` Loop

A `for` loop is useful when you want to process each item in a list.

Create:

``` text
for-loop.sh
```

Add:

``` bash
#!/usr/bin/env bash

for number in 0 1 2 3 4 5; do
    echo "Number: $number"
done
```

Run:

``` bash
chmod +x for-loop.sh
./for-loop.sh
```

------------------------------------------------------------------------

## 10. Loop Through a List of Words

``` bash
for environment in development staging production; do
    echo "Environment: $environment"
done
```

Output:

``` text
Environment: development
Environment: staging
Environment: production
```

This is a useful DevOps pattern because environments are often processed
as a list.

------------------------------------------------------------------------

## 11. Loop Through a Range

Bash supports brace expansion:

``` bash
for number in {1..5}; do
    echo "$number"
done
```

You can also use a step:

``` bash
for number in {0..10..2}; do
    echo "$number"
done
```

Output:

``` text
0
2
4
6
8
10
```

> Brace expansion creates the list before the loop runs.

------------------------------------------------------------------------

## 12. Loop Through Files

A `for` loop can process files.

Create some test files:

``` bash
touch app.log error.log access.log
```

Then:

``` bash
for file in *.log; do
    echo "Log file: $file"
done
```

The pattern:

``` bash
*.log
```

matches filenames ending in `.log`.

### Safer version

If there may be no matching files:

``` bash
for file in *.log; do
    if [[ -f "$file" ]]; then
        echo "Log file: $file"
    fi
done
```

This combines Day 9 `if` conditions with Day 10 loops.

------------------------------------------------------------------------

## 13. Loop Through Directories

Example:

``` bash
for item in /etc/*; do
    echo "$item"
done
```

You can also check whether each item is a directory:

``` bash
for item in /etc/*; do
    if [[ -d "$item" ]]; then
        echo "Directory: $item"
    fi
done
```

------------------------------------------------------------------------

## 14. C-Style `for` Loop

Bash also supports a C-style loop:

``` bash
for ((i=1; i<=5; i++)); do
    echo "Number: $i"
done
```

This is useful when working with counters.

The three parts are:

``` text
i=1       → initial value
i<=5      → condition
i++       → increment
```

This uses the arithmetic syntax from earlier lessons.

------------------------------------------------------------------------

# Section 5 - Arrays + Loops

## 15. Create an Array

An array stores multiple values in one variable.

Example:

``` bash
devops_tools=("docker" "kubernetes" "ansible" "terraform" "jenkins")
```

Display the first element:

``` bash
echo "${devops_tools[0]}"
```

Display all elements:

``` bash
echo "${devops_tools[@]}"
```

Count the elements:

``` bash
echo "${#devops_tools[@]}"
```

------------------------------------------------------------------------

## 16. Loop Through an Array

Use:

``` bash
for tool in "${devops_tools[@]}"; do
    echo "Tool: $tool"
done
```

### Important

Prefer:

``` bash
"${devops_tools[@]}"
```

rather than:

``` bash
${devops_tools[@]}
```

Quoting preserves each array element as a separate item.

------------------------------------------------------------------------

## 17. Read Multiple Values into an Array

Bash `read` can store multiple input values in an array using `-a`.

Example:

``` bash
read -r -a environments
```

Ask the user for input:

``` bash
read -r -p "Enter environments: " -a environments
```

Then loop through them:

``` bash
for environment in "${environments[@]}"; do
    echo "Environment: $environment"
done
```

Try:

``` text
development staging production
```

------------------------------------------------------------------------

# Section 6 - `while read` for Linux Files

## 18. Read a File One Line at a Time

Create a file:

``` bash
cat > servers.txt <<'EOF'
server01
server02
server03
server04
EOF
```

Read it line by line:

``` bash
while read -r line; do
    echo "Server: $line"
done < servers.txt
```

The important structure is:

``` bash
while read -r line; do
    commands
done < file
```

This is a very useful Linux automation pattern.

It can be used for:

-   Server lists
-   User lists
-   Configuration files
-   Log processing
-   Inventory files

------------------------------------------------------------------------

# Section 7 - Break and Continue

## 19. `break`

The `break` statement immediately exits the current loop.

Example:

``` bash
for number in {1..10}; do
    echo "$number"

    if [[ $number -eq 5 ]]; then
        break
    fi
done
```

Output:

``` text
1
2
3
4
5
```

Remember:

``` text
break → leave the loop
```

------------------------------------------------------------------------

## 20. `continue`

The `continue` statement skips the rest of the current iteration and
starts the next iteration.

Example:

``` bash
for number in {1..5}; do

    if [[ $number -eq 3 ]]; then
        continue
    fi

    echo "$number"
done
```

Output:

``` text
1
2
4
5
```

Remember:

``` text
continue → skip this iteration
```

------------------------------------------------------------------------

## 21. Break vs Continue

  Command      What it does
  ------------ -------------------------------
  `break`      Exits the entire current loop
  `continue`   Skips the current iteration

Think of it this way:

``` text
break
  ↓
LEAVE THE LOOP


continue
  ↓
SKIP THIS ITEM
  ↓
GO TO NEXT ITEM
```

------------------------------------------------------------------------

# Section 8 - AWS/DevOps Loop Examples

## 22. Check Multiple Services

Create:

``` text
service-check.sh
```

Add:

``` bash
#!/usr/bin/env bash

services=("sshd" "docker")

for service in "${services[@]}"; do

    if systemctl is-active --quiet "$service"; then
        echo "$service: RUNNING"
    else
        echo "$service: NOT RUNNING"
    fi

done
```

Make it executable:

``` bash
chmod +x service-check.sh
```

Run:

``` bash
./service-check.sh
```

> Service names can differ between Linux distributions. If a service
> does not exist on your system, check with:
>
> ``` bash
> systemctl list-units --type=service
> ```

------------------------------------------------------------------------

## 23. Check Log Files

Create some test logs:

``` bash
mkdir -p logs

printf 'INFO\nINFO\nERROR\n' > logs/app.log
printf 'INFO\nWARNING\nINFO\nINFO\n' > logs/system.log
printf 'ERROR\nERROR\n' > logs/error.log
```

Now process them:

``` bash
for file in logs/*.log; do

    if [[ -f "$file" ]]; then
        echo "Checking: $file"
        wc -l "$file"
    fi

done
```

This combines:

``` text
for
if
-f
variables
wc
file globbing
```

------------------------------------------------------------------------

## 24. Nested Loops

Nested loops are possible when one loop needs to run inside another.

Example:

``` bash
for environment in development staging production; do

    for region in us-east-1 eu-west-1; do
        echo "$environment -> $region"
    done

done
```

Output:

``` text
development -> us-east-1
development -> eu-west-1
staging -> us-east-1
staging -> eu-west-1
production -> us-east-1
production -> eu-west-1
```

Use nested loops carefully because they can make scripts harder to read.

------------------------------------------------------------------------

# Exercise 1 - Loop and Arithmetic

## Task

Create:

``` text
sum.sh
```

Calculate the sum of numbers from `1` to `100` using a `while` loop.

Expected result:

``` text
Sum from 1 to 100: 5050
```

### Requirements

Use:

-   `while`
-   a counter variable
-   arithmetic expansion or `((...))`
-   `echo`

### Challenge

Allow the user to specify the maximum number using `$1`.

Run:

``` bash
./sum.sh 100
```

Expected:

``` text
Sum from 1 to 100: 5050
```

Try:

``` bash
./sum.sh 10
./sum.sh 500
```

------------------------------------------------------------------------

# Exercise 2 - Environments Array

## Task

Create:

``` text
environment-loop.sh
```

Ask the user to enter several environments:

``` bash
read -r -p "Enter environments: " -a environments
```

Loop through the array and display:

``` text
Checking environment: development
Checking environment: staging
Checking environment: production
```

### Challenge

Use Day 9 conditional logic inside the loop.

If the environment is `production`:

``` text
WARNING: Production environment
```

For other environments:

``` text
Environment: development
Environment: staging
```

### Example

Input:

``` text
development staging production
```

Output:

``` text
Environment: development
Environment: staging
WARNING: Production environment
```

------------------------------------------------------------------------

# Exercise 3 - DevOps Service Monitor

## Task

Create:

``` text
service-monitor.sh
```

Create an array containing several services relevant to your Linux
system.

Example:

``` bash
services=("sshd" "docker")
```

Loop through the array.

For every service:

-   Check whether it is active.
-   Print `RUNNING` when active.
-   Print `NOT RUNNING` when inactive or unavailable.

Expected style:

``` text
Service Monitor
---------------

sshd: RUNNING
docker: NOT RUNNING
```

### Challenge

Keep counters:

``` text
Passed: 1
Failed: 1
```

At the end, display:

``` text
Service Check Complete
Passed: 1
Failed: 1
```

------------------------------------------------------------------------

# Final Mini Project - Server Health Monitor

## Task

Create:

``` text
server-health.sh
```

Build a small Linux server health-monitoring script that combines Days
8, 9, and 10.

### Requirements

The script must:

1.  Store several services in an array.
2.  Loop through the services.
3.  Check whether each service is running.
4.  Print `RUNNING` or `NOT RUNNING`.
5.  Check whether `/var/log` exists.
6.  Check whether `/var/log` is readable.
7.  Check whether `curl` is installed.
8.  Count passed checks.
9.  Count failed checks.
10. Print a final health status.
11. Return exit status `0` when all checks pass.
12. Return a non-zero exit status when one or more checks fail.

### Suggested structure

``` bash
#!/usr/bin/env bash

services=("sshd" "docker")

passed=0
failed=0

echo "================================"
echo "       SERVER HEALTH CHECK"
echo "================================"
echo

echo "Service Checks:"

for service in "${services[@]}"; do

    if systemctl is-active --quiet "$service"; then
        echo "$service: RUNNING"
        ((passed++))
    else
        echo "$service: NOT RUNNING"
        ((failed++))
    fi

done

echo
echo "Filesystem Checks:"

if [[ -d "/var/log" ]]; then
    echo "/var/log: OK"
    ((passed++))
else
    echo "/var/log: FAILED"
    ((failed++))
fi

if [[ -r "/var/log" ]]; then
    echo "/var/log readable: OK"
    ((passed++))
else
    echo "/var/log readable: FAILED"
    ((failed++))
fi

echo
echo "Command Checks:"

if command -v curl > /dev/null 2>&1; then
    echo "curl: INSTALLED"
    ((passed++))
else
    echo "curl: NOT INSTALLED"
    ((failed++))
fi

echo
echo "--------------------------------"
echo "Passed: $passed"
echo "Failed: $failed"
echo "--------------------------------"

if [[ $failed -eq 0 ]]; then
    echo "Health Status: PASSED"
    exit 0
else
    echo "Health Status: WARNING"
    exit 1
fi
```

### Important

The example above is a starting point. Students should understand and be
able to explain each section rather than simply copy it.

### Expected output style

``` text
================================
       SERVER HEALTH CHECK
================================

Service Checks:
sshd: RUNNING
docker: NOT RUNNING

Filesystem Checks:
/var/log: OK
/var/log readable: OK

Command Checks:
curl: INSTALLED

--------------------------------
Passed: 4
Failed: 1
--------------------------------
Health Status: WARNING
```

------------------------------------------------------------------------

# Review Questions

## Question 1

Why do we use loops in Bash?

## Question 2

What is the difference between:

``` bash
if
```

and:

``` bash
while
```

## Question 3

What is the difference between:

``` bash
while [[ condition ]]; do
```

and:

``` bash
until [[ condition ]]; do
```

## Question 4

What happens if the condition of a `while` loop never becomes false?

## Question 5

How can you stop an accidentally running infinite loop from the
terminal?

## Question 6

What is the difference between:

``` bash
break
```

and:

``` bash
continue
```

## Question 7

What does this do?

``` bash
for file in *.log; do
    echo "$file"
done
```

## Question 8

What does this mean?

``` bash
"${devops_tools[@]}"
```

## Question 9

What does `read -r -a environments` do?

## Question 10

What does this do?

``` bash
while read -r line; do
    echo "$line"
done < servers.txt
```

## Question 11

Why do we quote an array expansion like:

``` bash
"${array[@]}"
```

## Question 12

Why is the following useful in DevOps?

``` bash
for service in "${services[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "$service: RUNNING"
    fi
done
```

------------------------------------------------------------------------

# Instructor Solutions

> Complete the exercises before opening this section.

## Solution 1 - Sum Script

``` bash
#!/usr/bin/env bash

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 <maximum>"
    exit 1
fi

maximum="$1"

if [[ ! "$maximum" =~ ^[0-9]+$ ]]; then
    echo "Error: maximum must be a non-negative integer."
    exit 1
fi

number=1
sum=0

while [[ $number -le $maximum ]]; do
    ((sum += number))
    ((number++))
done

echo "Sum from 1 to $maximum: $sum"
```

Run:

``` bash
chmod +x sum.sh
./sum.sh 100
```

------------------------------------------------------------------------

## Solution 2 - Environment Loop

``` bash
#!/usr/bin/env bash

read -r -p "Enter environments: " -a environments

for environment in "${environments[@]}"; do

    if [[ "$environment" == "production" ]]; then
        echo "WARNING: Production environment"
    else
        echo "Environment: $environment"
    fi

done
```

------------------------------------------------------------------------

## Solution 3 - Service Monitor

``` bash
#!/usr/bin/env bash

services=("sshd" "docker")

passed=0
failed=0

echo "Service Monitor"
echo "---------------"
echo

for service in "${services[@]}"; do

    if systemctl is-active --quiet "$service"; then
        echo "$service: RUNNING"
        ((passed++))
    else
        echo "$service: NOT RUNNING"
        ((failed++))
    fi

done

echo
echo "Service Check Complete"
echo "Passed: $passed"
echo "Failed: $failed"
```

------------------------------------------------------------------------

# Completion Checklist

-   [ ] Reviewed the Day 9 → Day 10 connection
-   [ ] Created a `while` loop
-   [ ] Practiced loop conditions with `[[ ]]`
-   [ ] Understand how infinite loops happen
-   [ ] Practiced `Ctrl + C`
-   [ ] Created an `until` loop
-   [ ] Compared `while` and `until`
-   [ ] Created a basic `for` loop
-   [ ] Looped through a word list
-   [ ] Looped through a numeric range
-   [ ] Looped through files
-   [ ] Looped through directories
-   [ ] Practiced a C-style `for` loop
-   [ ] Created a Bash array
-   [ ] Looped through an array
-   [ ] Practiced `read -r -a`
-   [ ] Read a file with `while read -r`
-   [ ] Practiced `break`
-   [ ] Practiced `continue`
-   [ ] Checked multiple Linux services with a loop
-   [ ] Processed log files with a loop
-   [ ] Completed Exercise 1
-   [ ] Completed Exercise 2
-   [ ] Completed Exercise 3
-   [ ] Completed the Server Health Monitor project
-   [ ] Answered the review questions
-   [ ] Can explain the difference between `break` and `continue`
-   [ ] Can explain the difference between `while` and `until`
-   [ ] Can combine loops with Day 9 conditions

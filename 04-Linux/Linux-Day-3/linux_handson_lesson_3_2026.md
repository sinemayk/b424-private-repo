# Hands-on Linux-03: Editing, Reading, Finding, and Searching Files

## Goal

The goal of this hands-on training is to help students work efficiently with text files and logs on Linux systems used in AWS and DevOps environments.

---

## Learning Outcomes

By the end of this hands-on session, students will be able to:

- Edit files with Vim and Nano.
- Understand the essential Vim modes.
- Read the beginning and end of files with `head` and `tail`.
- Follow a growing log file with `tail -f`.
- Read small files with `cat` and large files with `less`.
- Search the filesystem with `find`.
- Search text with `grep`.
- Use common `grep` options such as `-i`, `-n`, `-v`, `-c`, `-w`, `-A`, `-B`, and `-C`.
- Combine commands with pipes for practical log investigation.

---
**PS1="$\e[1;32m$\u@\h:$\e[1;34m$\w$\e[m$\$"**
# Part 1 - Prepare the Lab Workspace

Go to your home directory:

```bash
cd ~
```

Create the Day 3 workspace:

```bash
mkdir -p ~/linux-day3/{config,logs,docs}
cd ~/linux-day3
```

Verify:

```bash
pwd
ls -la
```

---

# Part 2 - Check Which Editors Are Available

Check Vim or Vi:

```bash
command -v vim
command -v vi
```

Check Nano:

```bash
command -v nano
```

A minimal Linux server may not have every editor installed.

Use whichever editor exists in your environment, but practice the essential Vim commands even if Nano is your preferred editor.

---

# Part 3 - Edit a File with Vim

Open a configuration file:

```bash
vim ~/linux-day3/config/app.conf
```

## Essential Vim Workflow

### 1. Enter Insert Mode

Press:

```text
i
```

Type:

```text
APP_NAME=techpro-demo
APP_ENV=dev
PORT=8080
DEBUG=false
```

### 2. Return to Normal Mode

Press:

```text
Esc
```

### 3. Save the File

Type:

```text
:w
```

Press `Enter`.

### 4. Save and Exit

Type:

```text
:wq
```

Press `Enter`.

---

## Other Essential Vim Commands

| Command | Meaning |
|---|---|
| `i` | Enter Insert mode |
| `Esc` | Return to Normal mode |
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |
| `/word` | Search forward |
| `n` | Next search result |
| `dd` | Delete current line |
| `yy` | Copy current line |
| `p` | Paste |
| `u` | Undo |
| `Ctrl+r` | Redo |

Verify the file:

```bash
cat ~/linux-day3/config/app.conf
```

---

# Part 4 - Edit a File with Nano

If Nano is available, open:

```bash
nano ~/linux-day3/docs/notes.txt
```

Type:

```text
Day 3 Linux Notes
Vim and Nano are command-line text editors.
Linux administrators often edit configuration files remotely.
```

Useful Nano shortcuts:

```text
Ctrl+O    Save / write file
Ctrl+X    Exit
Ctrl+W    Search
Ctrl+K    Cut current line
Ctrl+U    Paste / uncut
Alt+U     Undo
Alt+E     Redo
```

Save and exit.

Verify:

```bash
cat ~/linux-day3/docs/notes.txt
```

---

# Part 5 - Create a Practice Log

Create a log file with multiple entries:

```bash
cat > ~/linux-day3/logs/app.log <<'EOF'
2026-09-20 09:00:01 INFO Application starting
2026-09-20 09:00:02 INFO Loading configuration
2026-09-20 09:00:03 INFO Connecting to database
2026-09-20 09:00:04 WARN Database response is slow
2026-09-20 09:00:05 INFO Retrying database connection
2026-09-20 09:00:06 ERROR Database connection failed
2026-09-20 09:00:07 INFO Waiting before retry
2026-09-20 09:00:08 INFO Connecting to database
2026-09-20 09:00:09 INFO Database connection successful
2026-09-20 09:00:10 INFO Starting web server
2026-09-20 09:00:11 INFO Listening on port 8080
2026-09-20 09:00:12 WARN Memory usage reached 70 percent
2026-09-20 09:00:13 INFO Health check passed
2026-09-20 09:00:14 INFO Request GET /health 200
2026-09-20 09:00:15 INFO Application ready
EOF
```

Check the file:

```bash
wc -l ~/linux-day3/logs/app.log
```

`wc` will be covered in more detail later; here we only use it to confirm that the file contains multiple lines.

---

# Part 6 - Read the Beginning with `head`

Display the first 10 lines:

```bash
head ~/linux-day3/logs/app.log
```

Display only the first 5 lines:

```bash
head -n 5 ~/linux-day3/logs/app.log
```

Display only the first 3 lines:

```bash
head -n 3 ~/linux-day3/logs/app.log
```

### Question

Which command would you use to quickly inspect the first 20 lines of a configuration export?

---

# Part 7 - Read the End with `tail`

Display the last 10 lines:

```bash
tail ~/linux-day3/logs/app.log
```

Display the last 5 lines:

```bash
tail -n 5 ~/linux-day3/logs/app.log
```

Display the last 3 lines:

```bash
tail -n 3 ~/linux-day3/logs/app.log
```

For traditional log files, the newest events are usually at the end of the file.

---

# Part 8 - Follow a Live Log with `tail -f`

This exercise works best with two terminal windows or two SSH sessions.

## Terminal 1

Run:

```bash
tail -f ~/linux-day3/logs/app.log
```

The command remains running and waits for new lines.

## Terminal 2

Append a new event:

```bash
echo "2026-09-20 09:01:00 INFO New deployment started" >> ~/linux-day3/logs/app.log
```

Append another event:

```bash
echo "2026-09-20 09:01:05 ERROR Deployment health check failed" >> ~/linux-day3/logs/app.log
```

Look at Terminal 1. The new log entries should appear immediately.

Stop `tail -f` with:

```text
Ctrl+C
```

This is one of the most useful basic techniques for watching a traditional application log in real time.

---

# Part 9 - Read Small Files with `cat`

Display the configuration file:

```bash
cat ~/linux-day3/config/app.conf
```

Display two files one after another:

```bash
cat ~/linux-day3/config/app.conf ~/linux-day3/docs/notes.txt
```

Create two small files:

```bash
echo "server=web01" > ~/linux-day3/config/server.conf
echo "region=us-east-1" > ~/linux-day3/config/aws.conf
```

Combine them:

```bash
cat ~/linux-day3/config/server.conf ~/linux-day3/config/aws.conf > ~/linux-day3/config/combined.conf
```

Verify:

```bash
cat ~/linux-day3/config/combined.conf
```

Use `cat` for quick inspection of relatively small files.

---

# Part 10 - Navigate Large Files with `less`

Open the log:

```bash
less ~/linux-day3/logs/app.log
```

Practice these keys:

```text
Up / Down    Move one line
Space        Move one screen forward
b            Move one screen backward
G            Go to end of file
g            Go to beginning
/ERROR       Search forward for ERROR
n            Go to next match
q            Quit
```

Try another file:

```bash
less /etc/passwd
```

---

# Part 11 - Use `less` with Command Output

Display process output through `less`:

```bash
ps aux | less
```

List files and inspect the result interactively:

```bash
find ~/linux-day3 -type f | less
```

You can use a pipe when a command produces more output than is comfortable to read directly in the terminal.

---

# Part 12 - Optional Utilities: `more` and `tac`

Try `more`:

```bash
more ~/linux-day3/logs/app.log
```

For interactive navigation and search, `less` is generally more capable.

Try `tac`:

```bash
tac ~/linux-day3/logs/app.log
```

`tac` prints the lines in reverse order.

> `tac` is a GNU utility and may not be available on every Unix-like system.

---

# Part 13 - Find Files by Name

Return to the lab directory:

```bash
cd ~/linux-day3
```

Find `app.conf`:

```bash
find . -name "app.conf"
```

Find all `.conf` files:

```bash
find . -name "*.conf"
```

The quotes around `*.conf` are important because they prevent the shell from expanding the wildcard before `find` evaluates it.

---

# Part 14 - Case-Insensitive File Search

Create:

```bash
touch ~/linux-day3/docs/README.md
```

Try a case-sensitive search:

```bash
find ~/linux-day3 -name "readme.md"
```

Now try:

```bash
find ~/linux-day3 -iname "readme.md"
```

`-iname` performs a case-insensitive filename match.

---

# Part 15 - Find by Type

Find regular files:

```bash
find ~/linux-day3 -type f
```

Find directories:

```bash
find ~/linux-day3 -type d
```

Find only log files:

```bash
find ~/linux-day3 -type f -name "*.log"
```

Find only configuration files:

```bash
find ~/linux-day3 -type f -name "*.conf"
```

---

# Part 16 - Find by Modification Time

Create a new file:

```bash
touch ~/linux-day3/docs/recent.txt
```

Find files modified in the last 24 hours:

```bash
find ~/linux-day3 -type f -mtime -1
```

Find files older than seven 24-hour periods:

```bash
find ~/linux-day3 -type f -mtime +7
```

Important:

`find -mtime` uses rounded 24-hour periods. It does not mean calendar dates such as "yesterday" in the everyday sense.

For minute-level checks, use `-mmin`:

```bash
find ~/linux-day3 -type f -mmin -10
```

This searches for files modified within roughly the last 10 minutes.

---

# Part 17 - Find by Size

Create a 12 MiB test file:

```bash
dd if=/dev/zero of=~/linux-day3/docs/sample.bin bs=1M count=12 status=none
```

Find files larger than 10 MiB:

```bash
find ~/linux-day3 -type f -size +10M
```

Find files smaller than 1 MiB:

```bash
find ~/linux-day3 -type f -size -1M
```

With GNU `find`, `M` represents MiB-sized units of 1024 × 1024 bytes.

---

# Part 18 - Search Text with `grep`

Search the log for `ERROR`:

```bash
grep "ERROR" ~/linux-day3/logs/app.log
```

Search for `WARN`:

```bash
grep "WARN" ~/linux-day3/logs/app.log
```

Remember the difference:

```text
find    searches filesystem entries
        names, types, time, size, paths

grep    searches text content
        lines matching a pattern
```

---

# Part 19 - Case-Insensitive Search with `grep -i`

Search for lowercase `error`:

```bash
grep "error" ~/linux-day3/logs/app.log
```

That may return nothing because `grep` is case-sensitive by default.

Try:

```bash
grep -i "error" ~/linux-day3/logs/app.log
```

---

# Part 20 - Show Line Numbers with `grep -n`

Run:

```bash
grep -n "ERROR" ~/linux-day3/logs/app.log
```

`-n` prefixes matching lines with their line number.

This is useful when you want to open an editor later and jump to the relevant area.

---

# Part 21 - Invert Matches with `grep -v`

Display all lines except `INFO` lines:

```bash
grep -v "INFO" ~/linux-day3/logs/app.log
```

This leaves lines such as:

```text
WARN
ERROR
```

---

# Part 22 - Count Matching Lines with `grep -c`

Count lines containing `INFO`:

```bash
grep -c "INFO" ~/linux-day3/logs/app.log
```

Count lines containing `ERROR`:

```bash
grep -c "ERROR" ~/linux-day3/logs/app.log
```

Important:

`grep -c` counts **matching lines**, not the total number of individual occurrences inside those lines.

---

# Part 23 - Match Whole Words with `grep -w`

Create:

```bash
cat > ~/linux-day3/docs/grep-words.txt <<'EOF'
ERROR
ERROR_CODE
NOERROR
The service returned ERROR during startup.
EOF
```

Run:

```bash
grep "ERROR" ~/linux-day3/docs/grep-words.txt
```

Then:

```bash
grep -w "ERROR" ~/linux-day3/docs/grep-words.txt
```

Compare the results.

---

# Part 24 - Show Context Around a Match

Show three lines after an error:

```bash
grep -A 3 "ERROR" ~/linux-day3/logs/app.log
```

Show two lines before an error:

```bash
grep -B 2 "ERROR" ~/linux-day3/logs/app.log
```

Show two lines before and after:

```bash
grep -C 2 "ERROR" ~/linux-day3/logs/app.log
```

These options are very useful during incident investigation because the event immediately before or after an error often provides important context.

---

# Part 25 - Use `grep` with Pipes

Search your command history for SSH commands:

```bash
history | grep "ssh"
```

Search running processes:

```bash
ps aux | grep "ssh"
```

List log files and filter by name:

```bash
find ~/linux-day3 -type f | grep "\.log$"
```

Filter the application log and page through the result:

```bash
grep -i "error" ~/linux-day3/logs/app.log | less
```

A common Linux/DevOps workflow is:

```text
command → pipe → grep → less
```
**Use grep with pipes:**

```bash
man pwd | grep "print"
man find | grep -A5 "variable"
info pwd | grep "print"
info pwd | grep -B5 "options"
history | grep "find"
---

# Part 26 - DevOps Scenario: Investigate an Application Problem

You receive this report:

> The application had a database problem during startup. Find the error and determine what happened immediately before and after it.

## Task 1 - Locate the Log

```bash
find ~/linux-day3 -type f -name "*.log"
```

## Task 2 - Find the Error

```bash
grep -n "ERROR" ~/linux-day3/logs/app.log
```

## Task 3 - Show Context

```bash
grep -C 2 "ERROR" ~/linux-day3/logs/app.log
```

## Task 4 - Inspect the End of the Log

```bash
tail -n 8 ~/linux-day3/logs/app.log
```

## Task 5 - Search Interactively

```bash
less ~/linux-day3/logs/app.log
```

Inside `less`, type:

```text
/ERROR
```

Then press:

```text
n
```

for the next match.

---

# Part 27 - Day 3 Challenge

Complete these tasks without copying the commands from the previous sections.

## Task 1

Create:

```text
~/day3-challenge/
├── config/
├── logs/
└── docs/
```

## Task 2

Use Vim or Nano to create:

```text
config/web.conf
```

with:

```text
APP_ENV=production
PORT=8080
REGION=us-east-1
```

## Task 3

Create a log containing at least 12 lines with a mixture of:

```text
INFO
WARN
ERROR
```

## Task 4

Display:

- The first 4 lines.
- The last 4 lines.

## Task 5

Start following the log with `tail -f`.

From a second terminal, append another `ERROR` line.

## Task 6

Find every `.log` file in the challenge directory.

## Task 7

Search for `ERROR` and display:

- Matching line numbers.
- Two lines before and after each error.

## Task 8

Open the log with `less` and search for `WARN`.

---

# Part 28 - Knowledge Check

### 1. What is the difference between Vim Normal mode and Insert mode?

---

### 2. Which Vim command saves and exits?

---

### 3. Which Nano shortcut saves a file?

---

### 4. What does `head -n 5 file.txt` do?

---

### 5. What does `tail -f app.log` do?

---

### 6. When would you use `less` instead of `cat`?

---

### 7. What is the difference between `find` and `grep`?

---

### 8. What is the difference between `find -name` and `find -iname`?

---

### 9. What does `find . -type f -name "*.log"` search for?

---

### 10. What does `grep -n` add to the output?

---

### 11. What does `grep -v` do?

---

### 12. Does `grep -c` count individual matches or matching lines?

---

### 13. What is the difference between `grep -A`, `grep -B`, and `grep -C`?

---

### 14. Why should wildcard patterns such as `*.log` normally be quoted when passed to `find -name`?

---

# Part 29 - Cleanup

Return to your home directory:

```bash
cd ~
```

Review before deleting:

```bash
pwd
ls
```

Remove the lab only if you no longer need it:

```bash
rm -r ~/linux-day3
```

Remove the challenge directory if you created it:

```bash
rm -r ~/day3-challenge
```

---

# Day 3 Summary

Commands practiced:

```text
vim
nano
head
tail
tail -f
cat
less
more
tac
find
grep
```

Important options practiced:

```text
head -n
tail -n
find -name
find -iname
find -type
find -mtime
find -mmin
find -size
grep -i
grep -n
grep -v
grep -c
grep -w
grep -A
grep -B
grep -C
```

Practical DevOps skills practiced:

```text
Editing configuration files
Inspecting logs
Following live logs
Locating files
Searching log contents
Showing error context
Combining commands with pipes
```

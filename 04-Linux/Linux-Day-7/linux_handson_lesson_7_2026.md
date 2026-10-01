# Hands-on Linux-07: Filters and Shell Operators

The goal of this practical training is to teach students how to use filters, pipelines, `sed`, shell operators, exit status, and special characters in Linux.

## Learning Outcomes

By the end of this hands-on training, students will be able to:

* Use filter commands.
* Build pipelines with the pipe (`|`) operator.
* Use `cat`, `tee`, `grep`, `cut`, `tr`, `wc`, `sort`, `uniq`, and `comm`.
* Use the `sed` command to preview and modify text.
* Use shell operators such as `;`, `&`, `&&`, and `||`.
* Check command exit status with `$?`.
* Use comments, escaping, and line continuation with `\`.
* Apply filters and pipelines to a simple AWS/DevOps log-analysis task.

## Outline

* **Section 1** - Using Filters
* **Exercise 1** - Filter Pipeline Practice
* **Exercise 2** - AWS/DevOps Log Analysis
* **Section 2** - The `sed` Command
* **Section 3** - Shell Operators, Exit Status & Special Characters
* **Exercise 3** - Control Operators

---

## Section 1 - Using Filters

### `cat`

* Displays file contents and can combine files.
* When used in a pipeline, `cat` simply copies standard input to standard output. Avoid using it when the next command can read the file directly.

* Create a directory and name it **filters**:

```bash
mkdir filters
cd filters
```

* Create a text file named `days.txt`:

```bash
vim days.txt
```

```text
Monday
Tuesday
Wednesday
Thursday
Friday
Saturday
Sunday
```

* View the contents of `days.txt`:

```bash
cat days.txt
```

* See what `cat` does when used repeatedly in a pipeline:

```bash
cat days.txt | cat | cat
```

* A more direct form is:

```bash
cat days.txt
```

* Create a text file named `count.txt`:

```bash
vim count.txt
```

```text
one
two
three
four
five
six
seven
eight
nine
ten
eleven
```

* View the contents:

```bash
cat count.txt
```

### `tee`

* Reads from standard input and writes the same data to standard output and to a file.
* `tee` is useful when you want to see a pipeline result and save it at the same time.

* Copy the contents of `count.txt` to `temp.txt` and also display them:

```bash
cat count.txt | tee temp.txt
```

* Check the file:

```bash
ls
cat temp.txt
```

* Append another line instead of overwriting the file:

```bash
echo "twelve" | tee -a temp.txt
```

* Verify:

```bash
cat temp.txt
```

### `grep`

* Prints lines matching a pattern.
* The most common use of `grep` is to filter lines containing a specific string.

* Create a text file named `tennis.txt`:

```bash
cat > tennis.txt
```

```text
Amelie Mauresmo, Fra
Justine Henin, BEL
Serena Williams, USA
Venus Williams, USA
```

> **Press `Ctrl+D` to send EOF.**

* View the file:

```bash
cat tennis.txt
```

* Display lines containing `Williams`:

```bash
grep "Williams" tennis.txt
```

* Search without depending on letter case:

```bash
grep -i "us" tennis.txt
```

* Display lines that do **not** contain `Williams`:

```bash
grep -v "Williams" tennis.txt
```

* Show matching line numbers:

```bash
grep -n "Williams" tennis.txt
```

* Count matching lines:

```bash
grep -c "Williams" tennis.txt
```

* Count matches through a pipeline:

```bash
grep "Williams" tennis.txt | wc -l
```

### `cut`

* The `cut` filter selects fields or character ranges from each input line.
* When working with fields, use `-d` to specify the delimiter and `-f` to select the field.

* View the contents of `/etc/passwd`:

```bash
cat /etc/passwd
```

* Display only usernames:

```bash
cut -d: -f1 /etc/passwd
```

* Display usernames and login shells:

```bash
cut -d: -f1,7 /etc/passwd
```

* Create a small CSV file:

```bash
cat << EOF > countries.csv
Country,Capital,Continent
USA,Washington,North America
France,Paris,Europe
Canada,Ottawa,North America
Germany,Berlin,Europe
EOF
```

* Display only the `Continent` field:

```bash
cut -d',' -f3 countries.csv
```

* Remove the header and display only the continent values:

```bash
tail -n +2 countries.csv | cut -d',' -f3
```

### `tr`

* The `tr` command means "translate."
* It can replace characters, convert case, squeeze repeated characters, or delete characters.

* Create a text file named `techproeducation.txt`:

```bash
cat << EOF > techproeducation.txt
Take a career voyage with us.
EOF
```

* View the contents:

```bash
cat techproeducation.txt
```

* Replace `a`, `e`, and `r` with `Q`, `A`, and `Z`:

```bash
cat techproeducation.txt | tr 'aer' 'QAZ'
```

* Write the contents of `count.txt` on one line:

```bash
cat count.txt | tr '\n' ' '
```

* Remove all lowercase vowels:

```bash
cat techproeducation.txt | tr -d 'aeiou'
```

* Convert lowercase letters to uppercase:

```bash
cat techproeducation.txt | tr 'a-z' 'A-Z'
```

### `wc`

* The `wc` command counts lines, words, bytes, and characters.

* Count lines, words, and bytes:

```bash
wc count.txt
```

* Count only lines:

```bash
wc -l count.txt
```

* Count only words:

```bash
wc -w count.txt
```

* Count bytes:

```bash
wc -c count.txt
```

> `wc -c` counts **bytes**, not characters.

* Count characters:

```bash
wc -m count.txt
```

> `wc -m` counts **characters**.

* Count the number of entries in `/etc/passwd`:

```bash
wc -l /etc/passwd
```

### `sort`

* The `sort` filter sorts lines.
* By default, `sort` performs lexicographic sorting.

* Create `marks.txt`:

```bash
cat << EOF > marks.txt
aaron   70
julia   80
albert  90
james   60
kate    60
john    80
oliver  75
tom     54
victor  30
walter  60
jane    100
EOF
```

* View the file:

```bash
cat marks.txt
```

* Sort alphabetically by the whole line:

```bash
sort marks.txt
```

* Reverse the order:

```bash
sort -r marks.txt
```

* Sort numerically:

```bash
sort -n marks.txt
```

> Because the first field contains names, `sort -n marks.txt` is mainly useful for demonstrating numeric comparison. For the marks column, sort by field 2:

```bash
sort -k2,2n marks.txt
```

* Sort by the second field in reverse numeric order:

```bash
sort -k2,2nr marks.txt
```

### `uniq`

* The `uniq` command removes adjacent duplicate lines.
* `uniq` does **not** sort the input.
* For a file containing duplicates in different locations, sort the file first so equal lines become adjacent.

* Create `trainees.txt`:

```bash
cat << EOF > trainees.txt
john
james
aaron
oliver
walter
albert
james
john
travis
mike
aaron
thomas
daniel
john
aaron
oliver
mike
john
EOF
```

* View the file:

```bash
cat trainees.txt
```

* Show unique names:

```bash
sort trainees.txt | uniq
```

* A shorter form is:

```bash
sort -u trainees.txt
```

### `comm`

* The `comm` command compares two **sorted** files line by line.
* By default, it displays three columns:
  1. Lines unique to the first file.
  2. Lines unique to the second file.
  3. Lines common to both files.

* Create `file1.txt`:

```bash
cat << EOF > file1.txt
Aaron
Bill
James
John
Oliver
Walter
EOF
```

* Create `file2.txt`:

```bash
cat << EOF > file2.txt
Guile
James
John
Raymond
EOF
```

* Sort both files:

```bash
sort file1.txt -o file1.txt
sort file2.txt -o file2.txt
```

* Compare them:

```bash
comm file1.txt file2.txt
```

* Display only lines common to both files:

```bash
comm -12 file1.txt file2.txt
```

---

## Exercise 1 - Filter Pipeline Practice

1. Create a file named `countries.csv` with the following content:

```text
Country,Capital,Continent
USA,Washington,North America
France,Paris,Europe
Canada,Ottawa,North America
Germany,Berlin,Europe
```

2. Complete the following tasks:

   * Display only the `Continent` column.
   * Remove the header.
   * Sort the continent names.
   * Display unique continent names.
   * Save the final output to `countries.txt`.

3. Display `countries.txt`.

### Suggested solution

```bash
tail -n +2 countries.csv | cut -d',' -f3 | sort | uniq > countries.txt
```

Then:

```bash
cat countries.txt
```

Expected unique values:

```text
Europe
North America
```

---

## Exercise 2 - AWS/DevOps Log Analysis

In real AWS/DevOps environments, engineers frequently inspect application and service logs to identify errors. In this exercise, you will combine `grep`, `cut`, `sort`, `uniq`, `wc`, and redirection.

### Step 1 - Create the log file

Create `app.log`:

```bash
cat << EOF > app.log
INFO Application started
INFO Database connected
ERROR Database connection failed
INFO Retrying connection
ERROR Timeout connecting to database
WARNING High memory usage
ERROR API request failed
EOF
```

### Step 2 - Display the log

```bash
cat app.log
```

### Step 3 - Find all ERROR lines

```bash
grep "ERROR" app.log
```

### Step 4 - Count the ERROR lines

```bash
grep -c "ERROR" app.log
```

You can also use a pipeline:

```bash
grep "ERROR" app.log | wc -l
```

### Step 5 - Extract the error message

The first field is the log level. Use `cut` to remove it:

```bash
grep "ERROR" app.log | cut -d' ' -f2-
```

### Step 6 - Sort the error messages

```bash
grep "ERROR" app.log | cut -d' ' -f2- | sort
```

### Step 7 - Remove duplicate error messages

```bash
grep "ERROR" app.log | cut -d' ' -f2- | sort | uniq
```

### Step 8 - Save the final result

```bash
grep "ERROR" app.log | cut -d' ' -f2- | sort | uniq > errors.txt
```

### Step 9 - Verify the result

```bash
cat errors.txt
```

### Expected result

```text
API request failed
Database connection failed
Timeout connecting to database
```

### Challenge

Modify the pipeline so that it saves the number of errors to `error_count.txt`.

```bash
grep -c "ERROR" app.log > error_count.txt
```

Then verify:

```bash
cat error_count.txt
```

---

## Section 2 - The `sed` Command

* `sed` is a stream editor. It can search, replace, filter, and delete text.

* Create a folder named `sed-command`:

```bash
cd ~
mkdir -p sed-command
cd sed-command
```

* Create `sed.txt`:

```text
Linux is an OS. Linux is life. Linux is a concept.
I like linux. You like linux. Everyone likes linux.
Linux is free. Linux is good. Linux is hope.
```

### Replacing or Substituting a String

* Preview a replacement without changing the original file:

```bash
sed 's/linux/ubuntu/' sed.txt
```

* `s` means substitution.
* `/` characters are delimiters.
* `linux` is the search pattern.
* `ubuntu` is the replacement text.

> By default, `sed` replaces only the first matching occurrence on each line.

### Changing the nth Occurrence

* Replace the third occurrence of `linux` in each line:

```bash
sed 's/linux/ubuntu/3' sed.txt
```

### Case-insensitive Replacement

```bash
sed 's/linux/ubuntu/i' sed.txt
```

### Replacing All Occurrences in a Line

```bash
sed 's/linux/ubuntu/g' sed.txt
```

### Combining `i` and `g`

```bash
sed 's/linux/ubuntu/ig' sed.txt
```

### Replacing from a Specific Occurrence

```bash
sed 's/linux/ubuntu/2ig' sed.txt
```

### Replacing a String in a Specific Line

* Change only line 2:

```bash
sed '2 s/linux/ubuntu/ig' sed.txt
```

### Deleting a Specific Line

* Delete line 2 from the output:

```bash
sed '2d' sed.txt
```

### Modifying the File Directly

First preview the change:

```bash
sed 's/linux/ubuntu/g' sed.txt
```

If the result is correct, modify the file:

```bash
sed -i 's/linux/ubuntu/g' sed.txt
```

### Creating a Backup

A safer option for practice is to create a backup:

```bash
sed -i.bak 's/linux/ubuntu/g' sed.txt
```

This keeps the original content in:

```text
sed.txt.bak
```

Verify:

```bash
cat sed.txt
cat sed.txt.bak
```

---

## Section 3 - Shell Operators, Exit Status & Special Characters

### `;` - Semicolon

* The semicolon lets you place multiple commands on one line.
* The next command runs whether the previous command succeeds or fails.

* Move back to the filters directory:

```bash
cd ../filters
```

* Run two commands sequentially:

```bash
cat days.txt ; cat count.txt
```

Another example:

```bash
echo Hello ; echo World
```

### `&` - Background Execution

* When a command ends with `&`, the shell does not wait for that command to finish.
* The command runs in the background and the shell prompt returns immediately.

* Run a foreground command:

```bash
sleep 10
```

* Run a background command:

```bash
sleep 20 &
```

* While it is running, execute other commands:

```bash
ls -l
cat count.txt
cat days.txt
```

* View background jobs:

```bash
jobs
```

* Wait for background jobs to finish:

```bash
wait
```

### `$?` - Exit Status

`$?` is a **special parameter**, not a control operator.

It contains the exit status of the most recently executed command.

* `0` normally means success.
* A **non-zero** value means the command did not succeed.

> Do not assume that every failure returns `1`. Different commands can return different non-zero exit statuses.

* Run a successful command:

```bash
ls
echo $?
```

* Run a command that does not exist:

```bash
lss
echo $?
```

A common result for `lss` is:

```text
127
```

### Common Exit Status Examples

| Exit Status | Common meaning |
| :---: | --- |
| `0` | Success |
| `1` | General error; common convention, not universal |
| `2` | Misuse of shell builtin or command syntax; context-dependent |
| `126` | Command found but cannot be executed |
| `127` | Command not found |
| `128+n` | Command terminated by signal `n` |

> Exit-status meanings are command-dependent. The important beginner rule is: **`0` usually means success; non-zero means failure or another non-success condition.**

### `&&` - Logical AND

* The second command runs only if the first command succeeds.

```bash
cat days.txt && cat count.txt
```

* If the first command fails, the second command is skipped:

```bash
cat days.text && cat count.txt
```

### `||` - Logical OR

* The second command runs only if the first command fails.

```bash
cat missing.txt || echo "File not found"
```

Another example:

```bash
echo first || echo second
```

Because `echo first` succeeds, `echo second` is not executed.

### Combining `&&` and `||`

These operators can be used for simple success/failure logic.

* Create a file:

```bash
touch deployment.txt
```

* Test whether it exists:

```bash
test -f deployment.txt && echo "File exists" || echo "File not found"
```

* Remove it and test again:

```bash
rm deployment.txt
test -f deployment.txt && echo "File exists" || echo "File not found"
```

### `#` - Comments

* Text after `#` is treated as a comment when `#` starts an unquoted comment.
* Comments are useful for documenting shell scripts.

```bash
# Check whether the file exists
ls
```

* A `#` inside quotes is part of the string:

```bash
echo '# is inside the string'
```

* Escape `#` when you want it treated as a literal character:

```bash
echo \#
```

### `\` - Escape Character

* A backslash can prevent the shell from treating the next character as special.

Example:

```bash
echo "\$100"
```

* Another example:

```bash
echo "\# is a literal hash character"
```

### `\` - Line Continuation

* A backslash at the end of a line continues the command on the next line.
* The shell treats the continued lines as one command.

```bash
echo this command is written \
not only on a single line \
but also on multiple lines.
```

---

## Exercise 3 - Control Operators

### Task 1 - Check a File

1. Search for `techproeducation.txt` in the current directory.
2. If it exists, display its contents.
3. If it does not exist, print:

```text
Too early!
```

### Task 2 - Create the File

Create the file:

```bash
echo "Congratulations." > techproeducation.txt
```

Repeat Task 1.

### Task 3 - Use `&&` and `||`

Try to solve the check with:

```bash
test -f techproeducation.txt && cat techproeducation.txt || echo "Too early!"
```

### Task 4 - Background Execution

Run:

```bash
sleep 15 &
```

Then:

```bash
jobs
```

Run another command while the process is running:

```bash
echo "I can continue working."
```

Finally:

```bash
wait
```

### Task 5 - Exit Status

Run:

```bash
ls techproeducation.txt
echo $?
```

Then:

```bash
ls missing-file.txt
echo $?
```

Explain why the two exit statuses are different.

---

## Final Challenge - AWS/DevOps Pipeline

Use the `app.log` file from Exercise 2.

Create a pipeline that:

1. Finds only `ERROR` entries.
2. Removes the `ERROR` field.
3. Sorts the messages.
4. Removes duplicates.
5. Saves the result to `final-errors.txt`.
6. Displays the saved result.

Try it without looking at the previous solution.

### Expected command

```bash
grep "ERROR" app.log | cut -d' ' -f2- | sort | uniq > final-errors.txt
cat final-errors.txt
```

### Final Check

You should now be able to explain:

* What a filter is.
* What the pipe (`|`) does.
* Why `grep file` is often preferable to `cat file | grep`.
* Why `uniq` is normally combined with `sort`.
* The difference between `wc -c` and `wc -m`.
* How `sort -n` differs from normal sorting.
* How `tee -a` differs from `tee`.
* Why `sed` should be previewed before using `-i`.
* The difference between `;`, `&`, `&&`, and `||`.
* What `$?` contains.
* Why non-zero exit statuses are not always `1`.
* The difference between `#` comments and `\` escaping.
* How these commands can be combined to analyze application logs.

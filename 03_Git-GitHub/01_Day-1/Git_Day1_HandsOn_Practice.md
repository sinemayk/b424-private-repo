# Git Day 1 — Complete Hands-On Practice

> **Goal:** Start from zero and learn Git by doing simple exercises.  
> We will work only with **local Git** today. GitHub will come later.

---

# 1. Check Git Installation

Open **Git Bash** or your terminal.

```bash
git --version
```

Example output:

```text
git version 2.x.x
```

This confirms that Git is installed.

You can also check where Git is installed:

```bash
which git
```

On Windows Command Prompt:

```bash
where git
```

---

# 2. Configure Git

Before creating commits, Git needs to know who you are.

```bash
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
```

Set `main` as the default branch:

```bash
git config --global init.defaultBranch main
```

Check the configuration:

```bash
git config --list
```

You should see something similar to:

```text
user.name=John Doe
user.email=john@example.com
init.defaultbranch=main
```

### Why do we configure name and email?

Every commit records information about its author.

For example:

```text
Author: John Doe <john@example.com>
```

---

# 3. Create Our First Git Project

Create a simple folder:

```bash
mkdir git-practice
```

Enter it:

```bash
cd git-practice
```

Check the folder:

```bash
ls
```

It should currently be empty.

Now initialize Git:

```bash
git init
```

Example output:

```text
Initialized empty Git repository in .../git-practice/.git/
```

Check hidden files:

```bash
ls -la
```

You should see:

```text
.git
```

The `.git` directory is what makes this folder a **Git repository**.

---

# 4. Understand `git status`

Run:

```bash
git status
```

Example:

```text
On branch main

No commits yet

nothing to commit
```

Think of `git status` as:

> **"Git, tell me what is happening right now."**

You will use this command constantly.

---

# 5. Create an Untracked File

Create a file:

```bash
echo "Hello Git" > app.txt
```

Check it:

```bash
cat app.txt
```

Output:

```text
Hello Git
```

Now:

```bash
git status
```

You should see something similar to:

```text
Untracked files:

    app.txt
```

The file exists, but Git is not tracking it yet.

## Git File State
```text
app.txt

Untracked
```
---
# 6. Move the File to the Staging Area
Run:
```bash
git add app.txt
```
Now check:
```bash
git status
```
You should see:
```text
Changes to be committed:

    new file: app.txt
```

The file is now **staged**.

```text
Untracked
    |
    | git add app.txt
    v
 Staged
```

Important:

> `git add` does **not** create a commit.

It only says:

> "Include this change in my next commit."

---

# 7. Create the First Commit

Now save the staged change into Git history:

```bash
git commit -m "Add app file"
```

Example output:

```text
[main abc1234] Add app file
 1 file changed, 1 insertion(+)
 create mode 100644 app.txt
```

Run:

```bash
git status
```

You should now see:

```text
nothing to commit, working tree clean
```

Our full flow was:

```text
Untracked
    |
    | git add
    v
 Staged
    |
    | git commit
    v
Committed
```

---

# Practice 1 — Repeat the File State Workflow

Create another file:

```bash
echo "My first notes" > notes.txt
```

Check:

```bash
git status
```

You should see:

```text
notes.txt
```

as **untracked**.

Stage it:

```bash
git add notes.txt
```

Check again:

```bash
git status
```

Commit:

```bash
git commit -m "Add notes file"
```

Check:

```bash
git status
```

Expected result:

```text
nothing to commit, working tree clean
```

You have now practiced:

```text
Untracked
→ Staged
→ Committed
```

twice.

---

# 8. Modify an Existing File

Our `app.txt` file is already committed.

Check it:

```bash
cat app.txt
```

Current content:

```text
Hello Git
```

Add another line:

```bash
echo "Git is easy" >> app.txt
```

Check:

```bash
cat app.txt
```

Now:

```text
Hello Git
Git is easy
```

Run:

```bash
git status
```

Git should report:

```text
modified: app.txt
```

The file was already tracked, so it is not called **untracked**.

Its state is now:

```text
Committed
    |
    | modify file
    v
Modified
```

---

# 9. Understanding `git diff`

Before staging the change, run:

```bash
git diff
```

You may see something similar to:

```diff
 Hello Git
+Git is easy
```

The important symbols are:

| Output | Meaning |
|---|---|
| `+` | A line was added |
| `-` | A line was removed |
| `@@` | Shows the location of the changed block |
| no symbol | Context / unchanged line |

Depending on your terminal theme:

- added lines are usually shown in **green**
- removed lines are usually shown in **red**

The exact colors can vary.

---

# 10. Practice Reading `+` and `-`

Change the file completely:

```bash
echo "Learning Git" > app.txt
```

Now:

```bash
git diff
```

You may see:

```diff
-Hello Git
+Learning Git
```

Read this as:

```text
- Hello Git
```

means:

> This line was removed.

And:

```text
+ Learning Git
```

means:

> This line was added.

Git is showing us the difference between:

```text
the committed version
```

and:

```text
our current working version
```

---

# Practice 2 — Create More Changes

Add two lines:

```bash
echo "Day 1" >> app.txt
echo "Practice" >> app.txt
```

Now run:

```bash
git diff
```

Try to identify:

```text
+
-
@@
```

Do not worry about understanding every character in the output.

For now, focus mainly on:

```text
+ = added
- = removed
```

---

# 11. Stage the Modified File

Run:

```bash
git add app.txt
```

Now:

```bash
git status
```

You should see:

```text
Changes to be committed:

    modified: app.txt
```

Our state is now:

```text
Committed
    |
    | modify
    v
Modified
    |
    | git add
    v
 Staged
```

---

# 12. `git diff` vs `git diff --staged`

Now run:

```bash
git diff
```

You may notice there is no output.

Why?

Because the change is no longer between the:

```text
Working Directory
```

and:

```text
Staging Area
```

The change has already been staged.

To see staged changes:

```bash
git diff --staged
```

Now Git shows the changes that will be included in the next commit.

Think of it like this:

```text
git diff

Working Directory
       ↕
Staging Area
```

And:

```text
git diff --staged

Staging Area
       ↕
Last Commit
```

---

# 13. Commit the Modified File

```bash
git commit -m "Update app content"
```

Then:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# 14. View Commit History

Run:

```bash
git log
```

You will see information such as:

```text
commit abc123456...
Author: John Doe
Date: ...

    Update app content
```

For a simpler output:

```bash
git log --oneline
```

Example:

```text
45cd781 Update app content
71ab321 Add notes file
18ef102 Add app file
```

Each line represents one commit.

---

# 15. Practice Commit History

Make another small change:

```bash
echo "Version 2" >> notes.txt
```

Stage:

```bash
git add notes.txt
```

Commit:

```bash
git commit -m "Update notes"
```

Now:

```bash
git log --oneline
```

You should see another commit at the top.

The newest commit appears first.

---

# 16. Understanding HEAD

Run:

```bash
git log --oneline --decorate
```

You may see:

```text
3fa1234 (HEAD -> main) Update notes
71ab321 Update app content
18ef102 Add app file
```

Focus on:

```text
HEAD -> main
```

A simple way to understand it:

> **HEAD shows where we currently are in Git history.**

Normally:

```text
HEAD
 |
 v
main
 |
 v
A --- B --- C
```

If `C` is our latest commit, HEAD points to `main`, and `main` points to `C`.

You do not need to memorize Git internals yet.

Just remember:

> **HEAD = our current position.**

---

# 17. Undo an Uncommitted File Change with `git restore`

Check the current content:

```bash
cat notes.txt
```

Now make a change:

```bash
echo "This change is a mistake" >> notes.txt
```

Check:

```bash
cat notes.txt
```

Run:

```bash
git status
```

You should see:

```text
modified: notes.txt
```

Check the difference:

```bash
git diff
```

Now imagine:

> "I don't want this change."

Restore the file:

```bash
git restore notes.txt
```

Check again:

```bash
cat notes.txt
```

The unwanted change should be gone.

Then:

```bash
git status
```

The working tree should be clean.

### Important

```bash
git restore notes.txt
```

discards uncommitted changes in that file.

So use it carefully.

---

# Practice 3 — Restore

Modify `app.txt`:

```bash
echo "Wrong line" >> app.txt
```

Check:

```bash
git diff
```

Then discard the change:

```bash
git restore app.txt
```

Verify:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# 18. Unstage a File

Create another modification:

```bash
echo "New configuration" >> app.txt
```

Stage it:

```bash
git add app.txt
```

Check:

```bash
git status
```

It should appear under:

```text
Changes to be committed
```

Imagine you accidentally staged it too early.

Remove it from the staging area:

```bash
git restore --staged app.txt
```

Now:

```bash
git status
```

The file should appear as modified but **not staged**.

Important:

```text
git restore --staged
```

does **not** delete your changes.

It simply moves the change:

```text
Staged
   |
   | git restore --staged
   v
Modified
```

---

# Practice 4 — Stage and Unstage

Stage the file again:

```bash
git add app.txt
```

Check:

```bash
git status
```

Unstage it:

```bash
git restore --staged app.txt
```

Check:

```bash
git status
```

Stage it again:

```bash
git add app.txt
```

Commit:

```bash
git commit -m "Update app configuration"
```

This repetition helps students clearly understand the staging area.

---


# 19. Create Your First Branch

Check your branches:

```bash
git branch
```

You should see:

```text
* main
```

The `*` tells you which branch you are currently using.

Create a new branch and switch to it:

```bash
git switch -c feature
```

Check:

```bash
git branch
```

Now:

```text
  main
* feature
```

The `*` moved to `feature`.

---

# 20. Make a Change in the Feature Branch

Create a file:

```bash
echo "New feature" > feature.txt
```

Check:

```bash
git status
```

Stage:

```bash
git add feature.txt
```

Commit:

```bash
git commit -m "Add feature file"
```

Now:

```bash
git log --oneline # --decorate
```

You may see:

```text
abc1234 (HEAD -> feature) Add feature file
...
```

---

# 19. Switch Back to Main

```bash
git switch main
```

Check files:

```bash
ls
```

Notice that `feature.txt` may disappear.

Why?

Because that file belongs to the `feature` branch.

Switch again:

```bash
git switch feature
```

Check:

```bash
ls
```

The file appears again.

This is one of the easiest ways to understand branches.

```text
main
 |
 A --- B
       \
        C
        |
      feature
```

---

# Practice 5 — Branch Switching

Switch to main:

```bash
git switch main
```

Check:

```bash
git branch
```

Switch to feature:

```bash
git switch feature
```

Check again:

```bash
git branch
```

Repeat this two or three times until students are comfortable.

---

# 20. Create Another Branch

Switch to main:

```bash
git switch main
```

Create another branch:

```bash
git switch -c documentation
```

Create:

```bash
echo "Project Documentation" > README.txt
```

Stage:

```bash
git add README.txt
```

Commit:

```bash
git commit -m "Add documentation"
```

Check history:

```bash
git log --oneline --graph --decorate --all
```

Now students can see multiple branches visually.

---

# 19. Merge a Branch

Switch back to main:

```bash
git switch main
```

Check:

```bash
ls
```

Now merge the `documentation` branch into `main`:

```bash
git merge documentation
```

Check:

```bash
ls
```

`README.txt` should now exist on `main`.

Check history:

```bash
git log --oneline --graph --decorate --all
```

---

# 20. Merge the Feature Branch

Still on `main`, run:

```bash
git merge feature
```

Check:

```bash
ls
```

Now `feature.txt` should also be available on `main`.

The basic rule is:

> First switch to the branch that should **receive** the changes.

For example:

```bash
git switch main
git merge feature
```

means:

> Merge `feature` **into `main`**.

---

# 19. Delete a Branch After Merge

Check:

```bash
git branch
```

Delete `documentation`:

```bash
git branch -d documentation
```

Delete `feature`:

```bash
git branch -d feature
```

Check:

```bash
git branch
```

You should now mainly see:

```text
* main
```

---

# 20. Simple Merge Conflict Practice

Now we intentionally create a conflict.

Create:

```bash
echo "PORT=8080" > config.txt
```

Stage:

```bash
git add config.txt
```

Commit:

```bash
git commit -m "Add port configuration"
```

---

# 19. Create a Branch for Another Port

Create:

```bash
git switch -c change-port
```

Change the same line:

```bash
echo "PORT=9000" > config.txt
```

Commit:

```bash
git add config.txt
git commit -m "Change port to 9000"
```

---

# 20. Change the Same Line on Main

Switch:

```bash
git switch main
```

Change the same file differently:

```bash
echo "PORT=8081" > config.txt
```

Commit:

```bash
git add config.txt
git commit -m "Change port to 8081"
```

Now we have:

```text
main:

PORT=8081
```

and:

```text
change-port:

PORT=9000
```

Git cannot automatically decide which one is correct.

---

# 19. Create the Conflict

While on `main`:

```bash
git merge change-port
```

You may see:

```text
CONFLICT (content): Merge conflict in config.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Run:

```bash
git status
```

Git tells us that we have an unmerged file.

---

# 20. Read the Conflict Markers

Open:

```bash
cat config.txt
```

You may see:

```text
<<<<<<< HEAD
PORT=8081
=======
PORT=9000
>>>>>>> change-port
```

Meaning:

```text
<<<<<<< HEAD
```

shows the version from our current branch.

```text
=======
```

separates the two versions.

```text
>>>>>>> change-port
```

shows the incoming branch version.

---

# 19. Resolve the Conflict

Suppose we decide:

```text
PORT=9000
```

is correct.

Edit the file so it contains only:

```text
PORT=9000
```

For a quick terminal example:

```bash
echo "PORT=9000" > config.txt
```

Check:

```bash
cat config.txt
```

Now:

```bash
git add config.txt
```

Then:

```bash
git commit -m "Resolve port conflict"
```

Check:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

The conflict is resolved.

---

# Practice 6 — Another Simple Merge Conflict

Create:

```bash
echo "COLOR=blue" > settings.txt
git add settings.txt
git commit -m "Add color setting"
```

Create a branch:

```bash
git switch -c red-color
```

Change:

```bash
echo "COLOR=red" > settings.txt
git add settings.txt
git commit -m "Change color to red"
```

Switch to main:

```bash
git switch main
```

Change:

```bash
echo "COLOR=green" > settings.txt
git add settings.txt
git commit -m "Change color to green"
```

Merge:

```bash
git merge red-color
```

Git should create another conflict.

Students should now resolve it themselves.

For example, choose:

```text
COLOR=red
```

Then:

```bash
git add settings.txt
git commit -m "Resolve color conflict"
```

This second practice is important because students usually understand conflicts much better after resolving them twice.

---

# 20. Terminal Output Practice

Run:

```bash
git status
```

Students should be able to identify these situations.

### Example 1 — Untracked

```text
Untracked files:
    test.txt
```

Meaning:

> Git sees the file, but we have never added it.

---

### Example 2 — Modified but Not Staged

```text
Changes not staged for commit:

    modified: app.txt
```

Meaning:

> Git tracks the file, but its latest change is not staged.

---

### Example 3 — Staged

```text
Changes to be committed:

    modified: app.txt
```

Meaning:

> The change is ready for the next commit.

---

### Example 4 — Clean

```text
nothing to commit, working tree clean
```

Meaning:

> Everything is committed.

---

# 19. Quick File-State Challenge

Create:

```bash
echo "Practice file" > challenge.txt
```

Before running any command, ask students:

> What state is the file in?

Answer:

```text
Untracked
```

Run:

```bash
git add challenge.txt
```

Ask:

> What state now?

Answer:

```text
Staged
```

Run:

```bash
git commit -m "Add challenge file"
```

Ask:

> What state now?

Answer:

```text
Committed
```

Modify:

```bash
echo "New line" >> challenge.txt
```

Ask:

> What state now?

Answer:

```text
Modified
```

Then:

```bash
git add challenge.txt
```

Answer:

```text
Staged
```

Then:

```bash
git commit -m "Update challenge file"
```

Answer:

```text
Committed
```

This short exercise is excellent for checking understanding.

---

# 20. Final Day 1 Practice

Students now complete the following without copying every command.

Create a new project:

```bash
mkdir final-git-practice
cd final-git-practice
git init
```

Create:

```text
app.txt
config.txt
notes.txt
```

The student should complete this workflow:

```text
Create app.txt
       ↓
Check git status
       ↓
Stage app.txt
       ↓
Commit
       ↓
Modify app.txt
       ↓
Check git diff
       ↓
Stage
       ↓
Check git diff --staged
       ↓
Commit
       ↓
Create feature branch
       ↓
Create feature.txt
       ↓
Commit
       ↓
Switch to main
       ↓
Merge feature
       ↓
Create a simple conflict
       ↓
Resolve conflict
       ↓
View history
```

At the end run:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

Then:

```bash
git log --oneline --graph --decorate --all
```

Students should be able to explain what they see.

---

# Day 1 Command Summary

```bash
# Git configuration
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
git config --global init.defaultBranch main
git config --list

# Create repository
git init

# Check repository
git status

# Stage files
git add app.txt
git add .

# Commit
git commit -m "Commit message"

# History
git log
git log --oneline
git log --oneline --graph --decorate --all

# Compare changes
git diff
git diff --staged

# Restore changes
git restore app.txt

# Unstage
git restore --staged app.txt

# Branch
git branch
git switch -c feature
git switch main
git branch -d feature

# Merge
git merge feature
```

# Day 1 Mental Model

By the end of the lesson, students should understand this without memorizing it mechanically:

```text
                         MODIFY
                           |
                           v
                     +----------+
                     | Modified |
                     +----------+
                          |
                       git add
                          |
                          v

+-----------+  git add  +--------+  git commit  +-----------+
| Untracked | --------> | Staged | -----------> | Committed |
+-----------+           +--------+              +-----------+
                            ^
                            |
                  git restore --staged
                            |
                         Modified
```

And the most important habit for beginners should be:

```bash
git status
```

> **Whenever you are confused, run `git status`.**

It tells you where your files are and usually tells you what command you can use next.

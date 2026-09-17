# Git & GitHub Day 3 — Complete Hands-On Practice

> **Day 3 Goal:** Move from everyday Git/GitHub usage into **DevOps-oriented Git workflows and automation**.
>
> Today we will practice:
>
> `stash → reset → revert → rebase → cherry-pick → tags → releases → README → CI/CD → GitHub Actions → secrets → AWS OIDC`

---

# 1. Create a Fresh Day 3 Repository

Create a new practice project:

```bash
mkdir day3-git-practice
cd day3-git-practice
git init
```

Create a simple file:

```bash
echo "Version 1" > app.txt
```

Stage and commit:

```bash
git add app.txt
git commit -m "Create app file"
```

Check:

```bash
git status
git log --oneline
```

Make sure the branch is called `main`:

```bash
git branch -M main
```

Our starting point is:

```text
A
↑
main
HEAD
```

---

# 2. Git Stash — Temporarily Save Unfinished Work

Imagine that you are working on `app.txt`.

Modify it:

```bash
echo "Work in progress" >> app.txt
```

Check:

```bash
git status
```

You should see:

```text
modified: app.txt
```

But imagine you suddenly need to work on something else.

The change is **not ready to commit**.

Use:

```bash
git stash
```

Check:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

Check the file:

```bash
cat app.txt
```

The unfinished change is temporarily gone from the working directory.

---

# 3. View Your Stash

Run:

```bash
git stash list
```

Example:

```text
stash@{0}: WIP on main: ...
```

Git has temporarily stored your changes.

Think:

```text
Working Directory
      ↓
   git stash
      ↓
Temporary Storage
```

---

# 4. Bring the Stashed Work Back

Run:

```bash
git stash pop
```

Check:

```bash
cat app.txt
```

You should see:

```text
Version 1
Work in progress
```

Check:

```bash
git status
```

The modification is back.

Commit it:

```bash
git add app.txt
git commit -m "Update app"
```

---

# Practice 1 — Stash Again

Modify the file:

```bash
echo "Temporary change" >> app.txt
```

Check:

```bash
git diff
```

Stash:

```bash
git stash
```

Check:

```bash
git status
```

Bring it back:

```bash
git stash pop
```

Check:

```bash
git diff
```

Then discard this practice change:

```bash
git restore app.txt
```

---

# 5. Stash Without Immediately Removing It

Create another modification:

```bash
echo "Another unfinished change" >> app.txt
```

Stash:

```bash
git stash
```

Check:

```bash
git stash list
```

Instead of:

```bash
git stash pop
```

you can use:

```bash
git stash apply
```

Difference:

```text
git stash pop
→ restores the stash
→ removes it from the stash list
```

```text
git stash apply
→ restores the stash
→ keeps it in the stash list
```

After practicing:

```bash
git restore app.txt
```

Delete the stash:

```bash
git stash drop stash@{0}
```

---

# 6. Undoing Changes — Mental Model

Before learning `reset` and `revert`, understand where the mistake exists.

```text
Working Directory
      |
      | git restore
      v

Staging Area
      |
      | git restore --staged
      v

Commit History
      |
      | git reset / git revert
      v
```

The command depends on **where the mistake is**.

---

# 7. Quick Review — `git restore`

Make an unwanted change:

```bash
echo "WRONG CHANGE" >> app.txt
```

Check:

```bash
git diff
```

Discard it:

```bash
git restore app.txt
```

Check:

```bash
git status
```

---

# 8. Git Reset — Undo a Local Commit

First create a practice commit:

```bash
echo "Practice reset" >> app.txt
git add app.txt
git commit -m "Practice reset commit"
```

Check:

```bash
git log --oneline
```

You should see your new commit at the top.

---

# 9. `git reset --soft`

Run:

```bash
git reset --soft HEAD~1
```

Check:

```bash
git log --oneline
```

The last commit is gone from the branch history.

But check:

```bash
git status
```

The changes are still **staged**.

Think:

```text
git reset --soft HEAD~1

Remove commit
     ↓
Keep changes staged
```

Commit again:

```bash
git commit -m "Practice reset commit"
```

---

# Practice 2 — Soft Reset

Create another commit:

```bash
echo "Soft reset practice" >> app.txt
git add app.txt
git commit -m "Temporary commit"
```

Check:

```bash
git log --oneline -3
```

Undo it:

```bash
git reset --soft HEAD~1
```

Check:

```bash
git status
```

Ask:

> Are the changes staged or unstaged?

Answer:

```text
Staged
```

Now commit again:

```bash
git commit -m "Keep soft reset practice"
```

---

# 10. Mixed Reset

Create another commit:

```bash
echo "Mixed reset practice" >> app.txt
git add app.txt
git commit -m "Practice mixed reset"
```

Now:

```bash
git reset HEAD~1
```

`git reset` without a mode normally performs a mixed reset.

Check:

```bash
git status
```

The commit is gone, but the file modification remains **unstaged**.

Think:

```text
git reset HEAD~1

Remove commit
     ↓
Keep changes
     ↓
Changes are NOT staged
```

Stage and recommit:

```bash
git add app.txt
git commit -m "Keep mixed reset practice"
```

---

# 11. Soft vs Mixed Reset

```text
git reset --soft HEAD~1

Commit removed
Changes kept
Changes STAGED
```

versus:

```text
git reset HEAD~1

Commit removed
Changes kept
Changes NOT STAGED
```

Simple memory trick:

```text
SOFT
→ changes stay closer to commit
→ staged
```

```text
MIXED
→ changes return to working directory
→ unstaged
```

---

# 12. `git reset --hard` — Instructor Demo

There is also:

```bash
git reset --hard HEAD~1
```

This can remove the commit **and discard file changes**.

```text
Commit
   ↓
reset --hard
   ↓
Commit removed
Changes removed
```

For beginners:

> Use `--hard` very carefully.

Do not use it on important uncommitted work.

For this course, understanding it is more important than repeatedly practicing it.

---

# 13. Git Revert — Safely Undo a Commit

Create a good commit:

```bash
echo "Application is working" > status.txt
git add status.txt
git commit -m "Add application status"
```

Now create a bad change:

```bash
echo "Application is BROKEN" > status.txt
git add status.txt
git commit -m "Break application status"
```

Check:

```bash
git log --oneline
```

Example:

```text
a123456 Break application status
b234567 Add application status
```

We want to undo the bad commit.

Use:

```bash
git revert HEAD
```

Your editor may open for the revert commit message.

Save and exit.

Now:

```bash
git log --oneline
```

You should see something similar to:

```text
c345678 Revert "Break application status"
a123456 Break application status
b234567 Add application status
```

Check:

```bash
cat status.txt
```

The good content should be restored.

---

# 14. Why Is Revert Different?

`revert` does **not delete the bad commit**.

Instead:

```text
Good Commit
    ↓
Bad Commit
    ↓
Revert Commit
```

The history remains visible.

That makes `revert` useful for shared repositories.

---

# Practice 3 — Revert

Create:

```bash
echo "PORT=8080" > config.txt
git add config.txt
git commit -m "Add port configuration"
```

Make a bad change:

```bash
echo "PORT=99999" > config.txt
git add config.txt
git commit -m "Set incorrect port"
```

Check:

```bash
git log --oneline -3
```

Undo the last commit safely:

```bash
git revert HEAD
```

Check:

```bash
cat config.txt
```

Expected:

```text
PORT=8080
```

---

# 15. Reset vs Revert

| `git reset` | `git revert` |
|---|---|
| Moves branch history | Creates a new commit |
| Can rewrite history | Preserves history |
| Useful for local/unshared cleanup | Safer for shared history |
| `--hard` can destroy work | Previous commits remain visible |

Simple classroom rule:

```text
Not pushed / private history
→ reset may be appropriate
```

```text
Already pushed / team repository
→ usually prefer revert
```

---

# 16. Merge vs Rebase

Create a clean situation.

Make sure you are on main:

```bash
git switch main
```

Create a branch:

```bash
git switch -c feature
```

Add:

```bash
echo "Feature work" > feature.txt
git add feature.txt
git commit -m "Add feature"
```

Now switch to main:

```bash
git switch main
```

Create another commit:

```bash
echo "Main update" > main.txt
git add main.txt
git commit -m "Update main"
```

Our history is now conceptually:

```text
        feature
           |
           C
          /
A --- B --- D
              |
             main
```

---

# 17. Rebase the Feature Branch

Switch:

```bash
git switch feature
```

Run:

```bash
git rebase main
```

Now view:

```bash
git log --oneline --graph --decorate --all
```

The feature commit has been replayed on top of `main`.

Conceptually:

```text
Before:

A --- B --- D   main
       \
        C       feature
```

After rebase:

```text
A --- B --- D --- C'
                    |
                  feature
```

The history becomes more linear.

---

# 18. Important Rebase Rule

Rebase rewrites commits.

So avoid casually rebasing commits that other team members are already using.

Simple rule:

```text
Local feature branch
→ rebase can be useful
```

```text
Shared public history
→ be careful
```

---

# 19. Merge the Rebasing Feature

Switch to main:

```bash
git switch main
```

Merge:

```bash
git merge feature
```

Check:

```bash
git log --oneline --graph --decorate --all
```

Delete the finished branch:

```bash
git branch -d feature
```

---

# 20. Git Cherry-Pick

Cherry-pick means:

> Bring **one specific commit** from another branch.

Create a branch:

```bash
git switch -c useful-change
```

Create:

```bash
echo "Useful configuration" > useful.txt
git add useful.txt
git commit -m "Add useful configuration"
```

Get the commit ID:

```bash
git log --oneline -1
```

Example:

```text
abcd123 Add useful configuration
```

Remember your real commit ID.

---

# 21. Cherry-Pick That Commit

Switch back:

```bash
git switch main
```

The file may not exist:

```bash
ls
```

Now cherry-pick:

```bash
git cherry-pick <commit-id>
```

For example:

```bash
git cherry-pick abcd123
```

Check:

```bash
ls
```

Now:

```bash
git log --oneline
```

The useful commit has been copied onto `main`.

Conceptually:

```text
feature branch

A --- B --- C
          ↑
        want C
```

```text
main

A --- D
      |
 cherry-pick C
      ↓
A --- D --- C'
```

---

# Practice 4 — Cherry-Pick

Create:

```bash
git switch -c hotfix
```

Create:

```bash
echo "HOTFIX=true" > hotfix.txt
git add hotfix.txt
git commit -m "Add hotfix"
```

Copy the commit ID:

```bash
git log --oneline -1
```

Switch:

```bash
git switch main
```

Cherry-pick:

```bash
git cherry-pick <commit-id>
```

Verify:

```bash
cat hotfix.txt
```

---

# 22. Git Tags

Tags give meaningful names to important commits.

Check existing tags:

```bash
git tag
```

Probably nothing appears.

Create an annotated tag:

```bash
git tag -a v1.0.0 -m "First stable version"
```

Check:

```bash
git tag
```

Expected:

```text
v1.0.0
```

Show it:

```bash
git show v1.0.0
```

Concept:

```text
A --- B --- C --- D
          ↑
        v1.0.0
```

A branch moves as new commits are created.

A tag normally stays attached to one specific commit.

---

# 23. Semantic Versioning

A common version format is:

```text
v1.2.3
```

Think:

```text
MAJOR.MINOR.PATCH
```

Examples:

```text
v1.0.0
First stable version
```

```text
v1.1.0
New backward-compatible feature
```

```text
v1.1.1
Bug fix
```

```text
v2.0.0
Major/breaking change
```

---

# Practice 5 — Create More Tags

Create:

```bash
echo "Small fix" >> app.txt
git add app.txt
git commit -m "Fix application"
```

Create:

```bash
git tag -a v1.0.1 -m "Bug fix release"
```

Check:

```bash
git tag
```

Expected:

```text
v1.0.0
v1.0.1
```

---

# 24. Connect Day 3 Repository to GitHub

Create an empty GitHub repository named:

```text
day3-git-practice
```

Then:

```bash
git remote add origin https://github.com/YOUR_USERNAME/day3-git-practice.git
```

Check:

```bash
git remote -v
```

Push:

```bash
git push -u origin main
```

Tags are not automatically pushed by a normal push.

Push them:

```bash
git push origin v1.0.0
git push origin v1.0.1
```

Or push all local tags:

```bash
git push origin --tags
```

---

# 25. Create a GitHub Release

On GitHub:

```text
Repository
→ Releases
→ Draft a new release
```

Choose:

```text
v1.0.0
```

Title:

```text
Version 1.0.0
```

Description:

```text
First stable release.

Features:
- Basic application
- Configuration file
- Git practice examples
```

Publish the release.

Mental model:

```text
Commit
   ↓
Tag
   ↓
GitHub Release
```

---

# 26. Create a Better README

Create or replace `README.md`:

````markdown
# Day 3 Git Practice

## Description

A simple repository for practicing advanced Git and GitHub Actions.

## Topics

- Git stash
- Git reset and revert
- Rebase
- Cherry-pick
- Tags and releases
- GitHub Actions

## Requirements

- Git
- GitHub account

## Run

```bash
cat app.txt
```

## CI/CD

GitHub Actions is used to run automated workflows.
````

Then:

```bash
git add README.md
git commit -m "Improve project README"
git push
```

---

# 27. What Happens After `git push`?

Until now:

```text
Developer
    ↓
Commit
    ↓
Push
    ↓
GitHub
```

But DevOps asks:

> Can something happen automatically after the push?

Yes.

For example:

```text
git push
    ↓
GitHub
    ↓
Build
    ↓
Test
    ↓
Deploy
```

This is where CI/CD begins.

---

# 28. Create Your First GitHub Actions Workflow

Create the directory:

```bash
mkdir -p .github/workflows
```

Create:

```bash
touch .github/workflows/first-workflow.yml
```

Add:

```yaml
name: First Workflow

on:
  push:
    branches:
      - main

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v7

      - name: Show Message
        run: echo "GitHub Actions is running!"

      - name: Show Files
        run: ls -la
```

---

# 29. Commit the Workflow

Run:

```bash
git status
```

Stage:

```bash
git add .github/workflows/first-workflow.yml
```

Commit:

```bash
git commit -m "Add first GitHub Actions workflow"
```

Push:

```bash
git push
```

Now open GitHub:

```text
Repository
→ Actions
```

You should see:

```text
First Workflow
```

running or completed.

---

# 30. Understand the Workflow

Our YAML:

```yaml
name: First Workflow
```

means:

> Name of the workflow.

```yaml
on:
  push:
```

means:

> Run when a push occurs.

```yaml
jobs:
```

means:

> Define the work GitHub should perform.

```yaml
runs-on: ubuntu-latest
```

means:

> Use a GitHub-hosted Ubuntu runner.

```yaml
steps:
```

means:

> Commands/actions performed inside the job.

---

# 31. Add Another Step

Edit the workflow:

```yaml
      - name: Show Git Version
        run: git --version
```

The full end of the workflow becomes:

```yaml
      - name: Show Files
        run: ls -la

      - name: Show Git Version
        run: git --version
```

Commit:

```bash
git add .
git commit -m "Add Git version check"
git push
```

Go back to:

```text
GitHub → Actions
```

A new workflow run should start.

---

# Practice 6 — Trigger Actions Again

Modify:

```bash
echo "CI/CD practice" >> app.txt
```

Commit:

```bash
git add app.txt
git commit -m "Update app for CI practice"
```

Push:

```bash
git push
```

Question:

> What triggered the workflow?

Answer:

```text
push to main
```

---

# 32. Create a Manual Workflow

Add another workflow:

```bash
touch .github/workflows/manual.yml
```

Content:

```yaml
name: Manual Workflow

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:
      - name: Manual Message
        run: echo "This workflow was started manually."
```

Commit:

```bash
git add .
git commit -m "Add manual workflow"
git push
```

Now open:

```text
GitHub
→ Actions
→ Manual Workflow
```

You should see:

```text
Run workflow
```

Run it manually.

---

# 33. Pull Request Workflow

Now connect Day 2 and Day 3.

Create:

```bash
touch .github/workflows/pr-check.yml
```

Add:

```yaml
name: Pull Request Check

on:
  pull_request:
    branches:
      - main

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v7

      - name: Validate
        run: echo "Pull request validation passed!"
```

Commit:

```bash
git add .
git commit -m "Add pull request workflow"
git push
```

---

# 34. Trigger the PR Workflow

Create:

```bash
git switch -c feature-pr-test
```

Change:

```bash
echo "PR test" > pr-test.txt
```

Commit:

```bash
git add pr-test.txt
git commit -m "Add PR test file"
```

Push:

```bash
git push -u origin feature-pr-test
```

On GitHub:

```text
Create Pull Request
```

Do **not** merge immediately.

Look at the PR.

You should see the GitHub Actions check running.

Concept:

```text
Feature Branch
      ↓
Push
      ↓
Pull Request
      ↓
GitHub Actions
      ↓
Check
      ↓
Review
      ↓
Merge
```

---

# Practice 7 — Failed Workflow

It is useful for students to see failure too.

Change the PR workflow temporarily:

```yaml
      - name: Intentionally Fail
        run: exit 1
```

Commit and push:

```bash
git add .
git commit -m "Practice failed workflow"
git push
```

Open the Pull Request.

The check should fail.

Open:

```text
Actions
→ Failed workflow
→ Failed job
```

Find:

```text
exit 1
```

Now students understand:

```text
Green check
→ workflow passed
```

```text
Red X
→ workflow failed
```

Remove the failing step afterward.

---

# 35. GitHub Secrets

Never put credentials directly into YAML like:

```yaml
password: my-secret-password
```

Instead use GitHub Secrets.

Create a harmless classroom secret.

On GitHub:

```text
Repository
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

Name:

```text
DEMO_MESSAGE
```

Value:

```text
hello-from-secret
```

---

# 36. Use the Secret Safely

Create:

```bash
touch .github/workflows/secret-demo.yml
```

Add:

```yaml
name: Secret Demo

on:
  workflow_dispatch:

jobs:
  secret-test:
    runs-on: ubuntu-latest

    steps:
      - name: Check Secret
        env:
          DEMO_MESSAGE: ${{ secrets.DEMO_MESSAGE }}
        run: |
          if [ -n "$DEMO_MESSAGE" ]; then
            echo "Secret is available."
          else
            echo "Secret is missing."
            exit 1
          fi
```

Commit:

```bash
git add .
git commit -m "Add secret demo workflow"
git push
```

Run it manually from GitHub Actions.

Expected:

```text
Secret is available.
```

We intentionally do **not** print the secret itself.

---

# 37. Variables vs Secrets

Simple distinction:

```text
Variable
→ normal configuration
→ not sensitive
```

Examples:

```text
APP_NAME
ENVIRONMENT
AWS_REGION
```

```text
Secret
→ sensitive value
```

Examples:

```text
API_TOKEN
PASSWORD
PRIVATE_KEY
```

Never treat a secret as normal source code.

---

# 38. GitHub Actions → AWS with OIDC

For AWS, the preferred architecture is:

```text
GitHub Actions
      ↓
     OIDC
      ↓
AWS IAM Role
      ↓
Temporary Credentials
      ↓
AWS
```

Instead of storing long-lived AWS access keys in GitHub, GitHub Actions can authenticate to AWS using OIDC and temporary credentials.

---

# 39. AWS OIDC Workflow Example — Instructor Preview

You do **not** need to configure the full IAM side during this Git class.

Use this to understand the structure:

```yaml
name: AWS OIDC Demo

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  aws-demo:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v7

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: us-east-1

      - name: Check AWS Identity
        run: aws sts get-caller-identity
```

Two important lines are:

```yaml
permissions:
  id-token: write
```

and:

```yaml
role-to-assume:
```

For now, understand the architecture.

You can configure the real IAM role later in the AWS/CI-CD module.

---

# 40. Final Day 3 Workflow

Students should now understand:

```text
Developer
    ↓
Issue
    ↓
Feature Branch
    ↓
Commit
    ↓
Push
    ↓
Pull Request
    ↓
GitHub Actions
    ↓
Automated Check
    ↓
Code Review
    ↓
Merge
    ↓
main
    ↓
Release
    ↓
Deploy
```

And later:

```text
main
  ↓
GitHub Actions
  ↓
OIDC
  ↓
AWS IAM
  ↓
AWS
```

---

# 41. Final Day 3 Challenge

Use your `day3-git-practice` repository.

Complete this without copying every command.

### Part 1 — Git

Create a branch:

```text
feature-final
```

Create:

```text
final.txt
```

Commit it.

Use:

```text
git stash
```

at least once during the exercise.

Create another local commit and practice:

```text
git reset --soft HEAD~1
```

Then recommit it correctly.

---

### Part 2 — Versioning

Create:

```text
v1.1.0
```

Push the tag.

Create a GitHub Release for:

```text
v1.1.0
```

---

### Part 3 — GitHub Actions

Create a workflow that runs on:

```text
pull_request
```

The workflow should:

```text
Checkout repository
↓
Show Git version
↓
List project files
↓
Print "Validation completed"
```

---

### Part 4 — Pull Request

Push your feature branch.

Create a Pull Request.

Verify that the workflow runs.

Check:

```text
Files changed
```

Check the automated workflow result.

Merge only after the workflow succeeds.

---

### Part 5 — Finish

Update local main:

```bash
git switch main
git pull
```

Check:

```bash
git log --oneline --graph --decorate --all -15
```

Check tags:

```bash
git tag
```

Check status:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

---

# Day 3 Command Summary

```bash
# Stash
git stash
git stash list
git stash pop
git stash apply
git stash drop stash@{0}

# Reset
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1

# Revert
git revert HEAD
git revert <commit-id>

# Rebase
git switch feature
git rebase main

# Cherry-pick
git cherry-pick <commit-id>

# Tags
git tag
git tag -a v1.0.0 -m "Version 1.0.0"
git show v1.0.0
git push origin v1.0.0
git push origin --tags

# History
git log --oneline
git log --oneline --graph --decorate --all
```

# Three-Day Git/GitHub Mental Model

```text
DAY 1
─────
Working Directory
      ↓
Staging Area
      ↓
Local Repository
      ↓
Branches / Merge / Conflicts


DAY 2
─────
Local Repository
      ↕
GitHub
      ↓
Issues
      ↓
Feature Branches
      ↓
Pull Requests
      ↓
Code Review


DAY 3
─────
Advanced Git
      ↓
Tags / Releases
      ↓
GitHub Actions
      ↓
CI/CD
      ↓
AWS
```


> **Git tracks our changes. GitHub helps the team collaborate. GitHub Actions turns repository events into DevOps automation.**

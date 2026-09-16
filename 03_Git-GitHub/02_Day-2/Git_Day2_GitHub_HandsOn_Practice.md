# Git & GitHub Day 2 — Complete Hands-On Practice

> **Day 2 Goal:** Connect local Git repositories to GitHub and practice the basic collaboration workflow.
>
> Today we will work with:
>
> **Local Repository → Remote Repository → Push / Clone / Fetch / Pull → Branch → Pull Request → Review → Merge**

---

# 0. Before We Start

You should already know these Day 1 commands:

```bash
git status
git add .
git commit -m "message"
git log --oneline
git branch
git switch
git merge
```

You also need:

- Git installed
- A GitHub account
- Your Git username and email configured

Check:

```bash
git --version
```

Check your identity:

```bash
git config --global user.name
git config --global user.email
```

If needed:

```bash
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
```

---

# 1. Create a Fresh Local Project

Create a new folder:

```bash
mkdir day2-github-practice
cd day2-github-practice
```

Initialize Git:

```bash
git init
```

Create a simple file:

```bash
echo "# Day 2 GitHub Practice" > README.md
```

Check the state:

```bash
git status
```

Stage it:

```bash
git add README.md
```

Commit it:

```bash
git commit -m "Initial commit"
```

Check:

```bash
git status
```

Expected:

```text
nothing to commit, working tree clean
```

Check the branch name:

```bash
git branch
```

If your branch is not `main`, rename it:

```bash
git branch -M main
```

---

# 2. Create an Empty Repository on GitHub

Go to GitHub.

Create a new repository.

Use a simple name:

```text
day2-github-practice
```

For this exercise:

- Choose **Public** or **Private**
- Do **not** initialize it with a README
- Do **not** add `.gitignore`
- Do **not** add a license

We already created a local repository, so keeping the GitHub repository empty makes the first connection easier.

After creating the repository, GitHub will show a URL similar to:

```text
https://github.com/YOUR_USERNAME/day2-github-practice.git
```

Copy your repository URL.

---

# 3. Connect the Local Repository to GitHub

Check whether a remote already exists:

```bash
git remote -v
```

For a new local repository, there should normally be no output.

Add GitHub as a remote:

```bash
git remote add origin https://github.com/YOUR_USERNAME/day2-github-practice.git
```

Check again:

```bash
git remote -v
```

Example:

```text
origin  https://github.com/YOUR_USERNAME/day2-github-practice.git (fetch)
origin  https://github.com/YOUR_USERNAME/day2-github-practice.git (push)
```

## What is `origin`?

`origin` is simply the conventional name for our remote repository.

Think of it like this:

```text
Local Repository
       |
       | origin
       v
GitHub Repository
```

---

# Practice 1 — Check the Remote

Run:

```bash
git remote
```

Expected:

```text
origin
```

Then:

```bash
git remote -v
```

Answer these questions:

1. What is the remote name?
2. What URL does it point to?
3. Do you see both `(fetch)` and `(push)`?

---

# 4. Push to GitHub for the First Time

Run:

```bash
git push -u origin main
```

Break it down:

```text
git push
```

means:

> Send local commits to a remote repository.

```text
origin
```

means:

> Use the remote named `origin`.

```text
main
```

means:

> Push the `main` branch.

```text
-u
```

means:

> Remember the upstream relationship.

After the first push, later you can normally use:

```bash
git push
```

Refresh the GitHub repository page.

You should now see:

```text
README.md
```

and your first commit.

---

# Practice 2 — Make Another Commit and Push It

Create another file:

```bash
echo "Hello from my local computer" > app.txt
```

Check:

```bash
git status
```

Stage:

```bash
git add app.txt
```

Commit:

```bash
git commit -m "Add app file"
```

Check your history:

```bash
git log --oneline
```

Now push:

```bash
git push
```

Refresh GitHub.

`app.txt` should now appear there.

### Mental Model

```text
Local change
     ↓
git add
     ↓
git commit
     ↓
Local Repository
     ↓
git push
     ↓
GitHub
```

---

# 5. Understand Local and Remote Repositories

At this point we have two repositories:

```text
YOUR COMPUTER                         GITHUB

Working Directory
       ↓
Staging Area
       ↓
Local Repository  ── git push ──>  Remote Repository
```

Important:

> `git commit` does not send anything to GitHub.

It saves the change in your **local repository**.

Only:

```bash
git push
```

sends commits to GitHub.

---

# Quick Check

Create a new local commit but do **not** push it yet:

```bash
echo "Local only" > local.txt
git add local.txt
git commit -m "Add local file"
```

Check GitHub in your browser.

Is `local.txt` there?

**No.**

Now:

```bash
git push
```

Refresh GitHub.

Now it should appear.

---

# 6. Clone a Repository

`git clone` is used when the repository already exists on GitHub and we want a local copy.

Move outside the current project:

```bash
cd ..
```

Clone your GitHub repository into a different folder:

```bash
git clone https://github.com/YOUR_USERNAME/day2-github-practice.git day2-clone
```

Enter it:

```bash
cd day2-clone
```

Check files:

```bash
ls
```

Check history:

```bash
git log --oneline
```

Check the remote:

```bash
git remote -v
```

Notice:

```text
origin
```

already exists.

Why?

Because `git clone` automatically:

- downloads project files
- downloads Git history
- creates a local Git repository
- creates the remote called `origin`
- checks out the default branch

---

# Practice 3 — Clone and Inspect

Run:

```bash
git status
```

Then:

```bash
git branch
```

Then:

```bash
git remote -v
```

Then:

```bash
git log --oneline --decorate -5
```

Students should identify:

- current branch
- remote name
- recent commits

---

# 7. Make a Change Directly on GitHub

Now we want to create a remote change.

On the GitHub website:

1. Open `README.md`
2. Click **Edit**
3. Add this line:

```text
This line was added on GitHub.
```

4. Commit the change on GitHub

Do not pull yet.

Go back to the terminal.

Your local clone does not automatically know about the new change.

Run:

```bash
git status
```

It may still say:

```text
nothing to commit, working tree clean
```

That is normal.

---

# 8. Use `git fetch`

Run:

```bash
git fetch origin
```

`git fetch` downloads information about remote changes.

It does **not** automatically integrate them into your current branch.

Check the history:

```bash
git log --oneline --decorate --all -8
```

You may see that:

```text
origin/main
```

is ahead of your local:

```text
main
```

Compare them:

```bash
git diff main origin/main
```

You should see the README change that exists remotely.

But check your local file:

```bash
cat README.md
```

The remote change has not yet been integrated into your local `main`.

---

# 9. Use `git pull`

Now run:

```bash
git pull
```

Check:

```bash
cat README.md
```

You should now see:

```text
This line was added on GitHub.
```

## Fetch vs Pull

```text
git fetch
    ↓
Download remote information
    ↓
Do not automatically integrate it
```

```text
git pull
    ↓
Download remote changes
    +
Integrate them into the current branch
```

Simple rule:

> **Fetch = look first**

> **Pull = bring the changes into my current branch**

---

# Practice 4 — Fetch First, Then Pull

Make another small edit to `README.md` on GitHub:

```text
Second remote update.
```

In the terminal:

```bash
git fetch
```

Check:

```bash
git log --oneline --decorate --all -8
```

Then:

```bash
git diff main origin/main
```

Finally:

```bash
git pull
```

Verify:

```bash
cat README.md
```

Repeat this practice once more if needed.

---

# 10. GitHub Authentication

When pushing or cloning private repositories, GitHub needs to know who you are.

Two common approaches are:

```text
HTTPS
```

and:

```text
SSH
```

---

# 11. Option A — SSH Authentication

Generate an SSH key:

```bash
ssh-keygen # ssh-keygen -t ed25519 -C "you@example.com"
```

For beginners, pressing **Enter** to accept the default path is fine.

Your key pair normally contains:

```text
Private key
~/.ssh/id_ed25519

Public key
~/.ssh/id_ed25519.pub
```

Important:

```text
PRIVATE KEY → stays on your computer
PUBLIC KEY  → can be added to GitHub
```

Never share your private key.

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output.

On GitHub:

1. Open **Settings**
2. Go to **SSH and GPG keys**
3. Choose **New SSH key**
4. Paste your public key
5. Save it

Test:

```bash
ssh -T git@github.com
```

You may be asked whether you trust the host the first time.

Type:

```text
yes
```

A successful authentication message should appear.

---

# 12. Change an Existing Remote from HTTPS to SSH

Check your current remote:

```bash
git remote -v
```

An HTTPS remote looks similar to:

```text
https://github.com/YOUR_USERNAME/day2-github-practice.git
```

Change it to SSH:

```bash
git remote set-url origin git@github.com:YOUR_USERNAME/day2-github-practice.git
```

Check:

```bash
git remote -v
```

Now it should look similar to:

```text
git@github.com:YOUR_USERNAME/day2-github-practice.git
```

Test it with:

```bash
git fetch
```

---

# 13. Option B — HTTPS Authentication

With HTTPS, GitHub does not accept your normal account password for Git operations.

Typical options include:

- Git Credential Manager
- GitHub CLI
- Personal Access Token (PAT)

If you create a PAT, prefer a **fine-grained token** when appropriate.

Important:

> Never write a real token inside your source code.

> Never commit a token to Git.

Treat tokens like passwords.

---

# 14. `.gitignore` — Ignore Files Git Should Not Track

Return to your main practice repository or continue in the clone.

Create some sample files:

```bash
echo "DEMO_PASSWORD=12345" > .env
echo "application log" > app.log
```

Check:

```bash
git status
```

Both may appear as untracked.

Create `.gitignore`:

```bash
touch .gitignore
```

Add rules:

```bash
echo ".env" >> .gitignore
echo "*.log" >> .gitignore
```

Check:

```bash
cat .gitignore
```

Expected:

```text
.env
*.log
```

Now:

```bash
git status
```

`.env` and `app.log` should no longer appear as untracked.

Stage `.gitignore`:

```bash
git add .gitignore
```

Commit:

```bash
git commit -m "Add gitignore rules"
```

Push:

```bash
git push
```

---

# 15. DevOps `.gitignore` Examples

Common examples:

```gitignore
.env
*.log
node_modules/
target/
*.pem
*.key
```

For DevOps work, never intentionally commit real:

```text
passwords
API tokens
AWS credentials
private keys
secret .env files
```

---

# Practice 5 — Add More Ignore Rules

Create:

```bash
mkdir logs
echo "error" > logs/error.log
echo "temporary" > test.log
```

Check:

```bash
git status
```

Because `*.log` is already ignored, the log files should not appear.

Add another pattern:

```bash
echo "*.tmp" >> .gitignore
```

Create:

```bash
echo "temporary data" > cache.tmp
```

Check:

```bash
git status
```

`cache.tmp` should also be ignored.

Commit the `.gitignore` update:

```bash
git add .gitignore
git commit -m "Update gitignore rules"
git push
```

---

# 16. Important Case — File Was Already Tracked

Create a **fake practice file**:

```bash
echo "DEMO_ONLY=12345" > tracked-secret.txt
```

Add and commit it:

```bash
git add tracked-secret.txt
git commit -m "Add demo tracked file"
```

Now add it to `.gitignore`:

```bash
echo "tracked-secret.txt" >> .gitignore
```

Run:

```bash
git status
```

Git still knows about `tracked-secret.txt`.

Why?

Because `.gitignore` mainly prevents **untracked** files from being added.

If a file is already tracked, stop tracking it with:

```bash
git rm --cached tracked-secret.txt
```

Now stage `.gitignore`:

```bash
git add .gitignore
```

Commit:

```bash
git commit -m "Stop tracking demo secret file"
```

Push:

```bash
git push
```

Important:

> `git rm --cached` stops tracking the current file, but it does not erase older copies from Git history.

If a **real credential** was committed, the credential should also be rotated/revoked.

---

# 17. Watch, Star and Fork

These actions are mainly done in the GitHub web interface.

Open a public repository.

## Watch

Use **Watch** when you want repository notifications.

Think:

```text
Watch = notify me
```

## Star

Use **Star** when you want to bookmark or show interest in a repository.

Think:

```text
Star = I want to find this repository again
```

## Fork

Use **Fork** to create your own GitHub copy of another repository.

Think:

```text
Original Repository
       ↓
      Fork
       ↓
Your GitHub Account
       ↓
Your Copy
```

Fork is especially useful when you want to contribute to a project but do not have direct write access.

---

# Practice 6 — Fork a Repository

Use a simple public repository provided by your instructor.

On GitHub:

1. Open the repository
2. Click **Fork**
3. Create the fork in your own GitHub account
4. Open your fork
5. Click **Code**
6. Copy its URL

Clone your fork:

```bash
git clone <YOUR_FORK_URL>
```

Enter the project:

```bash
cd <repository-name>
```

Check:

```bash
git remote -v
```

The `origin` remote should point to **your fork**.

---

# 18. Create a GitHub Issue

Go to your `day2-github-practice` repository.

Open:

```text
Issues
```

Create a new issue.

Use a very simple title:

```text
Add a project description
```

Description:

```text
The README needs a short project description.
```

Create the issue.

Suppose GitHub gives it:

```text
Issue #1
```

Your number may be different.

If available, also practice:

- Assigning the issue to yourself
- Adding a label
- Writing a comment

---

# 19. Create a Feature Branch for the Issue

Before starting new work, make sure local `main` is updated:

```bash
git switch main
git pull
```

Create a feature branch:

```bash
git switch -c feature-readme
```

Check:

```bash
git branch
```

Expected:

```text
* feature-readme
  main
```

Make a small change:

```bash
echo "" >> README.md
echo "This repository is used for Git and GitHub Day 2 practice." >> README.md
```

Check:

```bash
git diff
```

Stage:

```bash
git add README.md
```

Commit:

```bash
git commit -m "Add project description"
```

Push the branch:

```bash
git push -u origin feature-readme
```

---

# 20. Understand the Feature Branch Workflow

We now have:

```text
main
 |
 A --- B
       \
        C
        |
 feature-readme
```

We did **not** push the change directly to `main`.

Instead:

```text
main
  |
  +---- feature-readme
            |
            | commit
            |
            v
         git push
            |
            v
          GitHub
```

The next step is a Pull Request.

---

# 21. Create a Pull Request

Open the repository on GitHub.

GitHub will usually show a button similar to:

```text
Compare & pull request
```

Open it.

Check:

```text
base: main
```

and:

```text
compare: feature-readme
```

Use a clear title:

```text
Add project description
```

In the description, write:

```text
Adds a short project description to README.

Closes #1
```

Replace `#1` with your actual issue number.

Create the Pull Request.

---

# 22. Understand the Pull Request

A Pull Request means:

> "Please review my branch and consider merging it into another branch."

Our workflow:

```text
feature-readme
      |
      | push
      v
    GitHub
      |
      v
 Pull Request
      |
      v
    Review
      |
      v
    Merge
      |
      v
     main
```

Before merging, inspect:

- **Conversation**
- **Commits**
- **Files changed**

---

# Practice 7 — Read the PR Before Merging

On the Pull Request:

1. Open **Files changed**
2. Find the new README lines
3. Confirm that only the expected file changed
4. Return to **Conversation**

Ask:

> Did we accidentally change anything else?

If not, continue.

---

# 23. Code Review Practice

If students are working in pairs:

### Student A

Creates the Pull Request.

### Student B

Opens Student A's Pull Request and reviews the change.

Possible review actions:

```text
COMMENT
```

Ask a question or leave feedback.

```text
APPROVE
```

The change looks good.

```text
REQUEST CHANGES
```

The author should update something before merge.

For a very simple review comment, write:

```text
Looks clear. Good description.
```

Then approve the Pull Request.

If you are practicing alone:

- inspect **Files changed**
- read the diff
- continue to merge if your repository settings allow it

GitHub generally does not treat approving your own PR as a normal independent review.

---

# 24. Merge the Pull Request

After review, choose:

```text
Merge pull request
```

Confirm the merge.

The changes are now on remote `main`.

If the PR description contained:

```text
Closes #1
```

the linked issue may close automatically after the PR is merged.

---

# 25. Update Your Local `main` After the Merge

Your browser shows the new version of `main`, but your local repository may still be on the feature branch.

Run:

```bash
git switch main
```

Then:

```bash
git pull
```

Check:

```bash
cat README.md
```

The merged project description should now appear locally.

Check history:

```bash
git log --oneline --graph --decorate --all -10
```

---

# 26. Delete the Finished Local Feature Branch

After the feature has been merged:

```bash
git branch -d feature-readme
```

Check:

```bash
git branch
```

You should now mainly have:

```text
* main
```

If GitHub offers to delete the remote feature branch after merge, you can also do that from the PR page.

---

# Practice 8 — Repeat the Full Feature Workflow

Create another Issue:

```text
Add contact information
```

Then complete this flow yourself:

```text
Issue
  ↓
git switch main
  ↓
git pull
  ↓
git switch -c feature-contact
  ↓
Modify README.md
  ↓
git add README.md
  ↓
git commit
  ↓
git push -u origin feature-contact
  ↓
Open Pull Request
  ↓
Review
  ↓
Merge
  ↓
git switch main
  ↓
git pull
```

This time, try to use the commands without copying every step.

---

# 27. Remote Collaboration Conflict — Simple Practice

Now we simulate two developers using **two local clones**.

We can do this even with one GitHub account.

First, make sure `main` contains a simple file.

From your main repository:

```bash
git switch main
git pull
echo "MESSAGE=Original" > message.txt
git add message.txt
git commit -m "Add message file"
git push
```

Now move outside the repository:

```bash
cd ..
```

Clone it twice:

```bash
git clone <YOUR_REPO_URL> student-a
git clone <YOUR_REPO_URL> student-b
```

We now have:

```text
student-a
student-b
```

They simulate two developers.

---

# 28. Student A Makes the First Change

Enter Student A's copy:

```bash
cd student-a
```

Change the file:

```bash
echo "MESSAGE=Hello from A" > message.txt
```

Check:

```bash
git diff
```

Commit:

```bash
git add message.txt
git commit -m "Update message from A"
```

Push:

```bash
git push
```

Student A's version is now on GitHub.

---

# 29. Student B Changes the Same Line

Move to Student B:

```bash
cd ../student-b
```

Student B still has the old version.

Check:

```bash
cat message.txt
```

It should show:

```text
MESSAGE=Original
```

Student B changes the same line:

```bash
echo "MESSAGE=Hello from B" > message.txt
```

Commit:

```bash
git add message.txt
git commit -m "Update message from B"
```

Now try:

```bash
git push
```

The push should be rejected because GitHub has commits that Student B does not have.

You may see a message mentioning:

```text
rejected
```

or:

```text
fetch first
```

This is normal.

---

# 30. Student B Pulls the Remote Changes

Run:

```bash
git pull --no-rebase origin main
```

Because both Student A and Student B changed the same line differently, Git may report:

```text
CONFLICT (content): Merge conflict in message.txt
```

Check:

```bash
git status
```

Open:

```bash
cat message.txt
```

You may see:

```text
<<<<<<< HEAD
MESSAGE=Hello from B
=======
MESSAGE=Hello from A
>>>>>>> origin/main
```

---

# 31. Resolve the Remote Conflict

Choose a final value.

For example:

```text
MESSAGE=Hello from A and B
```

Replace the conflict content:

```bash
echo "MESSAGE=Hello from A and B" > message.txt
```

Check:

```bash
cat message.txt
```

Stage:

```bash
git add message.txt
```

Commit:

```bash
git commit -m "Resolve collaboration conflict"
```

Push:

```bash
git push
```

Now GitHub contains the resolved version.

---

# Practice 9 — Explain What Happened

Students should be able to explain:

```text
Student A
   ↓
commit
   ↓
push
   ↓
GitHub changed
```

Meanwhile:

```text
Student B
   ↓
different commit
   ↓
push rejected
   ↓
pull
   ↓
conflict
   ↓
resolve
   ↓
commit
   ↓
push
```

The important lesson:

> Before starting shared work, update your branch.

A common habit is:

```bash
git switch main
git pull
```

before creating a new feature branch.

---

# 32. Branch Protection / Rulesets —(Optional)

In professional teams, developers often do **not** push directly to `main`.

A common workflow is:

```text
Feature Branch
      ↓
Pull Request
      ↓
Review
      ↓
Checks
      ↓
main
```

GitHub can protect `main` using branch protection or rulesets.

Depending on the repository and GitHub plan/interface, open repository settings and look for:

```text
Settings
→ Rules / Rulesets
```

or branch protection settings.

Typical rules include:

- Require a Pull Request
- Require approving reviews
- Require status checks
- Block force pushes
- Block branch deletion

> **Classroom recommendation:** Treat this as an instructor-led demo first. Strict rules can prevent students from completing later exercises if configured incorrectly.

---

# 33. Why Protect `main`?

Without protection:

```text
Developer
   ↓
git push
   ↓
main
```

With a team workflow:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
Code Review
   ↓
Approved
   ↓
main
```

This is much safer for shared repositories.

It also prepares us for CI/CD, where automated checks can run before code is merged.

---

# 34. Final Day 2 Challenge

Complete the following workflow using your GitHub repository.

## Task

Create an Issue:

```text
Add environment documentation
```

Then complete:

```text
1. Update local main

2. Create a feature branch

3. Add a small section to README.md

4. Check the change with git diff

5. Stage and commit

6. Push the feature branch

7. Create a Pull Request

8. Link the Issue using:
   Closes #<issue-number>

9. Review the Files changed tab

10. Have a partner review if available

11. Merge the PR

12. Switch local repository back to main

13. Pull the merged change

14. Delete the finished local feature branch

15. Check the final history
```

Useful commands:

```bash
git switch main
git pull

git switch -c feature-environment

# modify README.md

git diff
git add README.md
git commit -m "Add environment documentation"

git push -u origin feature-environment

# Create and merge PR on GitHub

git switch main
git pull
git branch -d feature-environment

git log --oneline --graph --decorate --all -10
```

---

# 35. Day 2 Complete Mental Model

```text
                   GITHUB
              Remote Repository
                   ▲      │
                   │      │
            git push      │ git fetch / git pull
                   │      ▼
              Local Repository
                   ▲
                   │ git commit
                   │
               Staging Area
                   ▲
                   │ git add
                   │
            Working Directory
```

Team workflow:

```text
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
Code Review
  ↓
Merge
  ↓
main
```

---

# 36. Day 2 Command Summary

## Remote Repository

```bash
git remote
git remote -v
git remote add origin <repo-url>
git remote set-url origin <new-url>
```

## Push

```bash
git push -u origin main
git push
```

## Clone

```bash
git clone <repo-url>
```

## Fetch

```bash
git fetch
git fetch origin
```

## Pull

```bash
git pull
```

For the explicit merge-based conflict exercise:

```bash
git pull --no-rebase origin main
```

## Branch Workflow

```bash
git switch main
git pull
git switch -c feature-name

git add .
git commit -m "message"

git push -u origin feature-name
```

## `.gitignore`

Example:

```gitignore
.env
*.log
node_modules/
target/
*.pem
*.key
```

Already tracked file:

```bash
git rm --cached <file>
```

## SSH

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
ssh -T git@github.com
```

---

# 37. Quick Troubleshooting

## Error: `remote origin already exists`

Check:

```bash
git remote -v
```

If the URL is wrong:

```bash
git remote set-url origin <correct-url>
```

---

## Error: push rejected / fetch first

Your local branch is behind the remote.

Usually:

```bash
git pull
```

Then resolve any conflict if necessary.

Then:

```bash
git push
```

---

## Error: authentication failed

Check whether you are using:

```text
HTTPS
```

or:

```text
SSH
```

For SSH:

```bash
ssh -T git@github.com
```

For HTTPS, use an approved credential method such as Git Credential Manager, GitHub CLI, or a PAT rather than your GitHub account password.

---

## `.gitignore` is not ignoring my file

The file may already be tracked.

Check:

```bash
git status
```

If it was already committed:

```bash
git rm --cached <file>
```

Then commit the change.

---

## I do not know what is happening

Start with:

```bash
git status
```

Then check:

```bash
git branch
git remote -v
git log --oneline --decorate -5
```

These commands usually tell you:

- where you are
- what changed
- what branch you are on
- what remote you are connected to

---

# Day 2 Key Takeaway

By the end of Day 2, you should understand this complete flow:

```text
LOCAL GIT
   ↓
REMOTE / ORIGIN
   ↓
PUSH • CLONE • FETCH • PULL
   ↓
FEATURE BRANCH
   ↓
PULL REQUEST
   ↓
CODE REVIEW
   ↓
MERGE
   ↓
SHARED main BRANCH
```

> **Day 1 taught us how Git works locally.**

> **Day 2 teaches us how Git becomes a team workflow through GitHub.**

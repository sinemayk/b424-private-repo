# Hands-on Linux-06: Managing Users and Groups

## EC2 Lab Environment

Use the following EC2 configuration for this hands-on:

```text
AMI: Amazon Linux 2023
Architecture: x86_64
Instance Type: t3.micro
Storage: 8 GB gp3
Public IPv4 Address: Enabled
Security Group:
  SSH (22) -> My IP
```

> `t3.micro` is sufficient for this lesson because the lab focuses on Linux user and group management, sudo, password management, and SSH access. No larger instance type is required.

## Learning Outcomes

By the end of this hands-on training, students will be able to:

- Explain users and groups in Linux.
- Use `sudo` safely for administrative tasks.
- Inspect the current user, hostname, logged-in users, and user/group IDs.
- Read and interpret user information from `/etc/passwd`.
- Create, modify, and delete Linux users.
- Set passwords and inspect password-aging information.
- Manage Linux groups and supplementary group membership.
- Compare administrator groups on Amazon Linux and Ubuntu.
- Configure a new EC2 user for SSH key-based authentication.
- Apply Linux user/group management in a practical DevOps scenario.

## Outline

- **Section 1** - Sudo and Privilege Management
- **Section 2** - Basic User and System Commands
- **Section 3** - Switching Users
- **Section 4** - Understanding `/etc/passwd`
- **Section 5** - User Management
- **Section 6** - User Passwords and Password Aging
- **Section 7** - Group Management
- **Section 8** - Amazon Linux vs Ubuntu Administrator Groups
- **Section 9** - Creating an EC2 User with SSH Access
- **Section 10** - DevOps Team Scenario
- **Section 11** - Cleanup and Validation

---

# Section 1 - Sudo and Privilege Management

The `sudo` command runs a single command with elevated privileges.

> **Best practice:** Prefer `sudo command` for individual administrative operations instead of working permanently as `root`.

## 1.1 Compare a normal command with sudo

```bash
dnf check-update
sudo dnf check-update

dnf update
sudo dnf update
```

## 1.2 Try creating a directory without and with sudo

```bash
cd /
mkdir testdir
```

The first command should fail because a normal user does not normally have permission to create directories directly under `/`.

Now use:

```bash
sudo mkdir /testdir
ls -ld /testdir
```

Clean it up:

```bash
sudo rmdir /testdir
```

## 1.3 Open a root login shell only when needed

```bash
sudo -i
whoami
pwd
exit
```

You can also inspect the command prompt:

- `$` usually indicates a normal user.
- `#` usually indicates `root`.

---

# Section 2 - Basic User and System Commands

## 2.1 `whoami`

Display the current effective user:

```bash
whoami
```

## 2.2 `hostname`

Display the system hostname:

```bash
hostname
```

Display hostname information:

```bash
hostnamectl
```

Display the IP address associated with the hostname:

```bash
hostname -i
```

## 2.3 `whatis`

Display a one-line description from the manual database:

```bash
whatis passwd
whatis useradd
whatis groups

sudo mandb   # rebuilding the manual database, On Amazon Linux 2023, whatis comes from the man-db system.
```

## 2.4 `apropos`

Search command descriptions by keyword:

```bash
apropos password
apropos user
apropos group
```

## 2.5 `who`

Display users currently logged in:

```bash
who
```

Open a second SSH session to the same EC2 instance and run:

```bash
who
```

Compare the output.

## 2.6 `w`

Display logged-in users and what they are doing:

```bash
w
```

Compare:

```bash
who
w
```

## 2.7 `id`

Display UID, primary GID, and supplementary groups:

```bash
id
id root
id ec2-user
```

---

# Section 3 - Switching Users

Create a temporary lab user:

```bash
sudo useradd -m user1
```

## 3.1 `su`

Switch user without starting a login shell:

```bash
sudo su user1
whoami
pwd
exit
```

## 3.2 `su -`

Switch user and start that user's login environment:

```bash
sudo su - user1
whoami
pwd
exit
```

Compare the working directories from the two examples.

## 3.3 `sudo -i`

Start a root login shell:

```bash
sudo -i
whoami
pwd
exit
```

---

# Section 4 - Understanding `/etc/passwd`

Linux local user account information is stored in `/etc/passwd`.

Display the last few entries:

```bash
tail -5 /etc/passwd
```

Display only the `ec2-user` entry:

```bash
grep '^ec2-user:' /etc/passwd
```

You can also query account databases with `getent`:

```bash
getent passwd ec2-user
```

Example format:

```text
ec2-user:x:1000:1000:EC2 User:/home/ec2-user:/bin/bash
```

The fields are:

```text
username : password-placeholder : UID : GID : description : home-directory : login-shell
```

## Practice

Run:

```bash
getent passwd root
getent passwd ec2-user
getent passwd user1
```

For each user, identify:

- Username
- UID
- Primary GID
- Description
- Home directory
- Login shell

---

# Section 5 - User Management

## 5.1 Create users with `useradd`

Create a user:

```bash
sudo useradd user2
```

Check the account:

```bash
getent passwd user2
```

## 5.2 Create a home directory with `-m`

```bash
sudo useradd -m user3
ls -ld /home/user3
```

## 5.3 Use a custom home directory with `-d`

```bash
sudo useradd -m -d /home/user4home user4
ls -ld /home/user4home
getent passwd user4
```

## 5.4 Add a description with `-c`

```bash
sudo useradd -m -c "Cloud Developer" user5
getent passwd user5
```

## 5.5 Inspect user defaults

Instead of changing system-wide defaults during the lab, inspect them:

```bash
cat /etc/login.defs
```

Also inspect:

```bash
useradd -D # display the default settings used when creating new users.
```

> Do not change global account defaults during this exercise. Use command-line options such as `-m`, `-d`, and `-c` for individual users.

## 5.6 Modify a user with `usermod`

Change the user's description:

```bash
sudo usermod -c "AWS Solutions Architect" user5
getent passwd user5
```

Rename a lab user:

```bash
sudo usermod -l clouduser user2  # user2  →  clouduser , -l clouduser   → set the new login name to clouduser
getent passwd clouduser
```

> Renaming a username does not automatically rename the user's home directory. User and home-directory changes should be planned carefully in production.

## 5.7 Delete users

Delete a user but leave its home directory:

```bash
sudo userdel user3
ls -ld /home/user3
```

Delete another user together with its home directory:

```bash
sudo userdel -r user1
```

Check:

```bash
getent passwd user1
ls -ld /home/user1
```

---

# Section 6 - User Passwords and Password Aging

## 6.1 Set a password

Create a lab user:

```bash
sudo useradd -m user8
```

Set its password:

```bash
sudo passwd user8
```

## 6.2 Inspect `/etc/shadow`

User password hashes are stored in `/etc/shadow`.

Do not print the entire file unnecessarily. Query only the lab user:

```bash
sudo grep '^user8:' /etc/shadow
```

The password field contains a cryptographic password hash, not the original password.

## 6.3 Inspect password-aging settings

```bash
sudo chage -l user8
```

## 6.4 Set a maximum password age
chage = change/view password aging information

```bash
sudo chage -M 90 user8
```

## 6.5 Set a minimum password age

```bash
sudo chage -m 1 user8
```

## 6.6 Set a warning period

```bash
sudo chage -W 7 user8
```

## 6.7 Force a password change at next login

```bash
sudo chage -d 0 user8
```

Verify:

```bash
sudo chage -l user8
```

### `chage` Options Used

| Option | Meaning |
|---|---|
| `-l` | List password-aging information |
| `-M` | Maximum password age |
| `-m` | Minimum password age |
| `-W` | Password-expiration warning period |
| `-d 0` | Force password change at next login |

---

# Section 7 - Group Management

Linux users can belong to one primary group and multiple supplementary groups.

## 7.1 Inspect groups

```bash
groups
groups ec2-user
id ec2-user
```

Inspect the group database:

```bash
tail -10 /etc/group
```

Query a specific group:

```bash
getent group wheel
```

## 7.2 Create groups

```bash
sudo groupadd developers
sudo groupadd cloud
sudo groupadd aws
```

Verify:

```bash
getent group developers
getent group cloud
getent group aws
```

## 7.3 Create a lab user

```bash
sudo useradd -m devuser
```

## 7.4 Add a user to supplementary groups

Use `-aG`:

```bash
sudo usermod -aG developers devuser
sudo usermod -aG cloud,aws devuser
```

Check:

```bash
groups devuser
id devuser
```

### Important

`-a` means **append**.

`-G` specifies the supplementary group list.

Without `-a`, using `usermod -G` replaces the user's existing supplementary group list.

Demonstrate this only with the safe lab user:

```bash
sudo usermod -G aws devuser
groups devuser
```

Add the groups back:

```bash
sudo usermod -aG developers,cloud devuser
groups devuser
```

> Do not use the main EC2 administrator account to demonstrate removal of supplementary groups.

## 7.5 Rename a group

```bash
sudo groupmod -n engineering developers
```

Verify:

```bash
getent group engineering
```

## 7.6 Delete a group

```bash
sudo groupdel cloud
```

Verify:

```bash
getent group cloud
```

If no result is returned, the group no longer exists.

## 7.7 Add and remove users with `gpasswd`

Add the user to the `aws` group:

```bash
sudo gpasswd -a devuser aws
```

Verify:

```bash
groups devuser
```

Remove the user:

```bash
sudo gpasswd -d devuser aws
```

Verify:

```bash
groups devuser
```

## 7.8 Activate a new group in the current shell

After changing your own group membership, you normally need a new login session.

For a lab user, you can demonstrate:

```bash
sudo usermod -aG engineering devuser
sudo su - devuser
groups
newgrp engineering
id
exit
```

---

# Section 8 - Amazon Linux vs Ubuntu Administrator Groups

The administrative group is different on common Linux distributions.

## Amazon Linux

```bash
sudo useradd -m developer
sudo usermod -aG wheel developer
groups developer
```

Administrative group:

```text
wheel
```

## Ubuntu

On Ubuntu, the typical administrative group is:

```bash
sudo usermod -aG sudo developer
```

Administrative group:

```text
sudo
```

### Comparison

| Distribution | Common admin group |
|---|---|
| Amazon Linux | `wheel` |
| Ubuntu | `sudo` |

---

# Section 9 - Creating an EC2 User with SSH Access

## Goal

Create a new Linux user named `radwin`, configure SSH key-based authentication, and optionally grant administrative privileges.

## Step 1 - Connect to the EC2 instance

```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

## Step 2 - Create the user

```bash
sudo useradd -m radwin
```

Verify:

```bash
id radwin
ls -ld /home/radwin
```

## Step 3 - Optional: set a password

```bash
sudo passwd radwin
```

For normal EC2 administration, SSH key-based authentication is preferred over password login.

## Step 4 - Optional: give sudo privileges

On Amazon Linux:

```bash
sudo usermod -aG wheel radwin
```

Verify:

```bash
groups radwin
```

## Step 5 - Configure SSH access

Create the `.ssh` directory:

```bash
sudo mkdir -p /home/radwin/.ssh
```

Copy the existing authorized public key:

```bash
sudo cp /home/ec2-user/.ssh/authorized_keys /home/radwin/.ssh/
```

## Step 6 - Fix ownership and permissions

```bash
sudo chown -R radwin:radwin /home/radwin/.ssh
sudo chmod 700 /home/radwin/.ssh
sudo chmod 600 /home/radwin/.ssh/authorized_keys
```

Verify:

```bash
sudo ls -ld /home/radwin/.ssh
sudo ls -l /home/radwin/.ssh/authorized_keys
```

Expected permission pattern:

```text
drwx------  .ssh
-rw-------  authorized_keys
```

## Step 7 - Test remote login

From your local machine:

```bash
ssh -i your-key.pem radwin@your-ec2-public-ip
```

Then verify:

```bash
whoami
pwd
id
```

## Step 8 - Inspect the effective SSH configuration

Before editing SSH configuration, inspect it:

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication'
```

You should normally see public-key authentication enabled.

If you intentionally modify `/etc/ssh/sshd_config`, validate the configuration before restarting SSH:

```bash
sudo sshd -t
```

If no error is returned, restart the service:

```bash
sudo systemctl restart sshd
```

> Keep your original SSH session open until you have confirmed that a second SSH session can connect successfully.

---

# Section 10 - DevOps Team Scenario

## Scenario

Your company has three teams:

- Developers
- Cloud administrators
- CI/CD automation

Create a Linux user/group structure for the team.

## Tasks

1. Create these groups:

```text
developers
cloudadmins
cicd
```

2. Create these users:

```text
alice
bob
jenkins
```

3. Add:

- `alice` to `developers`
- `bob` to `developers` and `cloudadmins`
- `jenkins` to `cicd`

4. Verify all group memberships.

5. Give `bob` administrative access on Amazon Linux using the appropriate group.

6. Create a home directory for every user.

7. Display each user's `/etc/passwd` entry.

8. Display `bob`'s UID, GID, and supplementary groups.

9. Configure password aging for `alice`:

- Maximum age: 90 days
- Warning period: 7 days

10. Create an SSH-ready user named `deploy` by repeating the SSH key-authentication workflow from Section 9.

## Validation Commands

Use commands such as:

```bash
id alice
id bob
id jenkins
groups alice
groups bob
groups jenkins
getent passwd alice
getent passwd bob
getent passwd jenkins
getent group developers
getent group cloudadmins
getent group cicd
sudo chage -l alice
```

### Done Condition

The exercise is complete when:

- All users exist.
- All groups exist.
- Memberships match the scenario.
- `bob` has the appropriate Amazon Linux administrative group membership.
- Password-aging settings for `alice` are correct.
- `deploy` can log in remotely with SSH key-based authentication.

---

# Section 11 - Cleanup and Validation

Remove the temporary lab users that you no longer need:

```bash
sudo userdel -r user4
sudo userdel -r user5
sudo userdel -r user8
sudo userdel -r devuser
sudo userdel -r developer
sudo userdel -r radwin
```

Remove temporary groups if they are no longer required:

```bash
sudo groupdel engineering
sudo groupdel aws
```

> If a user or group has already been removed during the lab, the corresponding cleanup command may return an error. That is expected.

Final checks:

```bash
getent passwd user4
getent passwd user5
getent passwd user8
getent passwd devuser
getent passwd radwin

getent group engineering
getent group aws
```

If no account/group information is returned for the removed objects, cleanup is complete.

---

# Final Review

By the end of this hands-on, you should be comfortable with:

```bash
sudo
whoami
hostname
hostnamectl
whatis
apropos
who
w
id
su
getent
useradd
usermod
userdel
passwd
chage
groups
groupadd
groupmod
groupdel
gpasswd
newgrp
```

You should also understand:

- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- Primary and supplementary groups
- `wheel` vs `sudo`
- SSH key-based authentication on EC2
- Ownership and permission requirements for `.ssh`
- Why `usermod -aG` is safer than replacing supplementary groups accidentally

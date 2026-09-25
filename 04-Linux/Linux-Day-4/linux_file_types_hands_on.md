# Hands-on: Linux File Types

This hands-on exercise helps students identify and work with common Linux file types.

In Linux, the **first character** shown by `ls -l` tells us the file type.

---

## Quick Reference

| First Character | File Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `p` | Named pipe |
| `s` | Socket |

---

# 1. Create the Practice Directory

```bash
mkdir -p ~/linux-day4/file-types
cd ~/linux-day4/file-types
```

Check your current location:

```bash
pwd
```

---

# 2. Regular File `-`

Create a normal file:

```bash
touch regular.txt
```

Check it:

```bash
ls -l regular.txt
```

Example:

```text
-rw-r--r-- 1 user user 0 Sep 24 10:00 regular.txt
```

The first character is:

```text
-
```

Meaning:

```text
Regular file
```

---

# 3. Directory `d`

Create a directory:

```bash
mkdir mydir
```

Check it:

```bash
ls -ld mydir
```

Example:

```text
drwxr-xr-x 2 user user 4096 Sep 24 10:01 mydir
```

The first character is:

```text
d
```

Meaning:

```text
Directory
```

---

# 4. Symbolic Link `l`

Create a file:

```bash
echo "Hello Linux" > original.txt
```

Create a symbolic link:

```bash
ln -s original.txt shortcut.txt
```

Check it:

```bash
ls -l
```

Example:

```text
lrwxrwxrwx 1 user user 12 Sep 24 10:02 shortcut.txt -> original.txt
```

The first character is:

```text
l
```

Meaning:

```text
Symbolic link
```

Test the link:

```bash
cat shortcut.txt
```

Output:

```text
Hello Linux
```

Now check the target:

```bash
ls -l shortcut.txt
```

---

# 5. Hard Link

Create a hard link:

```bash
ln original.txt hardlink.txt
```

Check the inode numbers:

```bash
ls -li original.txt hardlink.txt
```

Example:

```text
123456 -rw-r--r-- 2 user user 12 Sep 24 10:03 original.txt
123456 -rw-r--r-- 2 user user 12 Sep 24 10:03 hardlink.txt
```

Notice:

```text
Both files have the same inode number.
```

Compare:

```text
Symbolic link -> points to a pathname
Hard link     -> points to the same inode
```

Try:

```bash
cat hardlink.txt
```

---

# 6. Character Device `c`

Do not create device files manually in this beginner lab.

Instead, inspect an existing character device:

```bash
ls -l /dev/null
```

Example:

```text
crw-rw-rw- 1 root root 1, 3 Sep 24 10:00 /dev/null
```

The first character is:

```text
c
```

Meaning:

```text
Character device
```

Character devices handle data as a stream of bytes or characters.

Common examples:

```text
/dev/null
/dev/tty
/dev/random
```

Try:

```bash
echo "Hello" > /dev/null
```

Nothing is displayed because `/dev/null` discards the data.

---

# 7. Block Device `b`

Block devices represent storage devices such as disks and volumes.

List block devices:

```bash
lsblk
```

On many EC2 instances, you may also see NVMe devices:

```bash
ls -l /dev/nvme*
```

Example:

```text
brw-rw---- 1 root disk 259, 0 Sep 24 10:00 /dev/nvme0n1
```

The first character is:

```text
b
```

Meaning:

```text
Block device
```

Common examples:

```text
SSD
HDD
EBS volume
```

In AWS, an attached **EBS volume** appears in Linux as a block device.

---

# 8. Named Pipe `p`

Create a named pipe:

```bash
mkfifo mypipe
```

Check it:

```bash
ls -l mypipe
```

Example:

```text
prw-r--r-- 1 user user 0 Sep 24 10:04 mypipe
```

The first character is:

```text
p
```

Meaning:

```text
Named pipe / FIFO
```

## Live Demo

Open two terminals.

### Terminal 1

```bash
cat mypipe
```

The command waits for data.

### Terminal 2

```bash
echo "Hello through the pipe" > mypipe
```

Terminal 1 displays:

```text
Hello through the pipe
```

This demonstrates simple process-to-process communication.

---

# 9. Socket `s`

Sockets are commonly used for communication between processes.

Find some existing socket files:

```bash
find /run -type s 2>/dev/null | head
```

Choose one path from the output and inspect it:

```bash
ls -l /run/<socket-path>
```

You may see something similar to:

```text
srwxr-xr-x 1 root root 0 Sep 24 10:00 example.sock
```

The first character is:

```text
s
```

Meaning:

```text
Socket
```

A socket allows processes to communicate with each other.

---

# 10. Compare the File Types

Create or inspect the following:

```bash
touch regular.txt
mkdir demo-dir
ln -s regular.txt demo-link
mkfifo demo-pipe
```

Now run:

```bash
ls -l
```

Identify the first character of each item.

Example:

```text
-rw-r--r--   regular file
drwxr-xr-x   directory
lrwxrwxrwx   symbolic link
prw-r--r--   named pipe
```

For device files:

```bash
ls -l /dev/null
lsblk
```

For socket files:

```bash
find /run -type s 2>/dev/null | head
```

---

# 11. Student Practice

Complete the following tasks.

1. Create a regular file called `app.log`.
2. Create a directory called `logs`.
3. Create a symbolic link called `latest.log` pointing to `app.log`.
4. Create a hard link called `backup.log` pointing to the same inode as `app.log`.
5. Create a named pipe called `events.pipe`.
6. Use `ls -l` to identify each file type.
7. Use `ls -li` to compare the inode numbers of `app.log` and `backup.log`.
8. Inspect `/dev/null`.
9. Run `lsblk` and identify the main storage device.
10. Find at least one socket under `/run`.

---

# 12. Final Review

Use:

```bash
ls -l
```

Remember:

```text
-  -> regular file
d  -> directory
l  -> symbolic link
c  -> character device
b  -> block device
p  -> named pipe
s  -> socket
```

A **hard link** does not have a separate file-type character because it is another directory entry pointing to the same inode as a regular file.

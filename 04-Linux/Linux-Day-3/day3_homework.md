# Day 3 Homework — Working With Files & Search

## Task 1 — Create the Environment

Create this structure:

```text
~/day3-homework/
├── config/
├── logs/
└── docs/
```

Use `mkdir -p`.

---

## Task 2 — Create and Edit a Config File

Create:

```text
~/day3-homework/config/app.conf
```

Use Vim or Nano and add:

```text
APP_NAME=demo
PORT=8080
LOG_LEVEL=INFO
DEBUG=false
```

Then change:

```text
PORT=9090
LOG_LEVEL=DEBUG
```

Save and exit.

---

## Task 3 — Create a Log File

Create:

```text
~/day3-homework/logs/app.log
```

Add at least **10 lines** containing:

- `INFO`
- `WARN`
- `ERROR`

Include at least **2 WARN** and **2 ERROR** entries.

---

## Task 4 — Inspect the Log

Use commands to:

1. Show the first 5 lines.
2. Show the last 5 lines.
3. Open the file with `less`.
4. Search for `ERROR` inside `less`.

Use:

```bash
head
tail
less
```

---

## Task 5 — Search the Log

Practice:

```bash
grep "ERROR" app.log
grep -n "ERROR" app.log
grep -i "error" app.log
grep -c "ERROR" app.log
grep -C 2 "ERROR" app.log
```

Answer:

- On which lines did the errors occur?
- What happened before and after an error?

---

## Task 6 — Find Files

Create a few `.log`, `.txt`, and `.md` files.

Then use `find` to:

- Find all regular files.
- Find all directories.
- Find all `.log` files.
- Find `readme.md` ignoring uppercase/lowercase.

Use:

```bash
-type f
-type d
-name
-iname
```

---

## Task 7 — Live Log Monitoring

Open two terminals.

### Terminal 1

```bash
tail -f ~/day3-homework/logs/app.log
```

### Terminal 2

```bash
echo "INFO Application recovered" >> ~/day3-homework/logs/app.log
```

Confirm that the new line appears in Terminal 1.

Stop `tail -f` with:

```text
Ctrl+C
```

---

## Bonus

Explain the difference between:

```text
find vs grep
head vs tail
tail vs tail -f
cat vs less
> vs >>
-name vs -iname
```

## Goal

```text
Edit files → Inspect files → Follow logs → Find files → Search text
```

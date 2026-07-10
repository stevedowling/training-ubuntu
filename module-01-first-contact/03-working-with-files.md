# 3 · Working with files

> **You'll learn:** to create, view, copy, move, rename, and delete files and directories - safely, including when wildcards are involved.

## Why this matters

Everything you do on Linux ends up as file operations: saving work, installing software, editing configuration, cleaning up disk space. These half-dozen commands are the verbs of the command line - you'll use them hundreds of times a day, so the habits you form now (especially around `rm`) matter.

## The big picture

| I want to... | Command | Memory hook |
|---|---|---|
| Make a directory | `mkdir notes` | *make dir* |
| Make an empty file | `touch todo.txt` | touch it into existence |
| See a file's contents | `cat todo.txt` / `less big.log` | con*cat*enate / *less* is more |
| Copy | `cp a.txt b.txt` | *c*o*p*y |
| Move or rename | `mv a.txt notes/` | *m*o*v*e (rename = move to a new name) |
| Delete a file | `rm a.txt` | *r*e*m*ove - **no trash, no undo** |
| Delete an empty directory | `rmdir notes` | |
| Delete a directory and contents | `rm -r notes` | `-r` = recursive |

One session using most of them:

```console
$ mkdir -p course/exercises        # -p: create parents as needed, no error if exists
$ cd course/exercises
$ touch notes.txt
$ echo "hello linux" > notes.txt   # write a line into it (more in module 3)
$ cat notes.txt
hello linux
$ cp notes.txt backup.txt
$ mv backup.txt notes-v1.txt       # rename
$ rm notes-v1.txt
```

## Viewing files without an editor

`cat` dumps a whole file to the screen - fine for short ones. For anything long, use a **pager**:

```console
$ less /var/log/syslog     # scroll with arrows/PgUp/PgDn, / to search, q to quit
$ head -5 /etc/passwd      # first 5 lines
$ tail -20 /var/log/syslog # last 20 lines
$ tail -f /var/log/syslog  # ...and keep following as new lines arrive (Ctrl+C to stop)
```

`tail -f` on a log while you reproduce a problem is one of the oldest debugging tricks on Unix - you'll use it constantly from module 6 on.

> [!TIP]
> Inside `less`: `/error` searches forward for "error", `n` jumps to the next match, `G` goes to the end, `q` quits. Learn these four and log files stop being scary.

## Wildcards: talking about many files at once

The shell expands **glob patterns** into matching filenames *before* the command runs:

| Pattern | Matches |
|---|---|
| `*.txt` | every name ending in `.txt` |
| `report-?.md` | `report-1.md`, `report-A.md` - `?` is exactly one character |
| `photo-202[45]*` | names starting `photo-2024` or `photo-2025` |

```console
$ cp *.txt backups/        # copy all .txt files into backups/
$ ls -l /var/log/*.log     # long-list every .log file there
```

Because the *shell* does the expansion, `cp` never sees `*.txt` - it sees the actual list of files. That's why `echo *.txt` is a safe way to preview what a pattern will match before using it with something destructive.

## Deleting without regrets

`rm` deletes immediately and permanently: no trash can, no undo. Three habits keep you safe:

```console
$ echo *.log               # 1. preview what the glob matches
$ rm -i *.log              # 2. -i asks before each deletion
$ rm -rI old-project/      # 3. -I asks once before mass/recursive deletion
```

> [!WARNING]
> The classic catastrophe is a stray space: `rm -r tmp /*` (note the space) deletes `tmp` and then *everything under `/`*. Type destructive commands slowly, and preview globs with `echo` or `ls` first. On Ubuntu 26.04, `rm` is still the battle-tested GNU version.

<details>
<summary>🔍 Deep dive: why renaming and moving are the same command</summary>

A directory on Linux is just a table mapping names to file contents (inodes - module 2 covers them properly). `mv a.txt b.txt` doesn't touch the file's data; it rewrites one table entry. Moving within a filesystem is the same operation whether the new name is in this directory or another one, which is why one command does both, and why `mv` on a 50 GB file within one disk is instant. Moving *across* filesystems (say, to a USB stick) really does copy the data and delete the original - `mv` hides that difference from you.

</details>

## 🛠️ Try it

Build the workspace this course uses from here on, then reorganize it - all from the shell:

1. In your home directory, create `linux-course/exercises` and `linux-course/scratch` in one command.
2. In `scratch`, create files `note-1.txt`, `note-2.txt`, `note-3.txt`, and `readme.md` (one `touch` command can take several names).
3. Copy *only the .txt files* into `exercises` using a single glob.
4. In `exercises`, rename `note-3.txt` to `note-final.txt`.
5. Delete the whole `scratch` directory, using the preview-then-delete habit.
6. Save your identity card from lesson 1 as `linux-course/exercises/identity.txt` (e.g. `uname -a > identity.txt` is a start).

<details>
<summary>💡 Hint 1</summary>

Step 1: `mkdir -p` accepts multiple paths: `mkdir -p linux-course/exercises linux-course/scratch`. Step 3: run `echo *.txt` first to check the glob.

</details>

<details>
<summary>✅ Solution</summary>

```console
$ cd
$ mkdir -p linux-course/exercises linux-course/scratch
$ cd linux-course/scratch
$ touch note-1.txt note-2.txt note-3.txt readme.md
$ echo *.txt                          # preview: note-1.txt note-2.txt note-3.txt
$ cp *.txt ../exercises/
$ cd ../exercises
$ mv note-3.txt note-final.txt
$ ls ../scratch                       # look before deleting
$ rm -rI ../scratch
$ uname -a > identity.txt
$ ls
identity.txt  note-1.txt  note-2.txt  note-final.txt
```

</details>

## ✋ Checkpoint

1. Predict: after `touch a.txt b.txt && mv a.txt b.txt`, what does `ls` show, and what happened to b.txt's original (empty) content?
2. Your colleague runs `rm *` in what they thought was `/tmp/build` but was actually their home directory. What's the damage - and would their `Documents` directory survive?
3. Why is `less /var/log/syslog` better advice than `cat /var/log/syslog`?
4. What's the difference between `rm -r project` and `rmdir project`?

<details>
<summary>Answers</summary>

1. `ls` shows only `b.txt` - `mv` renamed a.txt over b.txt, silently replacing it. `mv -i` would have asked first.
2. Every regular file in home is gone permanently, but `Documents` survives: bare `*` doesn't recurse into directories, and `rm` without `-r` refuses to delete them. (Hidden dotfiles also survive - `*` doesn't match leading dots.)
3. syslog can be thousands of lines; `cat` floods the terminal, `less` lets you scroll and search.
4. `rm -r` deletes the directory and everything inside; `rmdir` only ever deletes *empty* directories, making it the safer tool when you expect emptiness.

</details>

## 📚 Further reading

- [GNU coreutils manual: basic file operations](https://www.gnu.org/software/coreutils/manual/html_node/index.html) - the canonical reference for cp, mv, rm and friends
- `man rm` on your own machine - after the next lesson you'll read it comfortably

---

⬅️ [Previous: Navigating the filesystem](02-navigating-the-filesystem.md) · 🗺️ [Course map](../README.md) · ➡️ [Next: Getting help](04-getting-help.md)

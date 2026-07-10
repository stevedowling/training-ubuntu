# 2 · Reading and setting permissions

> **You'll learn:** to decode any `ls -l` line at a glance, and change who can read, write, or execute anything with `chmod` and `chown`.

## Why this matters

Half of practical Linux troubleshooting ends at a permission: the web server that can't read its config, the script that "won't run", the USB stick you can't write to. The permission model is small - nine bits plus a few specials - and once you can read it fluently, these stop being mysteries and become one-line fixes.

## The big picture

Every file answers three questions for three audiences:

```text
-rwxr-x--- 1 steve devs 4096 Jul 10 09:12 deploy.sh
│└┬┘└┬┘└┬┘   └─┬─┘ └┬─┘
│ │  │  │      │    └── group owner: "devs"
│ │  │  │      └─────── user owner: "steve"
│ │  │  └── others (everyone else): --- nothing
│ │  └───── group (members of devs): r-x read + execute
│ └──────── user (steve):            rwx read + write + execute
└── type: "-" file, "d" directory, "l" symlink
```

The kernel checks in order - **user, then group, then others** - and uses the *first* class that matches you. Which is why the owner can be locked out of their own file (`chmod 077` style accidents): matching stops at "user".

## What rwx means depends on the type

The same three letters mean different things for files and directories - the single most useful table in this module:

| Bit | On a file | On a directory |
|---|---|---|
| `r` | read the contents | list the names inside (`ls`) |
| `w` | change the contents | create, delete, rename entries inside |
| `x` | run it as a program | *enter* it (`cd`, or reach anything through it) |

Two consequences that surprise everyone:

- **Deleting a file needs `w` on the directory, not on the file.** `rm` removes a *directory entry* (lesson 4 explains why). You can delete a root-owned file from your own directory.
- A directory with `r` but not `x` lets you *list* names but not open, `cd` into, or `stat` anything - nearly useless. Directories travel as `r-x` or `rwx`.

## chmod: changing permissions

Two dialects; you need both. **Symbolic** is precise and readable:

```console
$ chmod u+x deploy.sh        # add execute for the user (owner)
$ chmod g-w,o-r notes.txt    # remove group write and others read
$ chmod a+r README.md        # a = all three classes
$ chmod u=rw,go= secret.txt  # set exactly: owner rw, group and others nothing
```

**Octal** sets all nine bits at once - each digit is r=4, w=2, x=1 summed, for user/group/others:

| Octal | Bits | Typical use |
|---|---|---|
| `755` | `rwxr-xr-x` | directories, scripts, programs |
| `644` | `rw-r--r--` | ordinary files |
| `600` | `rw-------` | private files (ssh keys, secrets) |
| `700` | `rwx------` | private directories |

```console
$ chmod 600 ~/.ssh/id_ed25519    # ssh refuses keys that are more open than this
$ chmod -R u+rwX,go=rX shared/   # -R recurses; capital X = x for directories only
```

> [!TIP]
> Fluency drill: translate every permission string you see for a week. `rw-r-----` = 640. `rwxr-x---` = 750. It becomes automatic within days, and both directions matter - docs and error messages use both dialects.

## chown: changing ownership

Only root can give files away (otherwise you could dodge disk quotas or frame other users), so `chown` is a sudo command:

```console
$ sudo chown lab report.txt          # new user owner
$ sudo chown lab:devs report.txt     # user and group at once
$ sudo chown -R www-data:www-data /var/www/site   # recursive - the classic web-server fix
$ chgrp devs report.txt              # group only - allowed without sudo if you own the file and are in devs
```

## umask: where default permissions come from

New files arrive as `664` and directories as `775` - why? Programs create files asking for maximal sensible permissions (`666`/`777`), and your **umask** subtracts bits. Ubuntu's default umask of `002` removes write-for-others:

```text
request 666  (rw-rw-rw-)          request 777  (rwxrwxrwx)
umask  -002                       umask  -002
result  664  (rw-rw-r--)          result  775  (rwxrwxr-x)
```

`umask 077` in your `~/.bashrc` makes everything you create private by default (`600`/`700`) - common on shared machines.

<details>
<summary>🔍 Deep dive: the special bits - setuid, setgid, sticky</summary>

Three more bits sit above the nine, and you've been benefiting from them all along:

- **setuid** (`chmod u+s`, shows as `rws`): the program runs *as its owner*, not as you. `ls -l /usr/bin/passwd` → `-rwsr-xr-x root`: that's how a normal user can edit root-only `/etc/shadow` to change their own password. `sudo` itself works this way.
- **setgid on a directory** (`chmod g+s`, shows as `rwsr-sr-x` style `s` in the group triplet): new files inside inherit the *directory's* group instead of the creator's primary group - the standard trick for shared team directories.
- **sticky** (`chmod +t`, shows as `t` in the others triplet): in a world-writable directory, only a file's owner may delete it. `ls -ld /tmp` → `drwxrwxrwt`: without that `t`, anyone could delete anyone's temp files.

setuid programs are the classic security tightrope - a bug in one hands out root. It's a big part of why Ubuntu 26.04 replaced sudo with the memory-safe sudo-rs (next lesson).

</details>

## 🛠️ Try it

A permissions lab in `~/linux-course/exercises/perms/`, using the `lab` user from lesson 1 as your test subject:

1. Create the directory, and inside it a script: `echo 'echo it works' > runme.sh`. Try `./runme.sh` - explain the error, fix it with chmod, run it.
2. Create `secret.txt` with some text and make it readable by *you alone* (octal). Verify: `su - lab`, then try to read it via its full path.
3. Back as yourself: make a directory `dropbox` where lab can create files but *cannot list* what's in it (write and enter, no read). Test both halves as lab.
4. Predict-then-check: `chmod 444 runme.sh && ./runme.sh` - what happens and why?
5. Cleanup drill: set `perms/` and everything in it to owner-full, group-read/enter, others-nothing, in one recursive command (mind the file/directory difference - capital `X`).

<details>
<summary>💡 Hint 1</summary>

Step 3: lab needs `wx` on the directory but not `r` - and lab is "others" from the file's point of view, so something like `o=wx`. Step 5: `u+rwX,g=rX,o=` with `-R`.

</details>

<details>
<summary>✅ Solution</summary>

```console
$ mkdir -p ~/linux-course/exercises/perms && cd ~/linux-course/exercises/perms
$ echo 'echo it works' > runme.sh
$ ./runme.sh                      # 1: Permission denied - no x bit
$ chmod u+x runme.sh && ./runme.sh
it works
$ echo "the answer is 42" > secret.txt
$ chmod 600 secret.txt            # 2
$ su - lab -c "cat $PWD/secret.txt"     # Permission denied
$ mkdir dropbox && chmod 733 dropbox    # 3: rwx for you, wx for group+others
$ su - lab
lab$ touch /home/steve/linux-course/exercises/perms/dropbox/hi.txt   # works
lab$ ls    /home/steve/linux-course/exercises/perms/dropbox          # Permission denied
lab$ exit
$ chmod 444 runme.sh && ./runme.sh      # 4: Permission denied - r without x isn't runnable
$ chmod -R u+rwX,g=rX,o= .              # 5: X gives x to dirs (and existing executables) only
```

(If `su - lab` can't even reach the path, check the `x` bit on every directory along the way - your home may be `750`, which blocks lab entirely. `chmod o+x ~` for the exercise, or run the test inside `/tmp` instead.)

</details>

## ✋ Checkpoint

1. Translate both ways: `rw-r-----` to octal, and `750` to letters.
2. A file is `-r--r--r-- root root` and sits in a directory where you have `rwx`. Can you delete it? Can you edit it?
3. Predict: you own `diary.txt` with permissions `----rw-rw-`. You're also in its group. Can you read your own file?
4. A web app under `/var/www` throws "Permission denied" reading its own config. Give the two commands you'd run *first* to diagnose (not fix) it.

<details>
<summary>Answers</summary>

1. `640`, and `rwxr-x---`.
2. Delete yes - deletion is controlled by `w` on the *directory*. Edit no - writing needs `w` on the file itself.
3. No - the kernel matches you as *user* first (`---`) and stops; group bits are never consulted for the owner. `chmod u+rw` fixes it.
4. `ls -l` on the config file (who may read it?) and `ps -o user= -C <appname>` or check the service's user (who is asking?) - permission bugs are always a mismatch between those two answers.

</details>

## 📚 Further reading

- `man chmod` - especially the sections on symbolic mode and `X`
- [Ubuntu Server docs: security basics](https://documentation.ubuntu.com/server/explanation/intro-to/security/) - permissions in the wider hardening picture

---

⬅️ [Previous: Users, groups, and root](01-users-groups-and-root.md) · 🏠 [Course home](../README.md) · ➡️ [Next: sudo - borrowing root safely](03-sudo-borrowing-root-safely.md)

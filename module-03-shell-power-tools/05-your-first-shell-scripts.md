# 5 · Your first shell scripts

> **You'll learn:** to turn a pipeline you'd retype into a script you run by name - with arguments, conditionals, loops, and enough safety rails to trust it.

## Why this matters

The moment you run the same three commands twice, they want to be a script. Scripts are also how the system itself works - Ubuntu boots, builds, and maintains itself substantially through shell scripts - so reading and writing them is core literacy, not just automation.

## The big picture

A complete, honest script - every feature of this lesson in 20 lines:

```bash
#!/bin/bash
# backup.sh - snapshot a directory into ~/backups, dated
set -euo pipefail                       # safety rails: die on errors and unset vars

src="${1:-}"                            # first argument, empty if missing
if [ -z "$src" ]; then                  # -z: is the string empty?
    echo "usage: backup.sh <directory>" >&2
    exit 1                              # non-zero = failure (lesson 1 pays off)
fi
if [ ! -d "$src" ]; then
    echo "error: $src is not a directory" >&2
    exit 1
fi

dest="$HOME/backups/$(basename "$src")-$(date +%F-%H%M%S)"
mkdir -p "$HOME/backups"
cp -r "$src" "$dest"
echo "backed up $src to $dest"
```

Read it top to bottom: a **shebang**, a **safety line**, **argument checks that fail loudly**, then the actual work - which is just module-1 commands. Most good scripts have exactly this anatomy: 70% checking, 30% doing.

## From pipeline to script

Three steps turn any one-liner into a command:

```console
$ nano ~/bin/shellstats                      # 1. write it (in your ~/bin from lesson 4)
#!/bin/bash
cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn
$ chmod +x ~/bin/shellstats                  # 2. make it executable (module 2!)
$ shellstats                                 # 3. run it like any command
```

The `#!/bin/bash` **shebang** must be line 1: when the kernel is asked to run a text file, it reads that line to learn which interpreter to hand the file to. No shebang, no reliable behaviour.

> [!NOTE]
> `sh` is not bash: on Ubuntu, `/bin/sh` links to **dash**, a smaller, faster, plainer shell used for boot scripts. Write `#!/bin/bash` and get bash's features; write `#!/bin/sh` only when you're deliberately writing portable POSIX shell.

## Variables, arguments, and the quoting rule

Inside a script, arguments arrive as `$1`, `$2`, ... with `$#` (count) and `$@` (all of them). Command output becomes a value with `$(...)`:

```bash
name="$1"
host="$(hostname)"
today="$(date +%F)"
echo "report for $host on $today: hello, $name"
```

And the rule that separates working scripts from haunted ones: **double-quote every expansion**. `"$1"`, `"$src"`, `"$(pwd)"`. Unquoted, a value containing a space splits into multiple words and a value containing `*` becomes a glob - `rm $file` with `file="my notes.txt"` tries to delete `my` and `notes.txt`.

## if: making decisions

```bash
if [ -f "$path" ]; then          # a file exists?
    echo "found it"
elif [ -d "$path" ]; then        # a directory?
    echo "it's a directory"
else
    echo "no such thing" >&2
fi
```

`[` is really a command (alias of `test` - `type [` proves it), so the spaces around it are mandatory. The tests you'll reach for:

| Test | True when |
|---|---|
| `-f x` / `-d x` / `-e x` | x is a file / directory / exists at all |
| `-z "$s"` / `-n "$s"` | string empty / non-empty |
| `"$a" = "$b"` | strings equal |
| `"$a" -eq "$b"` (also `-lt`, `-gt`) | numbers equal (less/greater than) |
| `! expr` | negation |

Because `if` really just checks an exit status (lesson 1!), any command works as a condition:

```bash
if grep -q "$USER" /etc/passwd; then
    echo "you exist"
fi
```

## Loops: doing it to everything

```bash
for f in *.log; do                      # glob loop - the daily driver
    echo "== $f =="
    tail -3 "$f"
done

for user in steve lab root; do          # list loop
    id "$user" || echo "$user missing"
done

while read -r line; do                  # process input line by line
    echo "got: $line"
done < /etc/hostname
```

The pattern behind most real scripts is *for each thing, do the module-1-through-3 stuff* - loops are just glue around commands you already know.

<details>
<summary>🔍 Deep dive: the safety rails, and the linter that reviews your scripts</summary>

Bash's default failure mode is to shrug and continue - a typo'd variable is empty, a failed command is ignored. `set -euo pipefail` flips all three:

- `-e`: exit on any command failure (instead of barreling on with half-done state)
- `-u`: unset variables are errors (catches `rm -rf "$TYPO/"` becoming `rm -rf /`)
- `-o pipefail`: a pipeline fails if *any* stage fails, not just the last

These rails have sharp corners of their own (sometimes you *expect* a failure - append `|| true` to allow one), but starting scripts with them and consciously relaxing beats debugging silent corruption.

The other habit of people whose scripts work: **shellcheck**, a linter that catches quoting bugs, useless-use-of-cat, and a hundred classic mistakes with eerily good explanations:

```console
$ sudo apt install shellcheck
$ shellcheck ~/bin/backup.sh
```

Make it part of the write-run loop from day one - it is the code review you always have.

</details>

## 🛠️ Try it

Build `organize` - a script that files loose downloads into folders by extension. In `~/bin/organize`:

1. Take one argument: the directory to organize. Reject missing args and non-directories, loudly, exiting non-zero (steal the anatomy from the big picture).
2. For every *file* directly in that directory, work out its extension (`ext="${f##*.}"` grabs text after the last dot - bash's built-in surgery), create `<dir>/<ext>/` if needed, and `mv` the file in.
3. Print a per-file line (`photo.jpg -> jpg/`) and a final count of files moved.
4. Test it on a scratch directory: `mkdir /tmp/mess && cd /tmp/mess && touch a.txt b.txt c.jpg d.pdf e.jpg` - then `organize /tmp/mess` and inspect with `ls -R`.
5. Run `shellcheck ~/bin/organize` and fix everything it flags.

Stretch: files with *no* extension (like `README`) - decide a behaviour (skip? an `other/` folder?) and implement it.

<details>
<summary>💡 Hint 1</summary>

The loop skeleton: `for f in "$1"/*; do [ -f "$f" ] || continue; ...; done` - the `continue` skips directories. For the no-extension stretch: when a name has no dot, `${f##*.}` returns the whole name - compare `"$ext" = "$(basename "$f")"` to detect it.

</details>

<details>
<summary>✅ Solution</summary>

```bash
#!/bin/bash
# organize - file loose files into subfolders by extension
set -euo pipefail

dir="${1:-}"
if [ -z "$dir" ]; then
    echo "usage: organize <directory>" >&2
    exit 1
fi
if [ ! -d "$dir" ]; then
    echo "error: $dir is not a directory" >&2
    exit 1
fi

moved=0
for f in "$dir"/*; do
    [ -f "$f" ] || continue                 # files only
    base="$(basename "$f")"
    ext="${f##*.}"
    if [ "$ext" = "$base" ]; then           # no dot at all
        ext="other"                         # stretch goal: the no-extension policy
    fi
    mkdir -p "$dir/$ext"
    mv "$f" "$dir/$ext/"
    echo "$base -> $ext/"
    moved=$((moved + 1))
done
echo "moved $moved files"
```

```console
$ chmod +x ~/bin/organize
$ mkdir /tmp/mess && touch /tmp/mess/{a.txt,b.txt,c.jpg,d.pdf,e.jpg,README}
$ organize /tmp/mess
a.txt -> txt/
...
moved 6 files
$ ls -R /tmp/mess       # jpg/ pdf/ txt/ other/
$ shellcheck ~/bin/organize && echo clean
```

</details>

## ✋ Checkpoint

1. Predict: a script containing `echo "one"; exit 0; echo "two"` prints what - and what does `echo $?` show right after running it?
2. `if [$x = 5]` fails with "command not found". Two separate bugs - name them.
3. A script works when you run it but fails from cron with "mycommand: not found". Diagnose, using lesson 4's deep dive.
4. What's the difference in danger between `rm -rf $dir/` and `rm -rf "$dir/"` when `dir` is unset - and which safety-rail flag turns both into a loud error instead?

<details>
<summary>Answers</summary>

1. Prints `one`; the `exit 0` ends the script, so "two" never runs; `$?` is 0.
2. Missing spaces - `[` is a command so it must be `[ $x`, and `5]` needs to be `5 ]`. (And for full credit: `$x` should be `"$x"`.)
3. cron runs a non-interactive shell with a minimal PATH - `mycommand` lives somewhere cron doesn't search. Use the absolute path in the script, or set `PATH=...` at the top of it.
4. Unquoted, `rm -rf $dir/` becomes `rm -rf /` - catastrophe; quoted, `rm -rf "/"` - still catastrophe, actually, because the slash remains! Quoting alone doesn't save you here; `set -u` does, by making the unset `$dir` a fatal error before rm ever runs.

</details>

## 📚 Further reading

- [ShellCheck online](https://www.shellcheck.net/) - paste any script, get schooled; the wiki page for each warning is a mini-lesson
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html) - opinionated, short, and includes "when to stop using shell" (their answer: ~100 lines)
- `man bash`, section SHELL GRAMMAR - when you're ready for the whole truth

---

⬅️ [Previous: The shell as a home](04-the-shell-as-a-home.md) · 🏠 [Course home](../README.md) · ➡️ Next module: [Processes & the Kernel](../module-04-processes-and-the-kernel/README.md)

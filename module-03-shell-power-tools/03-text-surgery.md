# 3 · Text surgery - sort, uniq, cut, sed, awk

> **You'll learn:** to slice columns, count and rank occurrences, and rewrite text in flight - turning raw command output and log files into answers.

## Why this matters

Module 1's deep dive showed that on Linux everything speaks text: configs, logs, `/proc`, command output. That was a promise, and this lesson collects on it. The same six tools that summarize a web log also rank disk hogs and reshape CSV exports - learn the pattern once, apply it to everything.

## The big picture

Most text questions are answered by one shape of pipeline - **extract, then aggregate**:

```console
$ cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn
     18 /usr/sbin/nologin
      9 /bin/false
      2 /bin/bash
      1 /bin/sync
```

Read it left to right: *pull out the shell column → group identical lines together → count each group → rank by count*. That's "which shells are used, by popularity" in one line. Swap the `cut` for a different extractor and the same tail end ranks IP addresses in a web log, error types in syslog, or file extensions in a project.

## The workhorses

| Tool | Job | Flags to know |
|---|---|---|
| `sort` | order lines | `-n` numeric, `-r` reverse, `-h` human sizes (2K < 1G), `-t: -k3` sort by field 3, colon-separated |
| `uniq` | collapse *adjacent* duplicates | `-c` count, `-d` show only dupes |
| `cut` | keep certain columns | `-d: -f1,7` fields 1 and 7 by delimiter, `-c1-8` by character |
| `wc` | count | `-l` lines, `-w` words, `-c` bytes |
| `tr` | translate characters | `tr a-z A-Z`, `tr -s ' '` squeeze repeats, `tr -d '\r'` delete |

```console
$ sort -t: -k3 -n /etc/passwd | tail -5        # five highest UIDs
$ du -sh /var/* 2>/dev/null | sort -rh | head -3    # biggest dirs, human-sorted
$ echo "too   many    spaces" | tr -s ' '      # squeeze runs of spaces to one
```

> [!WARNING]
> `uniq` only collapses lines that are *adjacent* - it's a deduplicating window, not a database. That's why it's nearly always `sort | uniq`, and why `sort -u` exists as the fused shortcut when you don't need counts.

## sed: find and replace, streamed

sed edits text as it flows past. You'll use one command for 90% of jobs - substitute:

```console
$ sed 's/nologin/NOPE/' /etc/passwd | head -3          # replace first hit per line (output only!)
$ sed 's/0/O/g' file.txt                               # /g = every hit on the line
$ sed -E 's/(steve|lab)/USER/g' /etc/passwd | head -3  # -E: same regex dialect as grep -E
$ sed -i.bak 's/^Port 22$/Port 2222/' myconfig         # -i: edit the file in place, keeping a .bak
$ sed -n '5,10p' /var/log/dpkg.log                     # not everything is s///: print lines 5-10
$ sed '/^#/d' /etc/login.defs                          # delete comment lines from the stream
```

sed never touches the input file unless you pass `-i` - it's a filter, printing the edited stream to stdout. The `s/old/new/` pattern is regex (lesson 2 pays off immediately), and the delimiter is yours to choose: `s|/var/log|/tmp|` avoids escaping slashes in paths.

## awk: columns with a brain

Where cut needs tidy delimiters, awk splits on *any whitespace* and lets you compute. Its model: for each line, fields are `$1`, `$2`... whole line is `$0`; an optional `condition { action }` pair runs per line:

```console
$ df -h | awk '{print $5, $6}'                        # columns 5+6 of df: use%, mountpoint
$ awk -F: '$3 >= 1000 {print $1}' /etc/passwd         # module 2's humans, formalized
$ ls -l /etc | awk '$5 > 10000 {print $9, $5}'        # files over 10 KB: name and size
$ awk '{sum += $2} END {print sum}' data.txt          # total a column
```

`-F:` sets the delimiter; `END { }` runs after the last line - which is how sums and averages come out. If a pipeline needs one field compared or one number totted up, awk is the tool that keeps it a one-liner.

<details>
<summary>🔍 Deep dive: awk is a whole programming language</summary>

What this lesson uses is maybe 5% of awk. It has variables, arrays, functions, printf, regex matches per-field, and BEGIN blocks - people have written database engines and games in it. One taste, the extract-and-aggregate pipeline as a single awk program:

```console
$ awk -F: '{count[$7]++} END {for (s in count) print count[s], s}' /etc/passwd | sort -rn
```

`count[$7]++` builds a hash table keyed by shell - no sort, no uniq, one pass. When a pipeline grows a third `awk`/`sed` stage, that's usually the sign to write either one awk program or (module wisdom) a Python script. `man awk` or the GNU gawk manual when the day comes; on Ubuntu, `awk` itself is - fittingly - a symlink chain through `/etc/alternatives` to `mawk`, exactly as lesson 4 of module 2 described.

</details>

## 🛠️ Try it

Real analysis on your own machine, answers into `~/linux-course/exercises/surgery.txt`:

1. Recreate the shell-popularity table from the big picture, then modify it: only shells used by *system* accounts (UID < 1000). Two tool choices - `awk` filter or nothing-but-cut won't do it alone.
2. Top 5 biggest directories directly under `/usr`, human-readable, biggest first.
3. From `ls -l /etc`, produce just `name size` for the 5 largest *files*, using awk (mind: `ls -l` lines for directories start with `d` - filter them).
4. Take `dpkg -l | tail -n +6` (installed packages, headers skipped) and count packages per *first letter* of the package name. Ranked. (Extract with `awk '{print $2}'`, then `cut -c1`, then the classic tail end.)
5. sed drill: print `/etc/os-release` with every `=` turned into `: `, and the quotes deleted (two sed expressions chain with `-e ... -e ...`, or pipe two seds).

<details>
<summary>💡 Hint 1</summary>

Step 1: `awk -F: '$3 < 1000 {print $7}' /etc/passwd | sort | uniq -c | sort -rn`. Step 3: `awk '/^-/ {print $9, $5}'` selects regular files - a regex condition instead of a numeric one.

</details>

<details>
<summary>✅ Solution</summary>

```console
$ cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn                    # 1a
$ awk -F: '$3 < 1000 {print $7}' /etc/passwd | sort | uniq -c | sort -rn # 1b
$ du -sh /usr/* 2>/dev/null | sort -rh | head -5                         # 2
$ ls -l /etc | awk '/^-/ {print $9, $5}' | sort -k2 -rn | head -5        # 3
$ dpkg -l | tail -n +6 | awk '{print $2}' | cut -c1 | sort | uniq -c | sort -rn | head   # 4: l wins (all those libs)
$ sed -e 's/=/: /' -e 's/"//g' /etc/os-release                           # 5
```

</details>

## ✋ Checkpoint

1. Predict: `printf "b\na\nb\n" | uniq` prints what - and how do you make it print two lines?
2. Your pipeline `... | sort -k2 | head` ranks sizes wrong: 100 comes before 20. One character fixes it - which?
3. Write the sed that changes every `/home/steve` to `/home/lab` in a stream, without escaping any slashes.
4. Which tool: "print the username and shell of every account whose shell is not nologin" - and roughly the command?

<details>
<summary>Answers</summary>

1. `b a b` unchanged - the two b's aren't adjacent. `sort | uniq` (or `sort -u`) gives `a b`.
2. `-n` (numeric sort): `sort -k2 -n`. Text sort puts "100" before "20" because 1 < 2 as characters.
3. `sed 's|/home/steve|/home/lab|g'` - any delimiter works after `s`; picking `|` sidesteps the slashes.
4. awk: `awk -F: '$7 !~ /nologin/ {print $1, $7}' /etc/passwd` (grep -v + cut also gets there - many right answers in this game).

</details>

## 📚 Further reading

- [GNU sed manual, "sed one-liners"](https://www.gnu.org/software/sed/manual/sed.html) - decades of collected tricks
- [The AWK Programming Language, 2nd ed.](https://awk.dev/) - by awk's authors (the A, W, and K), short and brilliant

---

⬅️ [Previous: grep and find](02-grep-and-find.md) · 🗺️ [Course map](../README.md) · ➡️ [Next: The shell as a home](04-the-shell-as-a-home.md)

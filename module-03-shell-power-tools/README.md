# Module 3 · Shell Power Tools

> **Goal:** stop running commands one at a time and start composing them - pipe programs together, search and reshape any text, make the shell your own, and automate the result as scripts.

This is the module where the command line starts paying rent. Lessons 1-3 build the composition skills (pipes → searching → transforming) and are best in order; lessons 4-5 (configuring your shell, scripting) round it out and lean on everything before them.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [Pipes and redirection](01-pipes-and-redirection.md) | Route stdout/stderr anywhere and chain small tools into answers |
| 2 | [grep and find](02-grep-and-find.md) | Find any text in any file, and any file on the system - plus a first dose of regex |
| 3 | [Text surgery](03-text-surgery.md) | Slice, count, rank, and rewrite text with sort, uniq, cut, sed, and awk |
| 4 | [The shell as a home](04-the-shell-as-a-home.md) | $PATH, variables, .bashrc, aliases, and history tricks that double your speed |
| 5 | [Your first shell scripts](05-your-first-shell-scripts.md) | Turn pipelines into named commands with arguments, ifs, loops, and safety rails |

## Before you start

- You need [Module 1](../module-01-first-contact/README.md) throughout. [Module 2](../module-02-users-and-permissions/README.md) helps (chmod appears when scripts arrive, and some examples read /etc/passwd with module-2 eyes) but isn't a hard requirement - the map shows them as parallel tracks.
- Exercises keep collecting in `~/linux-course/exercises/`, and from lesson 4 you'll build a personal `~/bin` that the rest of the course assumes exists.

> [!TIP]
> This module is dense with flags. Don't memorize - do the exercises, keep your winning one-liners in the exercises files, and let `Ctrl+R` (lesson 4) be your memory.

---

⬅️ [Module 2: Users & Permissions](../module-02-users-and-permissions/README.md) · 🏠 [Course home](../README.md) · ➡️ [Start: Pipes and redirection](01-pipes-and-redirection.md)

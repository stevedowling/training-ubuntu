# Module 2 · Users & Permissions

> **Goal:** understand who can do what on a Linux system - read any `ls -l` line at a glance, set ownership and permissions deliberately, use sudo with confidence, and know what a file *really* is underneath its name.

This module is where "Permission denied" stops being an obstacle and becomes information. Lessons 1-3 build on each other; lesson 4 (links and inodes) stands mostly alone and pays off across the whole rest of the course.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [Users, groups, and root](01-users-groups-and-root.md) | Who's who: identities, groups, system accounts, and what makes UID 0 special |
| 2 | [Reading and setting permissions](02-reading-and-setting-permissions.md) | Decode any `ls -l` line and wield chmod, chown, and umask |
| 3 | [sudo - borrowing root safely](03-sudo-borrowing-root-safely.md) | Use root power per-command, grant narrow slices of it, and read the audit trail |
| 4 | [Links and inodes](04-links-and-inodes.md) | What a file really is - and the hard links and symlinks Ubuntu is built on |

## Before you start

- You need [Module 1](../module-01-first-contact/README.md) reflexes: navigating, `ls -l`, reading man pages.
- The exercises create a throwaway user called `lab` in lesson 1 and reuse it through the module. Do them in order, or run `sudo adduser lab` yourself before starting at lesson 2 or 3.
- When you're done with the module: `sudo deluser --remove-home lab` cleans up.

> [!TIP]
> This module touches system state (users, sudoers entries) more than module 1 did. Everything is reversible and each lesson cleans up after itself - but a VM or container remains the ideal playground.

---

⬅️ [Module 1: First Contact](../module-01-first-contact/README.md) · 🏠 [Course home](../README.md) · ➡️ [Start: Users, groups, and root](01-users-groups-and-root.md)

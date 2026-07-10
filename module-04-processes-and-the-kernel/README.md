# Module 4 · Processes & the Kernel

> **Goal:** see the living system - every running program as a member of one process tree, signals as the way to control them, /proc as the kernel's open book, and syscalls as the boundary where all of it happens.

This is the module where the course goes below the command line. It cashes in promises from earlier modules: module 1's fork/exec deep dive becomes visible in ps and strace, module 2's deleted-file mystery turns up in /proc/PID/fd, and module 3's streams become file descriptors you can list. Do lessons 1-2 in order; 3 and 4 build on them.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [What a process is](01-what-a-process-is.md) | PIDs and the process tree, ps and top fluency, states, load average, and where the memory went |
| 2 | [Signals and job control](02-signals-and-job-control.md) | What Ctrl+C really sends, the kill escalation ladder, and juggling jobs in your shell |
| 3 | [/proc and /sys](03-proc-and-sys.md) | The kernel as a filesystem: every process inside out, hardware as files, live tunables |
| 4 | [Syscalls and strace](04-syscalls-and-strace.md) | The user/kernel boundary, and watching any program's every move across it |

## Before you start

- You need [Module 1](../module-01-first-contact/README.md) plus [Module 3](../module-03-shell-power-tools/README.md)'s pipes-and-grep reflexes throughout; [Module 2](../module-02-users-and-permissions/README.md) concepts (ownership, the lab user's leftovers) appear regularly.
- Lesson 4 needs one install: `sudo apt install strace` (and `htop` is a recommended companion for lesson 1).
- Everything in this module inspects or gently pokes *your own* processes - safe on any machine you can log into.

> [!TIP]
> Keep two terminals open through this module. Watching a process from one terminal while you signal it from another is half the learning.

---

⬅️ [Module 3: Shell Power Tools](../module-03-shell-power-tools/README.md) · 🏠 [Course home](../README.md) · ➡️ [Start: What a process is](01-what-a-process-is.md)

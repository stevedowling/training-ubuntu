# Module 4 · Processes & the Kernel

> **Goal:** see the living system - understand what a process is, inspect and control the ones running right now, and watch programs talk to the kernel through system calls.

## Learning objectives

- Explain PIDs, parent/child processes, and the process tree (`ps`, `pstree`, `top`, `htop`)
- Send signals with `kill`, and know what SIGTERM vs SIGKILL vs Ctrl+C really do
- Manage foreground/background jobs: `&`, `jobs`, `fg`, `bg`, `nohup`
- Read the kernel's live filesystems: `/proc` and `/sys`
- Watch system calls with `strace` and understand fork/exec
- Interpret load average, memory usage (`free`), and the OOM killer

## Planned lessons

1. What a process is - PIDs, the tree, ps and top
2. Signals and job control - politely (and impolitely) stopping things
3. /proc and /sys - the kernel as a filesystem
4. Syscalls and strace - watching the kernel boundary

> [!NOTE]
> **Not written yet** - ask Claude to "write module 4" when you get here.

---

⬅️ [Module 2: Users & Permissions](../module-02-users-and-permissions/README.md) · 🗺️ [Course map](../README.md)

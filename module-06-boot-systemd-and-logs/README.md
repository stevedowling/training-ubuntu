# Module 6 · Boot, systemd & Logs

> **Goal:** know what happens between the power button and your login prompt, run services like an administrator, and find anything in the logs.

## Learning objectives

- Trace the boot chain: UEFI → GRUB → kernel → initramfs → systemd
- Understand systemd units, and drive services with `systemctl` (start/stop/enable/status)
- Read unit files, and write a simple service of your own
- Master `journalctl`: filter by unit, time, priority, and boot
- Schedule work with systemd timers and classic cron
- Diagnose "why is boot slow" with `systemd-analyze`

## Planned lessons

1. From power button to login - the boot chain
2. systemd and services - systemctl fluency
3. journald - one log to rule them all
4. Timers and cron - running things on a schedule

> [!NOTE]
> **Not written yet** - ask Claude to "write module 6" when you get here.

---

⬅️ [Module 4: Processes & the Kernel](../module-04-processes-and-the-kernel/README.md) · 🗺️ [Course map](../README.md)

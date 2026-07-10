# Module 6 · Boot, systemd & Logs

> **Goal:** know what happens between the power button and your login prompt, drive services like an administrator, find anything in the logs, and schedule work that runs without you.

This is the operations module: module 4 taught you to see processes; this one teaches you to *manage* the system that starts and supervises them. Lesson 1 (boot) stands alone; lessons 2-4 build on each other - services, then their logs, then scheduling them.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [From power button to login](01-from-power-button-to-login.md) | The UEFI → GRUB → kernel → initramfs → systemd chain, and measuring your own boot |
| 2 | [systemd and services](02-systemd-and-services.md) | systemctl fluency, the running-vs-enabled axes, reading unit files, and writing your own service |
| 3 | [journald](03-journald.md) | Finding anything in the journal by service, time, priority, and boot - and managing its disk |
| 4 | [Timers and cron](04-timers-and-cron.md) | Both schedulers Ubuntu ships, the cron environment trap, and picking the right tool per job |

## Before you start

- Needs [Module 4](../module-04-processes-and-the-kernel/README.md) (PIDs, signals) and [Module 3](../module-03-shell-power-tools/README.md) scripting; [Module 5](../module-05-software-and-packages/README.md) makes cameo appearances (unattended-upgrades turns out to be a timer).
- The exercises create and destroy a `heartbeat` service in lesson 2 that lesson 3 briefly reuses - keep it around until then if you're doing the module in one sitting.
- Everything is reversible and cleans up after itself; a reboot is never *required*, though lesson 1 is more fun if you can do one.

> [!TIP]
> `systemctl status <thing>` is this module's `ls -l` - when in doubt, run it and read the whole block. By lesson 4 that habit will feel automatic.

---

⬅️ [Module 4: Processes & the Kernel](../module-04-processes-and-the-kernel/README.md) · 🗺️ [Course map](../README.md) · ➡️ [Start: From power button to login](01-from-power-button-to-login.md)

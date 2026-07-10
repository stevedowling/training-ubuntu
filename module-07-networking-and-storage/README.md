# Module 7 · Networking & Storage

> **Goal:** diagnose "the network is down" and "the disk is full" like a professional - climb the connectivity ladder rung by rung, work remote machines over SSH as fluently as your own, and command the chain from block device to mounted filesystem.

The final module, and the most operational: these four lessons are the ones you'll reach for during actual incidents. Lessons 1-2 are the network pair, 3-4 the storage pair - the pairs are independent, so start with whichever failure mode haunts you more.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [The network stack](01-the-network-stack.md) | The layer-by-layer debugging ladder: ip, ping, resolvectl, ss, and netplan |
| 2 | [SSH - remote work done right](02-ssh.md) | Key pairs, ssh config, scp and rsync, and the server-hardening basics |
| 3 | [Disks and filesystems](03-disks-and-filesystems.md) | Block devices to mount points, fstab literacy, and a zero-risk loop-disk lab |
| 4 | [Space and growth](04-space-and-growth.md) | The full-disk investigation procedure, the five usual suspects, and live growth with LVM |

## Before you start

- Needs [Module 4](../module-04-processes-and-the-kernel/README.md) (processes, lsof) and [Module 6](../module-06-boot-systemd-and-logs/README.md) (systemctl, journalctl) - both get used in anger here. Module 2's `lab` user returns for the SSH exercises.
- No second machine required: SSH is practiced against localhost, and all disk work happens on loop devices made from files - real mechanics, zero risk.
- One install along the way: `openssh-server` (lesson 2). `ncdu` from module 5 finally earns its keep in lesson 4.

> [!TIP]
> This module's exercises produce two artifacts worth keeping beyond the course: your `~/.ssh/config` and your ssh key pair. Everything else cleans up after itself.

---

⬅️ [Module 6: Boot, systemd & Logs](../module-06-boot-systemd-and-logs/README.md) · 🗺️ [Course map](../README.md) · ➡️ [Start: The network stack](01-the-network-stack.md)

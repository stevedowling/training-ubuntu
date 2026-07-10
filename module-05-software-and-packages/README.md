# Module 5 · Software & Packages

> **Goal:** own how software gets onto your machine - drive apt fluently, interrogate dpkg's database, weigh snaps and third-party sources with a clear trust model, and build from source once so packages never look like magic again.

The module has an arc: lesson 1 is the daily practice, lesson 2 opens the machinery, lesson 3 maps the other channels and their trust costs, and lesson 4 makes you do the package manager's job by hand - which is the fastest way to understand why it exists. Best in order.

## Lessons

| # | Lesson | You'll learn |
|---|--------|--------------|
| 1 | [apt essentials](01-apt-essentials.md) | update vs upgrade, install/remove/purge, searching the catalogue, and the automatic security patching you already have |
| 2 | [Under the hood](02-under-the-hood-dpkg-and-repos.md) | dpkg's database, dissecting a .deb, sources files, and the signature chain of trust |
| 3 | [Snaps and other channels](03-snaps-and-other-channels.md) | snap's sandbox and channels, PPAs and vendor repos, and choosing a channel per tool |
| 4 | [From source](04-from-source.md) | tar, configure/make/make install without making a mess - and the case for packages, proven by hand |

## Before you start

- Needs [Module 2](../module-02-users-and-permissions/README.md) (sudo throughout) and [Module 3](../module-03-shell-power-tools/README.md) reflexes (pipes and awk appear in the exercises).
- Everything installed in the exercises is either removed again or genuinely useful later (htop, ncdu, tldr, shellcheck, build-essential).
- Lesson 4 downloads a small source tarball from gnu.org - the one exercise in the course so far needing general internet beyond the Ubuntu archive.

> [!TIP]
> This module changes system state constantly - that's its subject. Every step is still reversible, but it's the best module yet for practicing on a VM or container snapshot you can roll back.

---

⬅️ [Module 2: Users & Permissions](../module-02-users-and-permissions/README.md) · 🗺️ [Course map](../README.md) · ➡️ [Start: apt essentials](01-apt-essentials.md)

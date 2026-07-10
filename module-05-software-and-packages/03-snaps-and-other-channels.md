# 3 · Snaps and other channels

> **You'll learn:** to manage snap packages alongside debs, add third-party sources with eyes open, and choose the right channel for any piece of software.

## Why this matters

On a stock Ubuntu 26.04 desktop you're *already* running snaps - Firefox and the App Center among them - so half your software updates by rules lesson 1 never mentioned. And sooner or later a project's install page will say "add our repository" or "download our .deb". Each channel has a different trust model and failure mode; choosing well is a professional skill.

## The big picture

| | deb (apt) | snap | third-party repo / PPA | random .deb |
|---|---|---|---|---|
| Curated by | Ubuntu archive | publisher, via Canonical's store | that publisher alone | that website alone |
| Updates | apt upgrade + unattended | automatic, snapd's schedule | with your apt upgrades | never - you're the updater |
| Isolation | none - files everywhere | confined sandbox | none | none |
| Freshness | frozen per release (secure, stale) | publisher-current | publisher-current | snapshot in time |
| Trust cost | Canonical | Canonical + publisher | full root trust in publisher | full root trust, no chain |

The strategy that falls out: **apt first** (integrated, boring, patched), **snap when you want the newer version or the sandbox**, **third-party repos only from vendors you'd trust with root** - because that's literally the transaction - and random .debs as a last resort, inspected first (lesson 2 taught you how).

## Driving snap

The verbs rhyme with apt:

```console
$ snap list                          # what's here? (surprise inventory on desktop)
$ snap find yt-dlp                   # search the store
$ snap info firefox                  # the card: channels, publisher, confinement
$ sudo snap install hello-world      # install
$ sudo snap remove hello-world       # remove
$ sudo snap refresh                  # update everything now (it also just... happens)
```

Two ideas have no apt equivalent. **Channels** - each snap publishes parallel tracks (`stable`, `candidate`, `beta`, `edge`), switchable live:

```console
$ sudo snap refresh firefox --channel=beta     # ride the beta
$ sudo snap refresh firefox --channel=stable   # ...and step back off
```

And **auto-refresh**: snapd updates snaps on its own schedule (a few times daily by default). You can time-window it, defer it, but not permanently disable it - a deliberate design choice trading control for a patched fleet:

```console
$ snap refresh --time                # when's the next pass?
$ sudo snap refresh --hold=24h firefox    # not today, thanks
```

## How snaps work (and why they're like that)

A snap is a **squashfs** image - a compressed, read-only filesystem - carrying the app *and its dependencies*. Install mounts it; nothing unpacks into `/usr`:

```console
$ ls /snap/firefox/                  # one directory per installed revision
current  5012  5077
$ mount | grep snap | head -3        # each revision is a mounted loop device
$ df -h /snap/firefox/current        # read-only, 100% full - by design
```

Consequences, good and bad, all from that one design:

- Self-contained → the publisher ships one artifact for every distro; no dependency hell.
- Read-only + versioned → updates are atomic, and `sudo snap revert firefox` is instant rollback (try *that* with apt).
- Confined → snaps run sandboxed, reaching the system only through declared **interfaces** (`snap connections firefox` lists them: camera? home files? network?). A compromised snap is a caged compromise.
- The costs: bigger downloads (bundled deps), a page of loop mounts in `df`, and occasional friction when the sandbox blocks something a classic app could do (a snap browser can't read `/etc`, by design).

## PPAs and third-party repositories

A **PPA** (Personal Package Archive) is someone's personal apt repository on Launchpad; vendors run the same idea at bigger scale (Docker, PostgreSQL, Microsoft all publish apt repos):

```console
$ sudo add-apt-repository ppa:mozillateam/ppa    # adds source + key, runs apt update
$ ls /etc/apt/sources.list.d/                    # see what it dropped (lesson 2 files!)
$ sudo add-apt-repository --remove ppa:mozillateam/ppa
```

What you actually did there, in lesson-2 terms: added a stranger's signing key to your trust store and their server to your sources. From then on, their packages install and upgrade *as root* like any other - including a package named `bash` if they chose to publish one, since a higher version number wins. Real vendors' official repos are routine practice; a random PPA "with the fix" from a forum post is a root-shaped leap of faith.

> [!TIP]
> Audit ritual, twice a year: `ls /etc/apt/sources.list.d/` and ask of each file "do I still want these people publishing software onto this machine?" Remove what you've stopped needing.

<details>
<summary>🔍 Deep dive: the trust ledger, and where flatpak fits</summary>

The single question that sorts every channel: **who can now put code on my machine, and what stops them putting *bad* code?**

- Ubuntu archive: Canonical; signature chain + their review and build infrastructure.
- Snap store: publisher uploads, Canonical's store scans and (for `strict` confinement) the sandbox contains; `verified` publisher ticks help.
- Vendor repo/PPA: the vendor, full stop - the signature chain proves it's *them*, not that it's *good*. Their key on your disk is standing permission.
- Downloaded .deb: whoever ran the web server, once, unverifiable later.

**Flatpak** - the other universal package format - plays snap's role with a different accent: desktop-app focused, decentralized repos (Flathub the big one), same sandbox-and-runtime idea. Ubuntu ships snap and not flatpak by default; on Ubuntu you'll mostly meet flatpak as the thing other distros' users tell you to use instead. Both are answers to the same real problem: "one Linux app artifact, safely, everywhere". Knowing the *question* is what transfers.

</details>

## 🛠️ Try it

Channel surfing - notes into `~/linux-course/exercises/channels.txt`:

1. Inventory the surprise: `snap list`. What's running as a snap that you assumed was a deb? For one of them, `snap info` it: publisher, channel, last refresh.
2. Full snap lifecycle: install `hello-world`, run it (`hello-world`), locate its mount (`ls /snap/hello-world/`, `mount | grep hello`), then remove it.
3. Peek in the cage: `snap connections firefox` (or any GUI snap). Which interfaces are connected? Anything it can't touch that surprises you?
4. When is snapd's next auto-refresh (`snap refresh --time`)? Push it: hold all refreshes for 6 hours, verify, release the hold (`sudo snap refresh --hold=6h` / `--unhold`).
5. Third-party recon *without commitment*: visit no website - instead `ls /etc/apt/sources.list.d/` and count non-Ubuntu sources already on your machine. For each: lesson 2's question - whose key, still wanted?
6. Decision drill, on paper: you need (a) `jq`, a small stable CLI tool; (b) the newest VS Code, updating itself; (c) a niche tool whose site offers only a .deb. Pick a channel for each and defend it in one line.

<details>
<summary>💡 Hint 1</summary>

Step 6 has defensible variations - the grading rubric is only "did you weigh freshness vs trust vs isolation out loud". (One classic answer: apt for jq; Microsoft's official repo or the snap for Code; inspect-then-install for the .deb, with a calendar note that *you* own its updates now.)

</details>

<details>
<summary>✅ Solution</summary>

```console
$ snap list                                        # 1: firefox, snapd, core*, gnome-* runtimes...
$ snap info firefox | head -15
$ sudo snap install hello-world && hello-world     # 2
Hello World!
$ ls /snap/hello-world/ && mount | grep hello
$ sudo snap remove hello-world
$ snap connections firefox | head                  # 3: audio, camera?, home, network...
$ snap refresh --time                              # 4
$ sudo snap refresh --hold=6h && snap refresh --time
$ sudo snap refresh --unhold
$ ls /etc/apt/sources.list.d/                      # 5: ubuntu.sources +... whatever history added
```

Step 6: see the hint - reasoning shown beats any particular answer.

</details>

## ✋ Checkpoint

1. Firefox-as-snap updated itself overnight; Firefox-as-deb waits for `apt upgrade`. Name one scenario where each behaviour is clearly the better one.
2. Predict: `df -h` shows a dozen loop devices at 100% use. Disk emergency?
3. A blog post from 2023 says "add this PPA to fix your wifi". Three questions to ask before typing it, using this lesson's trust ledger.
4. What can `snap revert` do that apt fundamentally can't, and which design choice makes it possible?

<details>
<summary>Answers</summary>

1. Auto-refresh wins for a browser (unpatched browsers are the #1 desktop attack surface); apt's wait-for-me wins for a production server mid-incident, where surprise restarts of anything are unwelcome.
2. No - snaps are read-only squashfs mounts, definitionally "full". Real disk pressure shows on `/` and friends; module 7 makes this reading fluent.
3. Who maintains it *now* (2023 was several Ubuntu releases ago)? Does it publish packages that shadow core system ones? Is the fix perhaps already in Ubuntu's own updates by now? (Any two of these earn the point - the reflex is the lesson.)
4. Instant rollback to the previous version - possible because old revisions stay on disk as intact, immutable images. apt overwrites files in place; going back means finding and reinstalling old .debs, config drift included.

</details>

## 📚 Further reading

- [snapcraft.io docs](https://snapcraft.io/docs) - channels, confinement, interfaces, from the source
- [Ubuntu Server docs: third-party repositories](https://documentation.ubuntu.com/server/how-to/software/third-party-apt-repositories/) - the official recipe, including the modern keyring handling

---

⬅️ [Previous: Under the hood](02-under-the-hood-dpkg-and-repos.md) · 🏠 [Course home](../README.md) · ➡️ [Next: From source](04-from-source.md)

# 2 · systemd and services - systemctl fluency

> **You'll learn:** to inspect, control, and create services - reading a systemctl status block like a dashboard, knowing exactly what enable does, and writing a unit file of your own.

## Why this matters

Everything that runs unattended - ssh, cron, the web server, unattended-upgrades from module 5 - is a systemd **service**, and `systemctl` is the switchboard for all of it. "Restart the service, check its status, make it start at boot" is the bread and butter of running any Linux machine, and the same four verbs work on every service ever written.

## The big picture

The atom of systemd is the **unit** - a named thing it manages, defined by an ini-style text file. Services are the star unit type, but the family is bigger:

| Unit type | Manages | Example |
|---|---|---|
| `.service` | a daemon or one-shot task | `ssh.service` |
| `.timer` | scheduled activation (lesson 4) | `apt-daily.timer` |
| `.target` | a named group/milestone (lesson 1) | `multi-user.target` |
| `.mount`, `.socket`, `.device` | mounts, sockets, hardware | `boot-efi.mount` |

And the one status block that tells you almost everything - learn to read it like an instrument panel:

```console
$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-07-09 08:12:44 NZST; 1 day 4h ago
   Main PID: 1247 (sshd)
      Tasks: 1 (limit: 18654)
     Memory: 6.2M (peak: 8.1M)
        CPU: 214ms
     CGroup: /system.slice/ssh.service
             └─1247 "sshd: /usr/sbin/sshd -D [listener]"
jul 10 11:02:33 mybox sshd[41200]: Accepted publickey for steve from 10.0.0.5...
```

Line by line: *Loaded* names the unit file and whether it's **enabled** (starts at boot); *Active* is the now-state with uptime; *Main PID* connects to module 4 (it's just a process!); *CGroup* lists every process belonging to the service; and the tail is its recent log - lesson 3's journal, pre-filtered for you.

## The verbs

```console
$ systemctl status ssh          # look (no sudo needed to look)
$ sudo systemctl start ssh      # start now
$ sudo systemctl stop ssh       # stop now
$ sudo systemctl restart ssh    # stop + start (drops connections)
$ sudo systemctl reload ssh     # re-read config WITHOUT restarting - when supported
$ sudo systemctl enable ssh     # start at every boot
$ sudo systemctl disable ssh    # don't
$ sudo systemctl enable --now ssh   # enable + start, one move
```

The distinction that trips everyone up for a week: **running and enabled are independent axes.** A service can be running but disabled (started by hand, gone after reboot) or stopped but enabled (will return at boot). `is-active` and `is-enabled` query one axis each - and "it came back after reboot!" or "it was down after reboot!" mysteries are always this table:

| | enabled | disabled |
|---|---|---|
| **active** | normal service | running until reboot |
| **inactive** | will start at boot | properly off |

Survey the whole machine:

```console
$ systemctl list-units --type=service            # what's loaded/active now
$ systemctl list-units --type=service --state=failed    # the red ones (also: systemctl --failed)
```

## Reading a unit file

```console
$ systemctl cat ssh.service      # the file, plus any overrides, with paths shown
```

```ini
[Unit]
Description=OpenBSD Secure Shell server
After=network.target auditd.service        # ordering: start me after these

[Service]
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS     # the actual command
Restart=on-failure                         # crash? bring it back
Type=notify

[Install]
WantedBy=multi-user.target                 # what "enable" hooks me onto
```

Three sections, three concerns: `[Unit]` is identity and ordering, `[Service]` is *how to run it* (the command, the restart policy, the user it runs as), `[Install]` is what enable attaches it to. `After=` orders, `Wants=`/`Requires=` pull dependencies in - lesson 1's parallel boot is systemd solving this graph.

To change a packaged service, never edit the file in `/usr/lib/systemd/system` (package upgrades overwrite it). Instead:

```console
$ sudo systemctl edit ssh       # opens a DROP-IN override; only your changed lines go in it
$ sudo systemctl daemon-reload  # after any unit-file change: re-read from disk
```

## Writing your own

A script you want managed - restarted on crash, logging to the journal, controllable with the same verbs - needs ~10 lines in `/etc/systemd/system/`:

```ini
# /etc/systemd/system/heartbeat.service
[Unit]
Description=Course heartbeat demo

[Service]
ExecStart=/home/steve/bin/heartbeat.sh
Restart=on-failure
User=steve

[Install]
WantedBy=multi-user.target
```

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now heartbeat
$ systemctl status heartbeat
```

That's the whole promotion path from module 3's scripts to real infrastructure: `nohup` was a trick; this is management.

<details>
<summary>🔍 Deep dive: enable is just a symlink (module 2 called it)</summary>

What does `enable` actually *do*? Watch it confess:

```console
$ sudo systemctl enable heartbeat
Created symlink '/etc/systemd/system/multi-user.target.wants/heartbeat.service'
  → '/etc/systemd/system/heartbeat.service'
```

A symlink, into a `.wants/` directory named after the `WantedBy=` target. At boot, systemd reaches `multi-user.target`, looks in its `.wants/` directory, and starts everything linked there. `disable` deletes the link. That's the entire mechanism - `ls /etc/systemd/system/multi-user.target.wants/` *is* the list of boot-enabled services, browsable with module 1 skills and decodable because module 2's lesson 4 taught you exactly what those `->` arrows mean.

(The `preset: enabled` in status blocks is the vendor's default for fresh installs - Ubuntu presets ssh to enabled on servers, so it's wired up the moment the package lands.)

</details>

## 🛠️ Try it

Run the full service lifecycle with your own creation:

1. Recon first: `systemctl --failed` (hopefully quiet), then pick a real service (`cron` is universal) and read its full status block aloud - state, enablement, PID, memory, last log lines.
2. Write the payload: `~/bin/heartbeat.sh` - an infinite loop logging `heartbeat $(date)` every 10 seconds (`while true; do echo "heartbeat $(date)"; sleep 10; done` - module 3 skills). chmod +x, test it, Ctrl+C.
3. Write `/etc/systemd/system/heartbeat.service` as above (your username, your path), `daemon-reload`, then `start` it (don't enable yet). Check status - find your loop's PID in the CGroup.
4. Prove the axes are independent: reboot-free version - `is-active` vs `is-enabled` now; then `enable` it and re-check both; then `stop` it and re-check again. Three different combinations visited.
5. Prove `Restart=on-failure` earns its keep: `sudo kill -9 <MainPID>` (module 4's rudest signal), wait a beat, `systemctl status heartbeat` - new PID?
6. Find the deep dive's symlink with your own ls. Then full cleanup: `disable --now`, delete the unit file, `daemon-reload`, and confirm `status` now says not-found.

<details>
<summary>💡 Hint 1</summary>

Step 5: kill -9 counts as failure, so systemd restarts it (default RestartSec is 100ms). If you'd used plain `systemctl stop`, no restart - systemd distinguishes *you stopping it* from *it dying*. That distinction is the whole point of the setting.

</details>

<details>
<summary>✅ Solution</summary>

```console
$ printf '#!/bin/bash\nwhile true; do echo "heartbeat $(date)"; sleep 10; done\n' > ~/bin/heartbeat.sh
$ chmod +x ~/bin/heartbeat.sh
$ sudo nano /etc/systemd/system/heartbeat.service     # the 10 lines from the lesson
$ sudo systemctl daemon-reload && sudo systemctl start heartbeat
$ systemctl status heartbeat                          # active (running), your PID in CGroup
$ systemctl is-active heartbeat; systemctl is-enabled heartbeat    # active / disabled
$ sudo systemctl enable heartbeat                     # watch the symlink line print!
$ systemctl is-enabled heartbeat                      # enabled
$ sudo systemctl stop heartbeat && systemctl is-active heartbeat   # inactive (but still enabled)
$ sudo systemctl start heartbeat
$ sudo kill -9 "$(systemctl show -p MainPID --value heartbeat)"    # 5
$ sleep 1 && systemctl status heartbeat               # active again, NEW MainPID
$ ls -l /etc/systemd/system/multi-user.target.wants/ | grep heartbeat   # 6: the symlink
$ sudo systemctl disable --now heartbeat
$ sudo rm /etc/systemd/system/heartbeat.service && sudo systemctl daemon-reload
$ systemctl status heartbeat                          # Unit heartbeat.service could not be found.
```

</details>

## ✋ Checkpoint

1. Nightly reboot, and nginx - installed months ago, working yesterday - is down this morning. `systemctl is-active` says inactive, `is-enabled` says disabled. Reconstruct what probably happened, and give the one command that prevents the rerun.
2. Predict: `restart` vs `reload` on a busy ssh server - which one kicks off every connected user, and why does reload even exist?
3. Your service edit to `/usr/lib/systemd/system/foo.service` vanished after `apt upgrade`. What should you have done, in two commands?
4. From the deep dive: without running systemctl at all, how do you list every service that will start with `multi-user.target`?

<details>
<summary>Answers</summary>

1. Someone started it by hand (active) but never enabled it - it lived in the running-but-disabled quadrant until the reboot cleared it. `sudo systemctl enable nginx` (or `enable --now` next time it's started manually).
2. `restart` stops the daemon - connections die. `reload` sends the config-reread signal (often SIGHUP - module 4!) to the running process; sessions survive. Reload exists precisely for config changes on stateful services.
3. `sudo systemctl edit foo` (drop-in override survives upgrades), then `sudo systemctl daemon-reload`. Package-owned files are the package's to overwrite - module 5 taught whose files those are.
4. `ls /etc/systemd/system/multi-user.target.wants/` - enablement is symlinks in the target's `.wants/` directory (plus the preset-enabled ones linked from `/usr/lib/systemd/system/multi-user.target.wants/`, for full credit).

</details>

## 📚 Further reading

- `man systemctl` and `man systemd.service` - the verb list and the [Service] options catalogue
- [systemd.io](https://systemd.io/) - the project's own explanations; the "About" pages beat most tutorials

---

⬅️ [Previous: From power button to login](01-from-power-button-to-login.md) · 🗺️ [Course map](../README.md) · ➡️ [Next: journald - one log to rule them all](03-journald.md)

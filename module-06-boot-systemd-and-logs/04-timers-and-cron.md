# 4 · Timers and cron - running things on a schedule

> **You'll learn:** to schedule work both ways Ubuntu offers - classic crontab lines and systemd timers - and to pick the right one per job.

## Why this matters

Backups at 02:00, cleanup weekly, a health check every five minutes - unattended scheduling is half the point of running servers. Ubuntu ships *two* schedulers side by side: venerable cron and systemd timers. You'll write both, and you'll inherit both from previous admins, so fluency in each is table stakes.

## The big picture

Your machine is already busy at night - look:

```console
$ systemctl list-timers --no-pager | head -6
NEXT                        LEFT     LAST                        PASSED  UNIT                 ACTIVATES
Fri 2026-07-10 15:04:11     2h 51min Thu 2026-07-09 15:02:44     23h ago apt-daily.timer      apt-daily.service
Fri 2026-07-10 06:14:00     17h      Fri 2026-07-10 06:14:02     6h ago  logrotate.timer      logrotate.service
...
```

There's module 5's unattended upgrades (`apt-daily`) and lesson 3's log rotation, both running as timers. The two systems at a glance:

| | cron | systemd timer |
|---|---|---|
| Define a job | one line in a crontab | two files: `.timer` + `.service` |
| Logs | mailed (usually nowhere) unless you redirect | in the journal, per unit, free |
| Missed while powered off | skipped, gone | `Persistent=true` runs it at next boot |
| Environment | nearly empty (module 3's famous trap) | the unit's own, declared |
| Best for | quick personal jobs, one-liners | anything you'd debug at 3am |

## crontab: the sacred five fields

Each user has a crontab; `crontab -e` edits yours (first run asks for an editor - module 3's `$EDITOR` in the wild):

```text
# ┌ minute (0-59)    ┌ hour (0-23)   ┌ day of month   ┌ month   ┌ day of week (0=Sun)
# │                  │               │                │         │
  30                 2               *                *         *      /home/steve/bin/backup.sh
  */5                *               *                *         *      /home/steve/bin/healthcheck.sh
  0                  9               *                *         1-5    /home/steve/bin/standup-reminder.sh
```

Reading practice: 02:30 daily; every 5 minutes; 09:00 weekdays. The grammar: `*` any, `*/5` every 5th, `1-5` range, `1,15` list. `crontab -l` lists, and system-wide schedules live in `/etc/crontab` and `/etc/cron.d/` (those add a *user* field), with the lazy-but-fine `/etc/cron.daily/` script-drop directories alongside.

> [!WARNING]
> The cron environment is nearly empty: minimal PATH, no aliases, no `.bashrc` - module 3's deep dive planted this flag and here is the battlefield. The three rules of surviving cron: absolute paths in the command, absolute paths *inside* the script, and capture output yourself: `30 2 * * * /home/steve/bin/backup.sh >> /home/steve/backup.log 2>&1` - because cron's default is to *mail* output to a mailbox nobody has read since 1998.

## systemd timers: scheduling with the full toolkit

A timer is a unit (lesson 2's whole vocabulary applies) that activates a service on schedule. Two small files replace the cron line:

```ini
# /etc/systemd/system/report.service        (what to run)
[Unit]
Description=Nightly course report
[Service]
Type=oneshot
ExecStart=/home/steve/bin/report.sh
User=steve
```

```ini
# /etc/systemd/system/report.timer          (when to run it)
[Unit]
Description=Run report nightly at 02:30
[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true                             # missed while off? run at next boot
[Install]
WantedBy=timers.target
```

```console
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now report.timer     # the TIMER gets enabled, not the service
$ systemctl list-timers report.timer           # NEXT/LAST at a glance
$ journalctl -u report.service                 # every run's output, forever (lesson 3)
```

`OnCalendar` reads as `weekday year-month-day hour:minute:second` with `*` wildcards - `Mon *-*-* 09:00:00`, `*-*-01 00:00:00` (monthly), or friendly words: `daily`, `weekly`, `hourly`. Validate any expression before trusting it:

```console
$ systemd-analyze calendar "Mon..Fri 09:00"    # parses it AND prints the next elapse time
```

The payoff for the extra file: journal logging, `Persistent=true` catch-up, dependencies (`After=network-online.target` for a job that needs the net), resource limits, and `systemctl start report.service` as a *manual test run* identical to the scheduled one - the debugging gift cron never gives you.

<details>
<summary>🔍 Deep dive: user timers - scheduling without sudo, and the other tricks</summary>

Everything above used system units; there's a personal tier. `systemctl --user` manages units in `~/.config/systemd/user/` - same file formats, no sudo, running as you:

```console
$ mkdir -p ~/.config/systemd/user
$ # drop report.service + report.timer there (no User= line needed)
$ systemctl --user daemon-reload && systemctl --user enable --now report.timer
$ journalctl --user -u report.service
```

One catch: user units normally run only while you're logged in - `loginctl enable-linger $USER` keeps your user manager alive around the clock, which is the standard move for personal servers.

Two more `[Timer]` tricks worth knowing exist: `OnBootSec=5min` / `OnUnitActiveSec=10min` give "10 minutes after the last run" interval scheduling, and `RandomizedDelaySec=30min` spreads fleet-wide thundering herds - it's why Ubuntu's apt-daily fires at a different minute on every machine. And for one-shot deferred jobs ("remind me in 2 hours"), the tiny classic `at` still works: `echo 'command' | at now + 2 hours`.

</details>

## 🛠️ Try it

Schedule the same heartbeat both ways, then judge them side by side:

1. Cron first: `crontab -e`, add a line running `date >> /home/<you>/linux-course/exercises/cron-beat.txt` every minute (`* * * * *` - full absolute paths!). Wait two minutes, confirm two lines, then `crontab -e` again and remove it.
2. While it beats: find cron's own record of running your job in the journal (`journalctl -u cron -n 5` - note the job runs but its *output* isn't there; you had to redirect it yourself).
3. Timer version: write `beat.service` + `beat.timer` (user units, deep-dive style - no sudo) with `OnCalendar=*-*-* *:*:00` (every minute) and `Persistent=true`, running `echo "beat $(date)"`. Enable, wait two minutes, read the output with `journalctl --user -u beat.service`.
4. The comparison that settles it: for each system, answer from your own machine - where's the output? could you test-run the job manually? what happens to a beat missed while powered off?
5. Validate before you schedule: `systemd-analyze calendar "Mon..Fri 09:00"` and two expressions of your own invention - one for "monthly, 1st, midnight", one deliberately wrong to see the error.
6. Cleanup: `systemctl --user disable --now beat.timer`, delete both files, `--user daemon-reload`. Confirm `list-timers` (user) is quiet.

<details>
<summary>💡 Hint 1</summary>

Step 3: service file is 5 lines (`[Unit]` Description, `[Service]` Type=oneshot + ExecStart), timer is 7. ExecStart needs an absolute path even for echo: `/usr/bin/echo` - or point it at a script in your `~/bin`. If the user timer won't fire, check `systemctl --user status beat.timer` first.

</details>

<details>
<summary>✅ Solution</summary>

```console
$ crontab -e        # add:  * * * * * date >> /home/steve/linux-course/exercises/cron-beat.txt
$ sleep 120 && wc -l ~/linux-course/exercises/cron-beat.txt    # 1: ≥2 lines
$ journalctl -u cron -n 5                                      # 2: "(steve) CMD (date >> ...)" - execution logged, output not
$ crontab -e        # remove the line
$ mkdir -p ~/.config/systemd/user
$ cat > ~/.config/systemd/user/beat.service <<'EOF'
[Unit]
Description=beat demo
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'echo "beat $(date)"'
EOF
$ cat > ~/.config/systemd/user/beat.timer <<'EOF'
[Unit]
Description=beat every minute
[Timer]
OnCalendar=*-*-* *:*:00
Persistent=true
[Install]
WantedBy=timers.target
EOF
$ systemctl --user daemon-reload && systemctl --user enable --now beat.timer   # 3
$ sleep 120 && journalctl --user -u beat.service -n 4          # timestamped beats, zero redirect plumbing
$ systemd-analyze calendar "*-*-01 00:00:00"                   # 5: monthly - shows next elapse
$ systemd-analyze calendar "Feb 30"                            # deliberately never - watch it say so
$ systemctl --user disable --now beat.timer                    # 6
$ rm ~/.config/systemd/user/beat.{service,timer} && systemctl --user daemon-reload
```

Step 4 answers: cron output lands wherever *you* redirected (or vanishes into mail); timer output is in the journal automatically. Timer jobs test-run with `systemctl --user start beat.service`; cron lines you test by... editing the schedule and waiting. Missed-while-off: cron skips silently; the timer's `Persistent=true` catches up at boot.

</details>

## ✋ Checkpoint

1. Read these cron fields aloud: `0 3 * * 0`, `*/15 9-17 * * 1-5`, `30 4 1 * *`.
2. A cron job works when you paste the command in your terminal but produces nothing from cron - no output file, no errors anywhere. Give the two most likely causes and the diagnostic first step, all from this module.
3. A nightly 02:00 backup on a laptop that's usually asleep at 02:00: which scheduler, which single directive, and why?
4. Predict: you `systemctl enable report.service` (not the timer) believing you've scheduled it. What actually happens at 02:30, and at boot?

<details>
<summary>Answers</summary>

1. Sundays 03:00; every 15 minutes during 09:00-17:59, weekdays; 04:30 on the 1st of every month.
2. The empty-environment trap (relative paths / PATH-dependent commands failing silently) and unredirected output going to mail. First step: check the job *ran* at all - `journalctl -u cron` - then wrap it as `... >> /tmp/debug.log 2>&1` and read the truth.
3. systemd timer with `Persistent=true` - a missed 02:00 runs at next wake/boot; cron would just skip it, nightly, forever.
4. Nothing at 02:30 - no timer is enabled. At boot, the *service* runs once (it's now hooked to multi-user.target) and never again. Enable the `.timer`; the service stays unenabled, activated only by the timer or by hand.

</details>

## 📚 Further reading

- `man 5 crontab` - the format (module 1's section-5 lesson, still paying off)
- `man systemd.timer` and `man systemd.time` - every [Timer] directive, and the OnCalendar grammar in full
- [crontab.guru](https://crontab.guru/) - paste any five fields, read them in English

---

⬅️ [Previous: journald](03-journald.md) · 🗺️ [Course map](../README.md) · ➡️ Next module: [Networking & Storage](../module-07-networking-and-storage/README.md)

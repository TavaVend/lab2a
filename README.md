1. Both unit files, final versions

ini
# disk-report.service
[Unit]
Description=Append disk usage to log
Documentation=man:df(1)
After=local-fs.target

[Service]
Type=oneshot
User=reports
ExecStart=/usr/local/bin/disk-report.sh
StandardOutput=append:/var/log/disk-report.log
StandardError=append:/var/log/disk-report.log
ini
# disk-report.timer
[Unit]
Description=Run disk-report every five minutes
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target

2. Initial journal error, annotated

Sep 25 06:05:02 UbuntuServer disk-report.sh[1826]: /usr/local/bin/disk-report.sh: line 2: /var/log/disk-report.log: Permission denied
Sep 25 06:05:02 UbuntuServer disk-report.sh[1828]: /usr/local/bin/disk-report.sh: line 3: /var/log/disk-report.log: Permission denied
Sep 25 06:05:02 UbuntuServer systemd[1]: disk-report.service: Main process exited, code=exited, status=1/FAILURE

The error said the reports user was denied permission to write to /var/log/disk-report.log. This confirmed the root cause: the file had earlier been created by root during manual testing, so reports — a non-root, non-owning user — had no write access to it.

3. Why Option B is better than Option A (one paragraph)

Option A (chowning the file to reports:reports) works, but it's fragile — if the log file is ever deleted, /var/log itself is still owned by root, so the service breaks again the next time it tries to create the file. Option B moves the write logic out of the script and into the unit file (StandardOutput=append:), so systemd — running as root — opens the file on the service's behalf. This means reports never needs write permission on /var/log at all. It's the least-privilege approach: the script doesn't even need to know a file path, and the output destination can be changed just by editing the unit, without touching the script.

4. systemctl list-timers disk-report.timer output

NEXT                        LEFT   LAST                          PASSED       UNIT                ACTIVATES
Sat 2026-09-26 00:00:00 UTC 17h    Fri 2026-09-25 06:10:00 UTC   1min 34s ago  disk-report.timer   disk-report.service

5. Two lines showing successful runs

Sep 25 06:07:49 UbuntuServer systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 25 06:09:03 UbuntuServer systemd[1]: Finished disk-report.service - Append disk usage to log.

6. Two sentences — why reports was created with those flags, and what an attacker would gain otherwise

The reports user was created with --no-create-home and --shell /usr/sbin/nologin because it's a service account whose only job is running one script — it never needs interactive login or a personal home directory. If those flags were omitted, an attacker who compromised the reports account could log in interactively via a shell and would have a writable home directory to stage files or persist in, expanding their reach well beyond the single log file the account is meant to own.

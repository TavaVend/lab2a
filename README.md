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

# disk-report.timer
[Unit]
Description=Run disk-report every five minutes
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target

2. Algne journal'i veateade, kommenteeritud

Sep 25 06:05:02 UbuntuServer disk-report.sh[1826]: /usr/local/bin/disk-report.sh: line 2: /var/log/disk-report.log: Permission denied
Sep 25 06:05:02 UbuntuServer disk-report.sh[1828]: /usr/local/bin/disk-report.sh: line 3: /var/log/disk-report.log: Permission denied
Sep 25 06:05:02 UbuntuServer systemd[1]: disk-report.service: Main process exited, code=exited, status=1/FAILURE

3. Miks Option B on parem kui Option A (üks lõik)

Option A (chown) töötab, kuid on habras — kui logifail kunagi kustutatakse, kuulub /var/log ikka root'ile ja teenus katkeb uuesti. Option B viib kirjutamise loogika skriptist unit-faili sisse (StandardOutput=append:), nii et systemd (root'ina) avab faili teenuse eest ise — reports ei vaja kunagi /var/log kirjutusõigust. See on kõige vähemõiguste (least-privilege) lahendus: skript ei pea faili teed teadmagi ja sihtkohta saab muuta ainult unit-faili muutes.

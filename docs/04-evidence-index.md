# Evidence Index

| ID | File | What it supports | Limitation |
| --- | --- | --- | --- |
| E01 | [Failed SSH attempts](../assets/evidence/01-failed-ssh-attempts.png) | Repeated failures from `192.168.161.198` against several usernames | Does not expose attempted passwords |
| E02 | [Full login record](../assets/evidence/02-full-login-record.png) | A `19:41:10` session from the suspicious source | Not the earliest accepted authentication |
| E03 | [Accepted password in auth.log](../assets/evidence/03-accepted-password-authlog.png) | First confirmed accepted password at `19:39:03.232692+02:00` | Cropped to the decisive log line; surrounding context was reviewed separately |
| E04 | [Bash history](../assets/evidence/04-shell-history.png) | Discovery, LaZagne, SCP, and `sys_monitor.sh` execution | Shell history can be altered; corroborate key actions |
| E05 | [Script SHA-1](../assets/evidence/05-script-sha1.png) | Maps a file named `sys_monitor.sh` to its SHA-1 | Precomputed inventory path differs from the `/tmp` execution path; script was not re-hashed |
| E06 | [Systemd unit](../assets/evidence/06-systemd-service.png) | Root execution, restart behavior, deceptive service name | Configuration alone does not prove start |
| E07 | [Systemd logs](../assets/evidence/07-systemd-service-logs.png) | Corroborates service start and restart behavior shortly before repeated account-creation attempts | Dense log screenshot; timing supports correlation rather than recovered script content |
| E08 | [Authorized-key identity](../assets/evidence/08-authorized-key-identity.png) | Correlates `kali@kali` with the SCP destination | Key comments are attacker-controlled text |
| E09 | [Shell-startup listeners](../assets/evidence/09-shell-startup-listeners.png) | Root `.bashrc` configured Ncat on port `7575`; `.profile` configured port `9000` | Proves configuration, not successful listener execution |
| E10 | [Local user creation](../assets/evidence/10-local-user-creation.png) | Exact `Regev` creation event and privileged groups | Later failed repeats are unrelated to the first success |
| E11 | [Cron payload fetch](../assets/evidence/11-cron-payload-fetch.png) | Intended scheduled download/decode/execute chain | Presence does not prove execution |
| E12 | [Cron syntax failure](../assets/evidence/12-cron-syntax-failure.png) | Cron ignored the file because of a bad minute field | Does not rule out manual execution |
| E13 | [Decoded payload](../assets/evidence/13-decoded-payload.png) | Intended collection and HTTP POST of password files | Static content proves capability, not execution |

Each image is included because it answers a specific evidentiary question. Decorative, duplicate, and dead-end screenshots were intentionally excluded.

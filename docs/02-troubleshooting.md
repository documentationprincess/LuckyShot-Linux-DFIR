# Troubleshooting and Analytical Corrections

Good incident analysis includes the wrong turns, not just the final answers. These were the points where I revised the working theory or limited a claim because the evidence did not support it.

## 1. USB Presence Did Not Mean USB Compromise

**What I was trying to determine:** Whether removable media explained the missing and modified files.

**What happened:** The collection contained USB inventory, but the listed devices were VMware virtual devices and Linux root hubs. There was no device, mount, file-transfer, or execution evidence tying removable media to the incident.

**How I resolved it:** I rejected USB as the supported access path and moved to authentication evidence.

**How I validated the correction:** `lastb` and `auth.log` established a coherent SSH credential-attack sequence from one source address.

## 2. The First Login Time Was Earlier Than `last` Suggested

**What I was trying to determine:** The first successful attacker login.

**What happened:** The summarized login records pointed to `19:41`, and `last -F` refined that event to `19:41:10`. Neither was the earliest accepted authentication.

**How I resolved it:** I treated session accounting as a lead and pivoted to the SSH authentication log.

**How I validated the correction:** `auth.log` recorded `Accepted password` for `administrator` at `2025-02-10T19:39:03.232692+02:00`, followed by the session-open event.

## 3. The Script Name Matched, but the Paths Did Not

**What I was trying to determine:** The SHA-1 of `sys_monitor.sh`.

**What happened:** Bash history and the systemd unit referenced `/tmp/sys_monitor.sh`, but the supplied SHA-1 inventory mapped the same filename under `/home/administrator/tmp/`.

**How I resolved it:** I reported the supplied SHA-1 as a strong association instead of claiming that I re-hashed the exact executed file.

**How I validated the limitation:** The expected `/tmp` copy was not recovered, so the path discrepancy remains visible in the final assessment.

## 4. A Cron Artifact Did Not Prove Cron Execution

**What I was trying to determine:** Whether the scheduled payload ran.

**What happened:** `/etc/cron.d/syscheck` contained a download, reverse, decode, and shell-execution chain. Its schedule began with invalid minute syntax.

**How I resolved it:** I checked daemon logs instead of assuming that the file worked.

**How I validated the correction:** Syslog reported `bad minute` and explicitly stated that the crontab file was ignored. The artifact is therefore documented as an attempted mechanism, not successful scheduled execution.

## 5. A Screenshot Gap Was Resolved From the Original Notes

**What I was trying to determine:** Which root shell startup file configured the lowest listener port.

**What happened:** The first PDF export did not expose the decisive comparison clearly enough.

**How I resolved it:** I returned to the original screenshot export rather than guessing from the narrative.

**How I validated the correction:** The recovered capture shows `.bashrc` configuring port `7575` and `.profile` configuring port `9000`.

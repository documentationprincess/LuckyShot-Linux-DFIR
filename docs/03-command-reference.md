# Command Reference

These commands were used to inspect the supplied forensic artifacts. Paths beginning with `[root]` refer to the mounted filesystem included in the lab evidence.

## Login and Authentication

```bash
cat live_response/system/lastb.txt
cat live_response/system/last.txt
cat live_response/system/last_-a_-F.txt

grep -nE 'Accepted (password|publickey)|session opened for user' \
  '[root]/var/log/auth.log'
```

Purpose:

- `lastb` identifies failed logins.
- `last` and `last -F` summarize sessions.
- `auth.log` provides the authoritative accepted-password event with microsecond precision.

## Shell History

```bash
find '[root]/home' -type f \
  \( -name '.bash_history' -o -name '.zsh_history' \) -print

cat '[root]/home/administrator/.bash_history'
```

Purpose: Reconstruct discovery, tool transfer, credential access, exfiltration, and script execution.

## Malicious Script and Hash

```bash
find '[root]' -type f -name 'sys_monitor.sh' -print 2>/dev/null

grep -Rni 'sys_monitor\.sh' \
  '[root]' live_response 2>/dev/null

grep -i 'sys_monitor' \
  hash_executables/hash_executables.sha1
```

Purpose: Search for the deleted or relocated script, then pivot to the supplied SHA-1 inventory.

## Systemd Persistence

```bash
cat '[root]/etc/systemd/system/systemd-networkm.service'

grep -Rni 'systemd-networkm' \
  '[root]/etc/systemd' '[root]/var/log' 2>/dev/null
```

Purpose: Confirm the unit configuration and correlate it with service-start events.

## SSH Key Correlation

```bash
awk '{print $NF}' '[root]/root/.ssh/authorized_keys'

grep 'scp ' '[root]/home/administrator/.bash_history'
```

Purpose: Compare the authorized-key comment with the SCP destination identity.

## Local Account Creation

```bash
grep -nE 'useradd.*Regev|new user: name=Regev' \
  '[root]/var/log/auth.log'
```

Purpose: Distinguish the `useradd` invocation from the successful account-creation event.

## Shell Startup Persistence

```bash
grep -RniE 'nc |netcat|ncat|socat|TCP-LISTEN|[0-9]{2,5}' \
  '[root]/root/.bashrc' \
  '[root]/root/.profile' \
  '[root]/root/.bash_profile' \
  '[root]/root/.bash_login' 2>/dev/null

tail -n 8 '[root]/root/.bashrc'
cat '[root]/root/.profile'
```

Purpose: Enumerate network-listener commands in root shell initialization files, compare their configured ports, and identify `.bashrc` as the file using the lowest port.

## Cron Artifact and Execution Status

```bash
cat '[root]/etc/cron.d/syscheck'

grep -n 'syscheck' '[root]/var/log/syslog'
```

Purpose: Compare intended scheduled behavior with the cron daemon's actual parsing result.

## Safe Payload Decoding

Do not pipe an unknown remote payload directly into Bash. Save and inspect it:

```bash
curl -fsSL 'https://pastebin.com/raw/SAuEez0S' \
  --output fetched-payload.txt

rev fetched-payload.txt |
  base64 -d > decoded-payload.sh

sed -n '1,200p' decoded-payload.sh
sha256sum fetched-payload.txt decoded-payload.sh
```

Purpose: Preserve the fetched and decoded content for static analysis without executing it.

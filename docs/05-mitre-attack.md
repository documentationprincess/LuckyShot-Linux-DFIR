# MITRE ATT&CK Mapping

The mapping distinguishes behavior observed in the evidence from behavior that was configured but did not execute through the documented mechanism.

| Status | Tactic | Technique | Evidence-based rationale |
| --- | --- | --- | --- |
| Observed | Credential Access | [T1110.001 - Password Guessing](https://attack.mitre.org/techniques/T1110/001/) | Repetitive SSH attempts against several usernames preceded a successful password authentication |
| Observed | Initial Access / Persistence | [T1078.003 - Local Accounts](https://attack.mitre.org/techniques/T1078/003/) | The local `administrator` account was used for remote access |
| Observed | Lateral Movement | [T1021.004 - SSH](https://attack.mitre.org/techniques/T1021/004/) | The attacker opened a remote shell through SSH |
| Observed | Discovery | [T1087.001 - Local Account](https://attack.mitre.org/techniques/T1087/001/) | `groups`, `/etc/passwd`, and `getent group sudo` were used to inspect accounts and privileges |
| Observed | Command and Control | [T1105 - Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/) | Git and curl transferred LaZagne and another credential-recovery script to the host |
| Observed | Credential Access | [T1555 - Credentials from Password Stores](https://attack.mitre.org/techniques/T1555/) | LaZagne was executed with its broad credential-recovery mode |
| Observed | Exfiltration | [T1048.002 - Asymmetric Encrypted Non-C2 Protocol](https://attack.mitre.org/techniques/T1048/002/) | Credential files were copied to the attacker host with SCP |
| Observed | Execution | [T1059.004 - Unix Shell](https://attack.mitre.org/techniques/T1059/004/) | The attacker executed commands and scripts through Bash |
| Observed | Persistence / Privilege Escalation | [T1543.002 - Systemd Service](https://attack.mitre.org/techniques/T1543/002/) | `systemd-networkm.service` launched the malicious script as root and restarted automatically |
| Observed | Persistence / Privilege Escalation | [T1098.004 - SSH Authorized Keys](https://attack.mitre.org/techniques/T1098/004/) | An attacker-associated key was present in root's `authorized_keys` |
| Observed configuration | Persistence / Privilege Escalation | [T1546.004 - Unix Shell Configuration Modification](https://attack.mitre.org/techniques/T1546/004/) | Root `.bashrc` and `.profile` were configured to launch Ncat listeners on ports `7575` and `9000` |
| Observed | Persistence | [T1136.001 - Local Account](https://attack.mitre.org/techniques/T1136/001/) | `Regev` was created and added to `sudo` and `adm` |
| Attempted; cron rejected it | Execution / Persistence | [T1053.003 - Cron](https://attack.mitre.org/techniques/T1053/003/) | `/etc/cron.d/syscheck` contained a scheduled payload chain, but cron ignored the invalid schedule |
| Payload content observed | Defense Evasion | [T1140 - Deobfuscate/Decode Files or Information](https://attack.mitre.org/techniques/T1140/) | The command reversed and Base64-decoded fetched content |
| Payload content observed | Credential Access | [T1003.008 - `/etc/passwd` and `/etc/shadow`](https://attack.mitre.org/techniques/T1003/008/) | The decoded payload read the Linux password and shadow files |
| Payload content observed | Exfiltration | [T1048.003 - Unencrypted Non-C2 Protocol](https://attack.mitre.org/techniques/T1048/003/) | The decoded payload submitted encoded credential data through HTTP POST |

## Mapping Notes

- **Password guessing vs. password spraying:** The logs show usernames and outcomes, not the passwords attempted. Password guessing is supported; password spraying would require evidence that the same password was reused across multiple accounts.
- **SCP:** SCP uses SSH's public-key cryptography and is represented here as encrypted exfiltration over a non-C2 protocol.
- **Cron:** The technique is marked attempted because the artifact was present, but syslog showed that cron rejected it.
- **Shell startup files:** The modified files prove the persistence configuration; the available evidence does not independently confirm a successful listener bind.
- **Decoded payload:** The commands demonstrate intended capability. They are not treated as proof that cron successfully ran them.

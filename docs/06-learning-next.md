# What to Learn Next

This list is follow-up work, not a prerequisite for publishing the finished project.

## 1. Linux Authentication Artifacts

Learn the difference between:

- `/var/log/auth.log`
- `wtmp` and `last`
- `btmp` and `lastb`
- Session creation vs. successful authentication
- Local timezone offsets and timestamp precision

**Mini exercise:** Explain why `last` showed `19:41` while `auth.log` established an earlier successful authentication at `19:39:03`.

## 2. Linux Persistence Locations

Review:

- Systemd unit files
- Cron directories and crontabs
- SSH `authorized_keys`
- `.bashrc`, `.profile`, and other shell startup files
- Privileged local account creation

**Mini exercise:** Build a one-page comparison of what triggers each persistence method and which logs can validate execution.

## 3. Bash and Safe Payload Analysis

Practice:

- Pipes and redirection
- `grep` regular expressions
- `find`
- `rev` and Base64 decoding
- Saving remote content before inspecting it

**Mini exercise:** Decode a harmless sample into a file and calculate hashes before and after decoding.

## 4. Detection Engineering

Write two simple detection ideas:

1. Several failed SSH usernames from one source followed by a successful login.
2. `useradd` creating an account that is immediately placed in `sudo` or `adm`.

Then identify the required data source, fields, time window, likely false positives, and triage steps.

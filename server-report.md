# Server Overload and Malware Cleanup Plan

The VPS at `2.24.64.92` is currently overloaded (800% CPU usage) due to unauthorized cryptocurrency mining software and backdoor processes installed by an attacker. 

This plan outlines the steps to terminate the mining processes, remove persistence mechanisms (backdoor users and cron jobs), and clean up the malicious files to restore server health and secure the system.

## User Review Required

> [!WARNING]
> We will be removing backdoor accounts (`pakchoi` and `sysupdate`) and cleaning up unauthorized cron jobs. 
> 
> Regarding SSH Keys:
> There are currently two SSH keys in `/root/.ssh/authorized_keys`. We will:
> 1. Back up `/root/.ssh/authorized_keys` to `/root/authorized_keys.bak`.
> 2. Keep only the legitimate public key (if known) or disable SSH key login temporarily if the user only uses password authentication. 
> Please let us know if you use a specific SSH key to access the VPS so we don't accidentally remove it.

## Open Questions

- **Do you use SSH keys to log in as root?** If yes, please paste your public key so we can preserve it. Otherwise, we will keep a backup of the current keys in `/root/authorized_keys.bak` but clean up the active authorized keys file if we identify malicious keys.
- **Were you running Node.js or database services on ports 3000, 3001, 4000, or 4003?** Nginx is currently configured to proxy domains (`cryptell.com`, `super-trader.xyz`, `influner.com`) to those local ports, but no processes are currently listening on them. If they were compromised web applications, they might be the entry point for the attacker.

## Proposed Changes

We will execute commands over SSH to clean up the VPS:

### 1. Stop Malicious Processes
- Terminate the CPU-hogging daemon `/usr/bin/dhpcd` (PID `1280602` or latest).
- Terminate the miner process `https` (PID `2010`) and its parent shell `/bin/busybox` / `sh run.sh` (PID `1993`).

### 2. Remove Persistent Cron Jobs
- Clear the crontab for `root` (which runs `dhpcd` hourly and previously tried to run a script in `/tmp/.est1/`).
- Clear the crontab for `sysupdate` (which runs `/usr/lib/systemd/.cache/update-check` every 20 minutes).

### 3. Remove Backdoor Users and Privileges
- Delete user `pakchoi` (UID 0 root backdoor account).
- Delete user `sysupdate` (UID 1337 backdoor account).
- Delete the sudoers rule `/etc/sudoers.d/.systemd-resolved` which grants `sysupdate` passwordless root access.
- Delete `/home/pakchoi` and `/home/sysupdate` directories.

### 4. Delete Malicious Files
- Remove `/usr/bin/dhpcd` (and `/bin/dhpcd` if it's not a symlink).
- Remove `/usr/lib/systemd/.cache/update-check` if it gets recreated.
- Inspect `/tmp` and clean up any remaining malicious scripts/folders.

---

## Verification Plan

### Automated / CLI Verification
- Run `ps aux --sort=-%cpu | head -n 10` to verify CPU usage is back to normal (< 5% for idle).
- Check `crontab -l` and `crontab -u sysupdate -l` to ensure they are empty/clean.
- Verify `id pakchoi` and `id sysupdate` return user not found.
- Verify `/etc/sudoers.d/.systemd-resolved` and `/usr/bin/dhpcd` are deleted.

# Troubleshooting log

Problems I hit while building the hardened server, with the cause and the fix for each.

## 1. `No route to host` and `Connection refused` after a reboot

**Symptom:** `ssh debian13` failed right after restarting my computer.
**Cause:** The VM and the libvirt `default` network were not running yet. `Connection refused` can also mean a fail2ban ban, so I checked that as well from the console.
**Fix:**
```bash
virsh net-list --all          # network must be active
virsh start <vm-name>
virsh autostart <vm-name>     # start with the host from now on
```

## 2. SSH terminal froze during the fail2ban test

**Symptom:** After a few failed logins, my open SSH session stopped responding, while the virt-manager console still worked.
**Cause:** fail2ban banned the host's IP (`192.168.122.1`). A ban blocks port 22 for that address, so it also cuts sessions that were already open. The virt-manager console talks to the VM through the hypervisor and not through the network, so the firewall does not affect it.
**Fix:** Unbanned from the console:
```bash
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip 192.168.122.1
```
**Lesson:** Always keep a console session open when testing bans or firewall rules.

## 3. `ssh -t host 'sudo ...' > file` hung forever

**Symptom:** Saving the UFW status to a file never finished.
**Cause:** The redirect sent everything to the file, including the `sudo` password prompt, so the command waited for a password I could not see.
**Fix:** Created the file on the VM, then copied it:
```bash
ssh debian13
sudo ufw status verbose > ~/ufw-status.txt
exit
scp debian13:ufw-status.txt configs/ufw-status.txt
```

## 4. `nmap` reported "Host seems down"

**Symptom:** `nmap 192.168.122.50` said the host was down, but SSH worked.
**Cause:** Without root, nmap checks whether a host is alive by probing ports 80 and 443. The firewall silently drops those probes, so nmap assumed the host was off. This is the firewall working as designed.
**Fix:**
```bash
nmap -Pn 192.168.122.50
```

## 5. Logging in as the wrong user

**Symptom:** `ssh debian13` tried to log in as the install user, not `sysadmin`.
**Cause:** `~/.ssh/config` had two `User` lines under one `Host`, and SSH uses the first value it finds.
**Fix:** Split it into two `Host` entries, one per user.

## 6. Default SSH key was offered instead of the lab key

**Symptom:** A plain `ssh sysadmin@...` asked for the account password.
**Cause:** SSH only tried my default key, not the dedicated lab key.
**Fix:** Added `IdentityFile ~/.ssh/id_ed25519_lab` and `IdentitiesOnly yes` to the host entry.

## 7. Screenshots missing on GitHub

**Symptom:** The Step 5 screenshots were committed but not visible on the repo's main page.
**Cause:** They lived on the `step/05-firewall` branch, and `main` still pointed at Step 4. A push was also rejected as non-fast-forward because my local branch was behind the remote.
**Fix:** Pulled the remote branch with `git pull --ff-only`, then merged each step branch into `main` with a pull request.

## If I get locked out of SSH

1. Open the VM in virt-manager and use the console, which bypasses SSH.
2. Undo the change that broke access (for example remove `/etc/ssh/sshd_config.d/01-hardening.conf` and restart `ssh`).
3. Or restore the last good snapshot: `virsh snapshot-revert <vm-name> <snapshot>`.

Connection refused / No route to host after a host reboot: the VM and libvirt network weren't up. Fixed with virsh start and virsh autostart.
The SSH tab froze during the fail2ban test: the ban blocked the host's IP, including open sessions. Fixed from the virt-manager console with fail2ban-client set sshd unbanip.
ssh -t ... > file hung: sudo's password prompt went into the redirected file. Fixed by writing the file on the VM and copying it with scp.
nmap said "host seems down": the firewall dropped its ping probes. Fixed with nmap -Pn.
Duplicate User lines in ~/.ssh/config: the first match wins, so I split it into two host entries.

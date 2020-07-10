# LazyAdmin

This was my first TryHackMe box. Lets see what I can discover!

##Port Enumeration

Started my hunt with a simple nmap scan:

```bash
➜  ~ sudo nmap -sC -sV -Pn -n $IP
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-10 12:06 MDT
Nmap scan report for 10.10.151.48
Host is up (0.25s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.05 seconds
```

Port 22 was open using OpenSSH 7.2p2 and port 80 was hosting Apache 2.4.18

##Directory Enumeration

Utilized gobuster to enumerate directories on the IP:

```bash
➜  ~ gobuster dir -t 150 -u http://$IP -w Desktop/wordlist2.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.151.48
[+] Threads:        150
[+] Wordlist:       Desktop/wordlist2.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/10 12:49:13 Starting gobuster
===============================================================
/content (Status: 301)
```

Browsing to 10.10.151.48/content revealed a website that was built using SweetRice. 
gobuster on /content:
```bash
➜  ~ gobuster dir -t 100 -u http://$IP/content -w Desktop/wordlist2.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.151.48/content
[+] Threads:        100
[+] Wordlist:       Desktop/wordlist2.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/10 12:47:14 Starting gobuster
===============================================================
/js (Status: 301)
/images (Status: 301)
/inc (Status: 301)
/as (Status: 301)
/_themes (Status: 301)
/attachment (Status: 301)
```

I manually went through each directory in the browser, and /inc had a couple interesting files including a ```mysql_backup/``` folder. I grabbed the file inside and examined it. Found an interesting entry: ```"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb```. Looks like a hash. The only vulnerable "moderm" hashing algorithms I know of are SHA1 and MD5, the latter being significantly easier to reverse, hell, I created a program in my Python Pentesting Repo to do it. Lo and Behold: ```Password123```. But where do we put this password? A potential hint within that webpage was "If you are the webmaster, please to go Dashboard > General > Website setting". Doing a quick google search for "sweetrice dashboard" I found a google image showing a location in /content/as, one of the directories gobuster found!  Sure enough, there's a login page at /content/as. Using user ```manager``` and password ```Password123``` I was able to get into the the dashboard! Huzzah!

Lets see what exploits SweetRice has:

##Exploitation

```bash
➜  ~ searchsploit sweetrice
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
SweetRice 0.5.3 - Remote File Inclusion       | php/webapps/10246.txt
SweetRice 0.6.7 - Multiple Vulnerabilities    | php/webapps/15413.txt
SweetRice 1.5.1 - Arbitrary File Download     | php/webapps/40698.py
SweetRice 1.5.1 - Arbitrary File Upload       | php/webapps/40716.py
SweetRice 1.5.1 - Backup Disclosure           | php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Request Forgery  | php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Request Forgery  | php/webapps/40700.html
SweetRice < 0.6.4 - FCKeditor Arbitrary Fil   | php/webapps/14184.txt
```

What caught my eye was ````SweetRice 1.5.1 - Arbitrary File Upload       | php/webapps/40716.py```. Reading through it, we can use a PHP reverse shell to gain access!

Browsing through the dasboard I found in the ad section that you can upload ~~Ad~~ arbitruary code...lets upload a PHP reverse shell and get going!

##Post Exploitation

*Currently at this stage of the box, this will be updated in the coming days*
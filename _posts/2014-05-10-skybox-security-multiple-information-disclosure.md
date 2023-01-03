---
title: "[CVE-2014-2084] - SKYBOX Security – Multiple Information Disclosure"
date: 2014-05-10
author: Luigi Vezzoso
layout: post
tags: 
    - CVE
    - vulnerability
category: Essays
---

Date: [22-Jan-2014]

Exploit Author: [Luigi Vezzoso]

Vendor Homepage: [http://www.skyboxsecurity.com]

Version: [Skybox View Appliances with ISO versions: 6.3.33-2.14, 6.3.31-2.14, 6.4.42-2.54, 6.4.45-2.56, 6.4.46-2.57]

Tested on: [Centos 6.4 kernel 2.6.32]

CVE : [CVE-2014-2084]

#OVERVIEW

A vulnerability has been found in some Skybox View Appliances’ Admin interfaces which would allow a potential malicious party to bypass the authentication mechanism and obtain read-only access to the appliance’s administrative menus. This would allow the malicious party to read system-related information such as interface names, IP addresses and the appliance status.

#INTRODUCTION

Skybox Security has a complete portfolio of security management tools that deliver the security intelligence needed to act fast to minimize risks and eliminate attack vectors.  Based on a powerful risk analytics platform that links data from vulnerability scanners, threat intelligence feeds, firewalls and other network infrastructure devices – Skybox gives you context to prioritize risks accurately and automatically, in minutes.  

#VULNERABILITY DESCRIPTION

It's possible to obtain useful information about the version and network configuration of skybox appliances bypassing the webui interface.

For the appliance system info open with a browser:

https://1.1.1.1:444/scripts/commands/getSystemInformation?_=111111111

For the appliance network info open with a browser:

https://1.1.1.1:444/scripts/commands/getNetworkConfigurationInfo

#VERSIONS AFFECTED

Skybox View Appliances with ISO versions: 6.3.33-2.14, 6.3.31-2.14, 6.4.42-2.54, 6.4.45-2.56, 6.4.46-2.57

#SOLUTION

Please refer to the vendor security advisor: Security Advisory 2014-3-25-1

#CREDITS

Luigi Vezzoso

email:  luigivezzoso@gmail.com

skype:  luigivezzoso
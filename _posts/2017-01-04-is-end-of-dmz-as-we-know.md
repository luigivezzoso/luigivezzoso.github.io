---
title: Is the end of the DMZ as we know?
date: 2017-01-04 13:00:00
author: Luigi Vezzoso
layout: post
tags: 
    - general
category: posts
---
The 2016 was denoted by an increased usage of term as Software Defined Network (SDN), Cloud Architecture, etc.....

As a network security guy I always trust to hardware security solution like firewall, IPS, etc. between well defined zones (LAN/TRUST-NETWORK, WAN/UNTRUST-NETWORK, DMZ).

So, the first advise to my clients was always to separate those zone introducing  a firewall and or an IPS in front the the datacenter and/or server farms (we suppose that the client has already an internet firewall :-) )

The introduction of an "inside" firewall is a challenging project for many reasons:

- first of all every time someone introduce  new application (client, external consultant, vendors, etc.) seams that nobody knows the application in deep to define exacly which TCP/UDP ports are required....
- the cloud/virtualized nature of application and/or system doesn't permitt the easy introduction of firewall.


*The result of those implementation it's usually a blandy configured firewall between the attacker and the application or server.... when the firewall exist.*

After attending some vmware SDN trainigs I have the opportunity to open my mind to a different approach on the network security. The adoption of  SDN in the highly virtualized or cloudy infrastructure permit a superior level of security added to a very flexible way to deploy the solution.

In particular trough the *microsegmentation* you can filter traffic between VMs!!! You can create DMZ everywhere is needed indipendent from the network topology or physical infrastructure.

The network and or system admin can deploy firewall in a very quick way reducing costs, effort and appling security from the beginning of the project/deploy od the apps.

But who can or should adopt the SDN solution?

In few words: every company that have already adopted cloud or virtualized solutions with a lots (some hundreds) of VMs.

Last concept: the adoption of SDN not require to remove the "old" security solution... It is complementary and can be succesful integrated with the already deployed solutions. 
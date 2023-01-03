---
title: "Is data at rest encryption worth against data breach?"
date: 2021-01-06
author: Luigi Vezzoso
layout: post
tags: 
    - data protection
category: posts
---
![Photo by marcos mayer on Unsplash](assets/postimages/marcos-mayer-8-ni1wtqcgy-unsplash_orig.jpg)

Some years ago I attended very interesting discussion between [evilsocket](https://www.evilsocket.net/) (a very know, productive and skillful  Hacker) and a speaker of HackInBO, one of the most important Italian hacker community event. The discussion was part of Q&A session at the end of talk and it was focused on the effectiveness of encryption to protect our data against a real criminal attack.

The origin of discussion was mainly focused on the speaker statement which (more or less) sounds like this:

>Using our encryption product (in particular was a Crypto API framework) an attacker could not access data in event of server (webapp) compromission

Probably this kind of statement is to generic and could be easily rebutted by an "expert" attacker.

Usage of a *crypto-framework* permits to developers to adopt a secure and standard API interface for all their needs (encryption, signature, etc.) without to reinvent the wheel every time. Usually not the whole APIs provided by the framework are used  at the same time by an application because part of the UI or the back-end could use just part of it for their scope. APIs are usually authenticated and just the user or service account used by the application server is able to use the API and *encryption keys*.

Here where the first objection to the speaker statement come out:

> if an attacker could exploit/compromise the application service. He/she can interact with the API to do almost (and more!) the application can do from their WebUI. So an attacker, just after obtained a shell or in general RCE permission on the system could CALL the API in order to decrypt, encrypt (insert) data.
>
>      ¯\_(ツ)_/¯

The discussion didn't go much further from that and in first instance seams that encryption could not offer much resistance or protection against a real attacker.... but is this really the only conclusion or we should have an higher level view of problem?
Let me go with some obvious and very know statements about security:

- security is not a "things" which you can buy, but is more a series of organizational process to be adopted in order to manage and assure a certain level of risk_

- security is always a multi-level approach and there isn't a all-in-one product/system which can absolve all kind of protection

So encryption is not a "panacea for all evil" and is a piece of the security architecture puzzle used to protect your organization.

In  particular to continue the example of vulnerable web application which use encryption API we can firmly affirm that application level security like *secure-coding*, *code-review*, *WAF*, *IPS*, *penetration testing*, etc. are probably the more appropriate way to reduce the attack surface of the application and to protect data for that specific attack-tree or scenario.
We must always remember that the attacker is very powerful and will try all the ways to gain access our data: not only the main entrance and we should protect all other doors too. And It is here were the encryption, our powerful tool, help us to maintain attacker out from our data.

If correctly implemented a crypto system is composed by HSM or KeyManager (depending on the required security level, the type of integration, etc.) and CryptoAPI and encryption software.

There are many possible scenario where encryption plays fundamental role protecting our data. Hereafter I'm just reporting some possible scenario related to the example previously discussed. In particular, as I already wrote, attacking the webapp/application service within the specific user/service used, is not the only way to gain access.

The attacker could try to attack other services (CIFS/NFS/FTP/RDP/...) obtaining access with different user to the system. The attacker could obtain access to system with user or root permission but if data encryption is correctly applied with this access he/she cannot access direct data access. Data stored in disks/databases is encrypted and only the data/keys owner can access them (the access is protected using mutual authentication with certificate).

An internal user (also an admin) can have access to critical system for maintenance. The use of encryption solution can add an additional security boundary between the user and data to increase the standard OS level protection (ACL, DACL, etc).  Every keys access could be tracked to identify which user/process is trying to access encrypted data.

Every private keys should be protected by an HSM or Smartcard. In this way we can assure that keys never leave the secure device. Webserver certificate, authentication certificate, etc. could be protected by tamper resistant, certified, security device and protected form exporting and reusing them for evil activities.
Just some conclusion. We are facing new data-breaches every few days. Multi-Level security approach is one of the most complete paradigm to be used. In our security infrastructure, encryption could be a very powerful tool to protect out data from an attacker because can protect several scenario.

If correctly applied data-at-rest encryption could help us to protect our data from several attack path:

- lateral movement
- private keys protection
- insider-threat (rogue admin)

Just that for today. I hope this post could be useful to better understand how encryption could be used to protect against real attack to block data-breach events. I hope also to be fair enough to  tell that other application level security measure should be taken to protect application: in particular I think that applying correct secure coding security practice and penetration assessment could reduce the attack surface and minimize risk of data-breaches....

But just in case *#encrypteverithing*!!